# Guideline gán nhãn — v0.2 — <mã>

- Phiên bản: v0.2
- Ngày: 2026-11-16
- Người viết / duyệt: <mã> / duyệt ở Pha 2

## 1. Mục tiêu

**Mô hình:** bộ phát hiện vật 2D cho camera giao thông / xe tự lái, cùng loại bài toán với benchmark KITTI 2D,
BDD100K detection, IDD detection.

**Nhãn dùng để:** huấn luyện và đánh giá mô hình phát hiện **người tham gia giao thông** (đi bộ / đang lái xe) và
**phương tiện đường bộ**. Mỗi box là một vật thật mà xe phải phản ứng (tránh, nhường, dừng).

**Vì sao tách lớp như dưới đây:** mỗi lớp có cách di chuyển khác nhau, nên xe tự lái dự đoán quỹ đạo khác nhau.
Người đi bộ chậm và đổi hướng bất ngờ. Người lái xe đi nhanh, theo luồng xe. Xe buýt dài và hay dừng đột ngột.
Xe ba bánh hay chạy chậm và chất hàng cồng kềnh.

**Sai ở đâu thì nguy hiểm (từ nặng đến nhẹ):**
1. Bỏ sót `person` / `rider` (người bị che, người ở mép ảnh, người ngồi trên xe).
2. Gán box cho vật không thật (tượng, ma-nơ-canh, billboard, phản chiếu, đồ chơi) → phanh vô cớ.
3. Gộp người lái với xe vào một box → không đếm được người.
4. Nhầm `rider` với `person`, hoặc nhầm lớp xe (tuk-tuk → `motorcycle`, buýt → `truck`).
5. Box lỏng, sai thuộc tính.

## 2. Lớp và thuộc tính

**Nguyên tắc chung**
- **R-01:** Chỉ gán vật thật, đúng kích thước thật, có mặt trực tiếp trong cảnh. Mỗi vật thuộc đúng một lớp.
- **R-02:** Không gán tượng, mô hình, đồ chơi, ma-nơ-canh, hình in (billboard, poster, tranh tường, sticker),
  hình trên màn hình, và mọi phản chiếu (gương, kính, đường ướt, kính mũ). Không vẽ box, cũng không vẽ ignore.
- **Phép thử 3 câu:** (a) Có đúng tỉ lệ thật không? (b) Có nằm trên mặt đất trong cảnh không (không trên bệ, trong tủ kính, trên giấy, trong gương)?
  (c) Có phải chất liệu người / xe thật không? → Cả 3 câu "có" thì gán; chỉ cần 1 câu "không" thì áp dụng R-02.

### Lớp

**Nhóm người**

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người **không dùng phương tiện để di chuyển**: đi bộ, đứng, ngồi trên ghế / đất, dắt xe đạp hoặc xe máy bằng tay. | 001, 002, 003, 007, 008, 009, 010, 011, 013, 014, 016, 019, 023, 025, 029, 030, 031, 032, 034, 036, 037, 039 | Người ngồi trên xe máy → `rider`; tượng `017`, Predator `011`, ma-nơ-canh `033`, billboard `018`, poster `030`, sticker `040` → R-02 |
| `rider` | Người **đang ngồi trên** xe máy, xe đạp, xích lô, xe ba bánh hở hoặc xe không mui (lái hoặc ngồi sau), kể cả khi xe đang dừng. | 004, 005, 006, 008, 009, 010, 014, 016, 019, 021, 022, 023, 025, 026, 029, 031, 034, 035, 039, 040 | Người đứng cạnh xe hoặc dắt xe → `person` (`003`, `029`); người trong xe kín chỉ thấy qua kính → không gán (`009`, `035`) |

