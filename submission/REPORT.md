# Báo cáo Day 08 — 2A202604054 - Vũ Duy Thế

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | mặc định `--conf 0.3` (không lớp nào đạt recall 90% ở ngưỡng ≤ 0.5: person tối đa 15/18 = 83%, motorbike 4/6 = 67%, car 7/8 = 88%; bicycle 0, bus 1, truck 0 vật gold < 5 → không chọn ngưỡng riêng) — nguồn `qc/threshold_sweep.md` | r01 (conf 0.25) bắt được người / xe máy ở xa; r02 (conf 0.5) giảm 35% box nhưng mất gần hết vật ở xa (img_007: 26 → 11 box), nên giữ ngưỡng thấp 0.3 để ưu tiên recall và để người duyệt xoá box thừa |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | `0.15` (recall 6/6 = 100%, precision 6/9 = 67% trên 4 ảnh gold) | YOLO26 không có lớp này nên gọi tuk-tuk là car / truck / bicycle (r01, r04); YOLOE bắt đúng tuk-tuk img_002 (0.93) và xích lô img_032 (r03). Hạ xuống 0.15 vì xích lô bị che chỉ có conf 0.27 |

- Chuỗi ngưỡng từ `threshold_sweep.py`: `--thr "three_wheeler=0.15"`

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 28 ảnh, 606 box; sau đó `python scripts/inject_traps.py --pre work/prelabel/merged_review/pre.coco.json --student "2A202604054"`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: **SAM 2.1-b** (`sam2.1_b.pt`). Mask bám viền tốt khi box / click đúng (mui, bánh, thân tuk-tuk ở img_004, img_017); chạy trên MPS 2.84 s/ảnh (r04), máy chịu được; trọng số tải tự do, không cần quyền như SAM 3. Không chọn MobileSAM vì ưu tiên chất lượng viền hơn tốc độ khi chỉ cần 3 vật.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Tuk-tuk / xích lô bị YOLO26 gọi sai lớp với conf cao (img_013 tuk-tuk TAXI = car 0.72; img_011 = truck 0.77; img_032 xích lô = bicycle 0.70): tăng ngưỡng lên 0.5 vẫn giữ các box sai này (r02) mà lại mất vật đúng ở xa. Sửa bằng **đổi model** cho lớp đó: YOLOE prompt `three_wheeler`, đặt YOLOE **trước** YOLO26 trong merge để box three_wheeler thắng box car / truck trùng chỗ (IoU ≥ 0.6); phần còn lại người duyệt đổi lớp (5 ca đổi lớp trong `review_stats.md`).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Cao mà sai: img_013 car 0.72 (thật là tuk-tuk), img_011 truck 0.77 (xe ba bánh bị cắt mép), img_032 bicycle 0.70 (xích lô). Thấp mà đúng: img_032 xích lô bị che YOLOE 0.27; img_007 nhóm xe máy / người ở xa 0.26–0.49 (r01) — đều mất nếu đặt ngưỡng 0.5.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: precision 43.2% / recall 82.1% trên 39 đối tượng gold là số **in-sample**; KTC 95% rất rộng (precision 33–55%, recall 67–91%). Lớp bicycle / bus / truck có 0–1 vật gold nên không kết luận gì được về ngưỡng của chúng; ngưỡng three_wheeler 0.15 dựa trên 6 vật.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 474 | 474/519 = 91.3% | 519 box máy |
| edited | 8 | 8/519 = 1.5% | 519 box máy |
| deleted | 37 | 37/519 = 7.1% | 519 box máy |
| added | 19 | 19/501 = 3.8% | 501 đối tượng sau duyệt |