**Nhóm xe**

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người cỡ nhỏ, thân liền khối: sedan, hatchback, wagon, SUV, MPV, taxi, xe cổ, xe diễu hành có biển số. | 002, 003, 005, 006, 012, 013, 015, 016, 018, 019, 021, 023, 024, 025, 028, 031, 032, 034, 035, 036, 039, 040 | Pickup → `truck`; Hiace → `van`; xe đồ chơi `038`; mô hình trên nóc `024` → R-02 |
| `van` | Xe dạng hộp, thân cao, khoang kín liền cabin, ≤ 16 chỗ (Hiace, Transit, Starex, xe cứu thương dạng van). | 005, 011, 016 | Minibus thân buýt → `bus` (`009`); MPV gầm thấp → `car` |
| `truck` | Xe chở hàng có **thùng tách khỏi cabin**: pickup (kể cả có nắp thùng), xe tải nhỏ, xe tải thùng, đầu kéo kèm rơ-moóc, xe chở ô tô, songthaew. | 001, 002, 007, 008, 012, 016, 018, 025, 028, 036, 037 | Tượng pickup `027` → R-02; xe thùng liền cabin → `van` |
| `bus` | Xe chở khách công cộng / đường dài có thân buýt (cửa khách, nhiều hàng ghế), gồm minibus thân buýt. | 006, 009, 021, 031, 037, 040 | Hiace → `van`; songthaew → `truck` |
| `motorcycle` | Xe hai bánh có động cơ (xe số, tay ga, mô tô, xe máy điện), tính cả sidecar hoặc thùng gắn bên; **chỉ box phần xe**. | 003, 005, 006, 008, 009, 010, 011, 013, 014, 016, 021, 022, 023, 025, 026, 029, 030, 031, 034, 035, 036, 039 | Xe đạp → `bicycle`; tuk-tuk → `three_wheeler`; chỉ thấy gương ở đáy `040` → không gán |
| `bicycle` | Xe đạp hai bánh không động cơ (kể cả xe đạp chở hàng); **chỉ box phần xe**. | 003, 006, 010, 029, 034, 039 | Xe đạp điện có động cơ → `motorcycle` (`029`); xích lô → ignore |
| `three_wheeler` | Xe ba bánh có động cơ chở khách / hàng: tuk-tuk, xe lam, auto-rickshaw; box gồm cả mái và hàng trên xe. | 004, 016, 035, 040 | Xe máy kéo thùng bên → `motorcycle` (`025`); xích lô đạp → ignore |

**Lớp vùng**

| Lớp | Định nghĩa | Ví dụ (ảnh) |
|---|---|---|
| `ignore_region` | Vùng không chấm điểm, chỉ dùng cho: (A) phương tiện lạ không thuộc lớp nào (xích lô, xe đẩy hàng), (B) vật cao < 20 px, (C) cụm vật không tách được (đám đông, bãi xe), (D) quá tối hoặc mờ đến mức không nhận ra lớp. | (A) 023, 039 · (B, C) 002, 003, 006, 008, 011, 013, 019, 021, 023, 025, 030, 031, 032, 036, 039 |

### Vật không thuộc lớp nào

| Loại | Ảnh |
|---|---|
| Vật không thật (R-02) | 001 poster; 011 Predator; 013 tượng + phản chiếu; 015 phản chiếu đường; 017 tượng; 018 billboard; 020 xe trong gương; 024 mô hình vàng; 026 tranh tường; 027 tượng pickup; 028 tượng ngựa; 030 poster; 031 tranh; 033 ma-nơ-canh; 036 ảnh quảng cáo; 038 đồ chơi; 040 sticker + phản chiếu kính mũ |
| Xe chủ / ego (Cityscapes: void) | 020 gương; 040 gương xe máy ở đáy ảnh |
| Đèn, biển báo, nón giao thông, động vật, đồ vật | 004, 009, 013, 017, 028, 032 |
| Ảnh không có vật nào (vẫn xuất COCO) | 017, 020, 027, 033, 038 |

### Thuộc tính