(Đã loại 28 đối tượng quanh bẫy khỏi bảng; 5 trường hợp deleted + added là đổi lớp, nên số vật máy sót thật ≈ 14.)

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? **three_wheeler**: 17 deleted trên 42 box máy (22 accepted + 3 edited + 17 deleted) = 40% box bị xoá, so với person chỉ 4/269 = 1.5%. Nguyên nhân là ngưỡng 0.15 thấp cộng với YOLOE báo nhầm xe đẩy bán hàng / xe bán tải là three_wheeler (img_002, conf 0.37–0.40), sinh box trùng và box ở vật bị cắt mép ảnh (img_003). Đây là cái giá của việc chọn ngưỡng để đạt recall 100% trên gold. Lớp bị **sót** nhiều nhất là motorbike (8 added / 137 box máy), chủ yếu xe nhỏ trong ảnh dày (img_034, img_035).
- So với gold: precision / recall của **máy** 32/74 = 43.2% / 32/39 = 82.1% (mẫu số: 39 đối tượng gold trên 4 ảnh; 42 FP, 7 FN). Precision thấp vì `--conf 0.3` và three_wheeler 0.15 cố tình để nhiều box thừa cho người duyệt xoá.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 11/12 = 91.7% (KTC 95%: 65–99%, ngưỡng đạt 80%); loose 3/3, wrong_class 3/3, missing 3/3, extra 2/3. Loại lọt: **extra** ở img_034. Vì sao: img_034 là ảnh dày (29 vật), mình tập trung tìm vật bị **sót** (miss object) mà không kiểm lại từng box máy có trùng với một vật thật hay không — tức đã bỏ bước "đối chiếu từng box với vật" của checklist. Cùng ảnh này QC sau đó còn tìm thêm 1 xe máy bị sót và 1 box cao 17 px < 20 px.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 91.3% nhưng bắt bẫy vẫn 11/12, nên mình không chấp nhận mù quáng ở ảnh thưa. Tuy vậy QC tìm 5 lỗi đều nằm ở ảnh dày hoặc ca khó (img_003, img_005, img_018, img_034, img_035) và bẫy lọt cũng ở ảnh dày: thiên kiến tự động hoá xuất hiện khi ảnh nhiều vật — mình mệt và có xu hướng tin box máy, chỉ "lướt" box đã có. Bẫy 12 cái quá ít để kết luận chắc (1 bẫy = 8 điểm %).

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 15/4 = **3.75 phút/ảnh** vs duyệt 40/24 = **1.67 phút/ảnh** (giảm ~56%). Tính theo vật còn chênh hơn: gán tay 15 phút / 39 vật = 0.38 phút/vật, duyệt 40 phút / 516 vật = 0.08 phút/vật. Thiên lệch: mình gán gold trước nên đã quen cảnh phố và guideline, lô batch được hưởng hiệu ứng quen tay; ngược lại thời gian duyệt chưa tính vòng Rework sửa 5 lỗi QC, nên tiết kiệm thật nhỏ hơn con số trên.
- Kết luận: **có**, cho person / motorbike / car (accepted cao, person 264/269 giữ nguyên); **chỉ có điều kiện** cho three_wheeler (40% box bị xoá — máy gợi ý được vị trí nhưng mỗi box phải kiểm). Dựa trên: thời gian giảm ~56%/ảnh, accepted 474/519 = 91.3%, recall máy so với gold 82.1%, nhưng QC vòng 1 vẫn 5/8 ảnh có lỗi nên không thể bỏ bước duyệt / QC.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ảnh > 25 vật: **đếm vật trước khi bật box máy**, ghi số đếm; sau khi duyệt, số box phải lệch số đếm ≤ 2 (kiểm được bằng cách so số đếm với số đối tượng trong `batch_reviewed.json`).
  2. Mọi box three_wheeler có conf < 0.45 và mọi box ở mép ảnh phải mở xem và quyết định rõ (giữ / xoá / đổi lớp); kiểm bằng số: 0 box three_wheeler conf < 0.45 được `accepted` mà không có ghi chú.
  3. Không release khi chưa qua QC mẫu: tỉ lệ ảnh lỗi ≤ 2% **và** bắt bẫy ≥ 10/12; nếu vừa sửa xong thì rút mẫu QC mới (seed khác) trên bản đã sửa trước khi chốt.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1-b (`sam2.1_b.pt`) | 3 (2 dương `400,480` `600,110` · 1 âm `720,300`) | **Có** người lái: 1 click âm chỉ khoét lỗ lốm đốm quanh vai/lưng, thân + chân người lái vẫn trong mask | Khung mui hở: người và đồ phía sau (người áo hồng, chai nước) nhìn xuyên khung bị tô vào mask; viền thân, bánh, mui bám tốt | `tuktuk_img013_sam2.jpg` |
| img_004 | SAM 2.1-b (`sam2.1_b.pt`) | 3 (2 dương `620,480` `700,560` · 1 âm `465,450`) | Không: không có người lái trên xe; người đứng giữa hai xe (click âm) không bị dính | Click dương `700,560` đặt sát mặt đất nên mask tràn xuống **bóng đổ** dưới cản trước; tuk-tuk xanh bên cạnh không bị lấn | `tuktuk_img004_sam2.jpg` |
| img_017 | SAM 2.1-b (`sam2.1_b.pt`) | 3 (2 dương `560,420` `680,500` · 1 âm `645,355`) | Không: ghế lái trống, không có hàng hoá | Mask tốt nhất trong 3 ảnh: ôm đủ mui, thân, 3 bánh; click âm rơi vào **mui xe** (không có người để loại) nên vô tác dụng, mui vẫn trong mask | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? **Giúp**: 4.2.2 (lỗ > 5% diện tích phải bỏ) chỉ ra lỗi ở img_013 — khoảng nhìn xuyên khung mui hở bị tô vào mask; 4.2.3 (bóng đổ không thuộc vật) chỉ ra lỗi tràn bóng ở img_004; 4.2.4 (hai vật chạm nhau không chồng polygon) áp được cho người lái ngồi trong xe. **Thiếu**: 4.2 không nói rõ người lái / hành khách ngồi *trong* xe có bị loại khỏi mask xe không, và cách đặt click (bao nhiêu click âm) để tách họ; câu "kính xe thuộc vật" còn treo ở câu hỏi mở 5. Cả hai đã chốt trong v1.1 (CAL-04).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: chưa thử; chỉ dùng `sam2_points.py` offline.

## 6. Calibration (Part 6)

- Số quyết định: 6 (`calibration_decisions.csv`: 2 `loi_may`, 2 `cau_hoi_mo`, 1 `can_review`, 1 `bay_lot`); quan trọng nhất: **CAL-01** (tuk-tuk luôn là `three_wheeler`, merge YOLOE trước YOLO26) vì đây là lỗi hệ thống của máy, không sửa được bằng ngưỡng và ảnh hưởng ~28% ảnh có xe ba bánh; tiếp theo là **CAL-03** (vật bị cắt mép ảnh) vì chiếm 6/14 box `can_review` và đã làm thay đổi nhãn trong `qc_fixed.json`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu mở 2 (xe máy dày gần camera: gán từng vật nếu cao ≥ 20 px, `ignore_region` chỉ cho cụm xa < 20 px) và câu 5 (kính thuộc mask xe, khung hở là lỗ); tuk-tuk / xích lô luôn `three_wheeler`; tiêu chí nhận three_wheeler (3 bánh hoặc mui + khoang khách, xe đẩy không phải); vật cắt mép chỉ gán khi đủ nhận lớp; người lái tách khỏi mask xe; đếm vật trước khi duyệt ảnh dày.
- Nguyên nhân gốc: **máy** cho CAL-01, CAL-02 (thiếu lớp three_wheeler, YOLOE báo nhầm); **guideline mơ hồ** cho CAL-03, CAL-04, CAL-05 (v1.0 không nói về vật cắt mép, người trong mask xe, xe dày gần camera); **người duyệt chưa theo checklist** cho CAL-06 (bẫy extra lọt ở ảnh dày).

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh `can_review` — img_003, img_005, img_035 — + 5 ngẫu nhiên — img_004, img_009, img_017, img_018, img_034 — seed 42), 190 đối tượng; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt, dễ bỏ sót đúng những gì đã bỏ sót lúc duyệt).
- Lỗi theo **đối tượng**: 5/190 = 2.6% · theo **ảnh**: 5/8 = 62.5% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: **sai vị trí/kích thước** 2 (img_005 #44 person chỉ ôm nửa trên, cao 117 → 246 px; img_018 #162 motorbike nhỏ 21 → 33 px); **bỏ sót** 2 (img_034, img_035: xe máy trong cảnh dày chưa có box); **sai lớp** 1 (img_003 #3 three_wheeler bị cắt mép ảnh). Không ghi nhận thừa, sai thuộc tính, sai mask.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 5/8 ảnh lỗi → cận trên Wilson 95% là **86%**; ngay cả khi 0/8 ảnh lỗi thì cận trên vẫn 32%, nên mẫu 8 ảnh không bao giờ chứng minh được ngưỡng 2%. Mẫu chỉ đủ để **bác bỏ** Release (rõ ràng vượt ngưỡng), không đủ để khẳng định chất lượng sau khi sửa.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): Script **kiểm được**: định dạng COCO, lớp hợp lệ, box < 20 px (`validate_labels` bắt box 17 px ở img_034), đếm accepted / edited / deleted / added, tỉ lệ bắt bẫy, tính tỉ lệ lỗi và cận trên từ phiếu. Script **không kiểm được**: box có ôm sát vật thật không (chỉ so với box máy, không so với ảnh), vật mà cả máy lẫn người cùng bỏ sót (img_034, img_035 chỉ phát hiện nhờ nhìn bằng mắt), mình có thật sự làm đủ checklist không, và lỗi trong 16 ảnh không nằm trong mẫu.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không Hold vì guideline đã đủ để quyết các ca gặp phải (các ca mơ hồ đã chốt ở v1.1) và dữ liệu không hỏng; Rework vì tỉ lệ ảnh lỗi 5/8 = 62.5% vượt xa ngưỡng 2% và lỗi lặp theo kiểu (box lệch + bỏ sót đều ở ảnh dày); chưa thể Release.
Nếu Rework / Hold, việc cần làm tiếp: (1) đã sửa đủ 5 lỗi QC + xoá 3 box vật cắt mép theo CAL-03, xuất `labels/qc_fixed.json` và build lại release (24 ảnh, 512 đối tượng, `validate_labels` OK); (2) duyệt lại riêng 6 ảnh dày (005, 018, 021, 024, 028, 029) theo quy tắc 1 ở mục 4 — đếm vật trước khi nhìn box máy; (3) rút mẫu QC mới với seed khác trên bản đã sửa và chỉ Release khi tỉ lệ ảnh lỗi ≤ 2%.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