**`occluded_level`** (cùng thang 0–3 như KITTI) — % diện tích phần vật trong ảnh bị vật khác che.
Không tính phần bị mép ảnh cắt, không tính tự che.

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` thấy rõ | < 10% | 2 người đi bộ `014` |
| `1` che ít | 10% – < 40% | xe sau hàng rào `018` |
| `2` che nhiều | 40% – < 80% | Hiace sau nhóm rider `005` |
| `3` gần hết / không ước lượng được | ≥ 80% | người chỉ thấy chân `007` |

- Nằm giữa hai mức → chọn mức cao hơn.
- Rider che xe máy, xe máy che chân rider → cả hai đều tính là bị che.

**`truncated`** — `yes` nếu vật bị mép ảnh cắt (`001`, `012`, `015`, `016`); ngược lại `no`.

**`can_review`** — tick khi đã vẽ box nhưng chưa chắc. Vật chưa biết có nên vẽ hay không → ghi `image_issues.csv`.

## 3. Quy tắc vẽ box

**Mọi box**

| ID | Quy tắc |
|---|---|
| R-11 | Ôm sát phần nhìn thấy, sai số ≤ 2 px (vật cao < 50 px: ≤ 1 px); không kéo dài để đoán phần bị che. |
| R-13 | Bị che mà còn nhận ra lớp thì vẫn vẽ; vật bị cột chia đôi → 1 box. |
| R-14 | Bị cắt mép: vẽ đến mép ảnh, `truncated=yes`. |
| R-15 | Vật cao < 20 px → `ignore_region` (KITTI chỉ đánh giá vật cao từ 25 px trở lên; lab dùng mốc 20 px). |
| R-18 | Không gán xe chủ (`020`, `040`). |
| R-19 | Mờ chuyển động (`014`, `015`): ôm phần thân đậm, bỏ vệt mờ, tick `can_review`. Ảnh đêm gán bình thường; không nhận ra lớp → ignore. |
| R-20 | Vật sau hàng rào (`018`) vẫn gán. Người trong xe kín chỉ thấy qua kính → không gán. Phản chiếu trên kính → không gán. |

**Box người (`person`, `rider`)**
- **R-07a — person hay rider?** Nếu người đó đi bằng chân → `person`. Nếu đang ngồi trên phương tiện → `rider` (theo Cityscapes).
  - Người dắt xe → `person` (`003`, `029`).
  - Người ngồi trên xe máy đang đậu → `rider` (`010`, `008`).
- **R-10 — Người và xe là box riêng:** mỗi rider 1 box, xe 1 box. 2 người trên 1 xe máy = 2 `rider` + 1 `motorcycle` (`026`).
- Box người tính mũ bảo hiểm, nón, ba lô; **không** tính đồ gánh hoặc cầm xa thân (đòn gánh `003`, ống thép `029`).
- Người ở hàng ghế hở của xe ba bánh / xe không mui → `rider` (tài xế tuk-tuk `004`, `016`; người lái xe mui trần `019`).

**Box xe**
- Tính gương chiếu hậu, cản, giá nóc, sidecar (`022`), thùng gắn bên (`025`); không tính ăng-ten.
- **R-12 Hàng hóa:** hàng trong khung thùng xe → nằm trong box (`012`). Hàng thò ra ngoài → không tính, trừ `three_wheeler`: box gồm cả hàng chất trên mái (`035`).
- **R-17 Xe chở xe** (`002`): xe chở = 1 `truck` (đầu kéo + rơ-moóc, theo Cityscapes); mỗi ô tô được chở = 1 `car`.
- Xe đậu hay chạy đều gán như nhau (`024`, `030`).

**`ignore_region`**
- (A) Phương tiện lạ: xích lô (`023`, `039`); người ngồi trên vẫn gán `rider`.
- (B) Vật < 20 px: gộp các vật nằm gần nhau thành một vùng.
- (C) Cụm ≥ 3 vật cùng lớp chồng lấn, không tách được mép → 1 vùng (`019`, `021`, `030`, `032`). Vật tách được đứng trước cụm vẫn vẽ box riêng.
- Box vật được phép nằm bên trong vùng ignore.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| 002 | Xe chở ~10 ô tô | R-17 | 1 `truck` + mỗi xe được chở 1 `car` | Cityscapes: rơ-moóc của xe tải thuộc truck |
| 003 | Người áo đỏ dắt xe đạp | R-07a | `person` + `bicycle` | Đang đi bằng chân |
| 004 | Tuk-tuk có tài xế | 3w, R-07a | `three_wheeler` + `rider` | IDD autorickshaw |
| 005 | Cận cảnh rider, xe bị che | R-10, R-13 | Mỗi người 1 `rider`; mỗi xe `motorcycle` occ 2–3 | Người và xe tách box |
| 007 | Chỉ thấy chân người, ảnh tối | R-13 | `person` occ 3 + `can_review` | Không bỏ sót người |
| 008 | Songthaew | truck | `truck` + `can_review` | Thân xe là pickup |
| 009 | Minibus County, khách ngồi trong | bus, R-20 | `bus`; khách không gán | Thân buýt, cửa khách |
| 010 | Cô gái ngồi trên xe máy đang đậu | R-07a | `rider` + `motorcycle` | Đang ngồi trên phương tiện |
| 011 | Đầu Predator; xe máy độ trưng bày | R-02 | Predator không gán; xe → `motorcycle` + `can_review` | Sai tỉ lệ người; xe có thể không chạy được |
| 012 | Pickup choán đáy ảnh | truck, R-14 | `truck` truncated | Không phải xe chủ |
| 014 | Rider mờ chuyển động | R-19 | `rider` + `motorcycle` + `can_review` | Mép không chắc |
| 018 | 4x4 sau hàng rào; billboard | R-20, R-02 | Xe 4x4 → `car` + `can_review`; billboard không gán | Không thấy thùng sau |
| 019 | Người lái xe mui trần; mascot; khán giả | R-07a, R-16 | Người lái → `rider`; mascot → `rider` + `can_review`; khán giả → ignore | Cityscapes: người lái xe không mui là rider |
| 020 | Gương xe chủ | R-18 | Không gán gì | Cityscapes: ego vehicle là void |
| 021 | Kẹt xe máy | R-16 | Vật tách được → box riêng; khối phía sau → ignore | Tiêu chí tách được mép |
| 023 | Xích lô có người đạp | ignore (A) | Xích lô → ignore; người đạp → `rider` | Không thuộc lớp xe nào |
| 024 | Mô hình Pacer vàng trên nóc xe | R-02 | Không gán + ghi `image_issues` | Nhỏ hơn xe thật |

## 5. QC và phiên bản

**Ngưỡng QC áp dụng** (giữ nguyên `taxonomy.json → rules`):

| Đại lượng | Ngưỡng |
|---|---|
| Chiều cao box tối thiểu | 20 px |
| IoU đạt | ≥ 0.85 (box cao ≥ 50 px); ≥ 0.70 (nhỏ hơn) |
| Ghép cặp box | IoU ≥ 0.5 |
| IAA | IoU trung bình ≥ 0.85, cùng lớp ≥ 95% |
| Ảnh lỗi để Release | ≤ 2% |

- Số liệu ghi `tử số/mẫu số`; không có cặp box → N/A.
- Loại lỗi: `bo_sot`, `thua`, `sai_lop`, `sai_vi_tri_kich_thuoc`, `sai_thuoc_tinh`, mỗi lỗi ghi Rule ID.
- Cùng kiểu lỗi lặp ở ≥ 2 ảnh → Rework.
- Chỉ một người đặt ignore lên một vật → đưa vào calibration.
- Cặp dễ lệch cần theo dõi riêng: `person`↔`rider`, `car`↔`van`↔`truck`, `bus`↔`van`.

**Ai quyết ca mơ hồ**
1. Người gán làm theo quy tắc; chưa chắc → `can_review` / `image_issues.csv`.
2. Reviewer nhóm chỉ định chốt các ca ở mục 4, ghi mã vào `calibration_decisions.csv`; người kiểm lại khác người sửa.
3. Đổi lớp / cách box → `semantic`: tăng phiên bản, `pham_vi=lo`, rà lại cả lô. Chỉ sửa câu chữ → `editorial`.

**Lịch sử phiên bản**

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | 6 lớp: car, truck, van, motorcycle, person, ignore_region | — |
| v0.1 | Định nghĩa lớp, ngưỡng che, R-02, quy tắc ignore | 005, 017, 020, 033, 038 |
| v0.2 | Thêm `rider` (theo Cityscapes, BDD100K) | Rider chiếm phần lớn người trong ảnh VN / Thái: 005, 006, 021, 026 |
| v0.2 | Thêm `bus` (theo Cityscapes, BDD100K) | 006, 009, 021, 031, 037, 040 |
| v0.2 | Thêm `bicycle` (theo Cityscapes, BDD100K) | 003, 006, 010, 029, 034, 039 |
| v0.2 | Thêm `three_wheeler` (theo IDD autorickshaw) | 004, 016, 035, 040 |
| v0.2 | Xích lô / xe lạ → ignore (theo IDD vehicle fallback, KITTI DontCare) | 023, 039 |
