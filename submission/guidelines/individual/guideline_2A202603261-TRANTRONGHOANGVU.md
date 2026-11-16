# Guideline gán nhãn — v0.2 — <mã>

- Phiên bản: v0.2
- Ngày: 2026-11-16
- Người viết / duyệt: <mã> / duyệt ở Pha 2

## 1. Mục tiêu

**Mô hình:** bộ phát hiện vật 2D cho camera giao thông/xe tự lái (KITTI 2D, BDD100K detection, IDD detection).

**Nhãn dùng để:** huấn luyện và đánh giá mô hình phát hiện **người tham gia giao thông** (đi bộ/lái xe) và **phương tiện đường bộ**. Mỗi box là một vật thật xe cần phản ứng.

**Vì sao tách lớp:** Mỗi lớp có cách di chuyển khác nhau, giúp xe tự lái dự đoán quỹ đạo chính xác.

**Sai ở đâu thì nguy hiểm (từ nặng đến nhẹ):**
1. Bỏ sót `person` / `rider`.
2. Gán box cho vật không thật (tượng, ma-nơ-canh, billboard, phản chiếu) → phanh vô cớ.
3. Gộp người lái với xe → không đếm được người.
4. Nhầm `rider` với `person`, hoặc nhầm lớp xe.
5. Box lỏng, sai thuộc tính.

## 2. Lớp và thuộc tính

**Nguyên tắc chung**
- **R-01:** Chỉ gán vật thật, đúng kích thước, trực tiếp trong cảnh. Mỗi vật thuộc đúng một lớp.
- **R-02:** Không gán tượng, mô hình, đồ chơi, hình in, phản chiếu. Không vẽ box/ignore.
- **Phép thử 3 câu:** (a) Đúng tỉ lệ thật? (b) Nằm trên mặt đất? (c) Chất liệu người/xe thật? → Tất cả "có" thì gán; chỉ cần 1 "không" → R-02.

### Lớp

**Nhóm người**

| Lớp | Định nghĩa | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người **không dùng phương tiện để di chuyển**: đi bộ, đứng, ngồi, dắt xe bằng tay. | 001, 002, 003, 007, 008, 009, 010, 011, 013, 014, 016, 019 | Người ngồi trên xe máy → `rider`; tượng, ma-nơ-canh, billboard → R-02 |
| `rider` | Người **đang ngồi trên** xe máy, xe đạp, xích lô, xe ba bánh hở hoặc xe không mui. | 004, 005, 006, 008, 009, 010, 014, 016, 019, 021, 022, 023, 025, 026 | Người đứng cạnh/dắt xe → `person` (`003`); người trong xe kín → không gán (`009`) |

**Nhóm xe**

| Lớp | Định nghĩa | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người cỡ nhỏ, thân liền khối. | 002, 003, 005, 006, 012, 013, 015, 016, 018, 019, 021, 023 | Pickup → `truck`; Hiace → `van`; xe đồ chơi → R-02 |
| `van` | Xe dạng hộp, thân cao, khoang kín liền cabin, ≤ 16 chỗ. | 005, 011, 016 | Minibus thân buýt → `bus`; MPV gầm thấp → `car` |
| `truck` | Xe chở hàng có **thùng tách khỏi cabin**: pickup, xe tải, đầu kéo kèm rơ-moóc. | 001, 002, 007, 008, 012, 016, 018, 025, 028 | Tượng pickup → R-02; xe thùng liền cabin → `van` |
| `bus` | Xe chở khách công cộng/đường dài có thân buýt. | 006, 009, 021, 031, 037, 040 | Hiace → `van`; songthaew → `truck` |
| `motorcycle` | Xe hai bánh có động cơ, tính cả sidecar; **chỉ box phần xe**. | 003, 005, 006, 008, 009, 010, 011, 013, 014, 016, 021, 022, 023, 025, 026 | Xe đạp → `bicycle`; tuk-tuk → `three_wheeler` |
| `bicycle` | Xe đạp hai bánh không động cơ; **chỉ box phần xe**. | 003, 006, 010, 029, 034, 039 | Xe đạp điện có động cơ → `motorcycle`; xích lô → ignore |
| `three_wheeler` | Xe ba bánh có động cơ chở khách/hàng: tuk-tuk, xe lam. Box gồm cả mái và hàng. | 004, 016, 035, 040 | Xe máy kéo thùng bên → `motorcycle`; xích lô đạp → ignore |

**Lớp vùng**

| Lớp | Định nghĩa | Ví dụ (ảnh) |
|---|---|---|
| `ignore_region` | Vùng không chấm điểm cho: (A) phương tiện lạ, (B) vật cao < 20 px, (C) cụm vật không tách được, (D) quá tối/mờ. | (A) 023, 039 · (B, C) 002, 003, 006, 008, 011, 013 |

### Vật không thuộc lớp nào

| Loại | Ảnh |
|---|---|
| Vật không thật (R-02) | 001 poster; 011 Predator; 013 tượng; 015 phản chiếu; 017 tượng; 018 billboard |
| Xe chủ / ego (Cityscapes: void) | 020 gương; 040 gương xe máy |
| Đèn, biển báo, động vật, đồ vật | 004, 009, 013, 017 |
| Ảnh không có vật nào | 017, 020, 027, 033 |

### Thuộc tính

**`occluded_level`** (% diện tích vật bị che, thang 0–3 như KITTI). Không tính bị mép ảnh cắt/tự che.

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` thấy rõ | < 10% | 2 người đi bộ `014` |
| `1` che ít | 10% – < 40% | xe sau hàng rào `018` |
| `2` che nhiều | 40% – < 80% | Hiace sau nhóm rider `005` |
| `3` gần hết / không ước lượng được | ≥ 80% | người chỉ thấy chân `007` |

- Nằm giữa hai mức → chọn mức cao hơn.
- Rider che xe máy, xe máy che chân rider → cả hai đều tính bị che.

**`truncated`** — `yes` nếu vật bị mép ảnh cắt (`001`, `012`); ngược lại `no`.

**`can_review`** — tick khi đã vẽ box nhưng chưa chắc. Vật chưa biết có nên vẽ hay không → ghi `image_issues.csv`.

## 3. Quy tắc vẽ box

**Mọi box**

| ID | Quy tắc |
|---|---|
| R-11 | Ôm sát phần nhìn thấy, sai số ≤ 2 px; không kéo dài đoán phần bị che. |
| R-13 | Bị che mà còn nhận ra lớp thì vẫn vẽ; vật bị cột chia đôi → 1 box. |
| R-14 | Bị cắt mép: vẽ đến mép ảnh, `truncated=yes`. |
| R-15 | Vật cao < 20 px → `ignore_region`. |
| R-18 | Không gán xe chủ (`020`, `040`). |
| R-19 | Mờ chuyển động (`014`): ôm phần thân đậm, bỏ vệt mờ, tick `can_review`. |
| R-20 | Vật sau hàng rào (`018`) vẫn gán. Người trong xe kín/phản chiếu → không gán. |

**Box người (`person`, `rider`)**
- **R-07a — person hay rider?** Người đi bằng chân → `person`. Ngồi trên phương tiện → `rider`.
  - Người dắt xe → `person` (`003`).
  - Người ngồi trên xe máy đang đậu → `rider` (`010`).
- **R-10 — Người và xe là box riêng:** mỗi rider 1 box, xe 1 box. 2 người trên 1 xe máy = 2 `rider` + 1 `motorcycle` (`026`).
- Box người tính mũ bảo hiểm, nón, ba lô; **không** tính đồ gánh/cầm xa thân.
- Người ở hàng ghế hở của xe ba bánh / xe không mui → `rider`.

**Box xe**
- Tính gương, cản, giá nóc, sidecar; không tính ăng-ten.
- **R-12 Hàng hóa:** hàng trong khung thùng xe → nằm trong box. Hàng thò ra ngoài → không tính, trừ `three_wheeler`: box gồm cả hàng trên mái.
- **R-17 Xe chở xe** (`002`): xe chở = 1 `truck`; mỗi ô tô được chở = 1 `car`.
- Xe đậu hay chạy đều gán như nhau.

**`ignore_region`**
- (A) Phương tiện lạ: xích lô (`023`, `039`); người ngồi trên vẫn gán `rider`.
- (B) Vật < 20 px: gộp các vật nằm gần nhau thành một vùng.
- (C) Cụm ≥ 3 vật cùng lớp chồng lấn, không tách được mép → 1 vùng. Vật tách được đứng trước cụm vẫn vẽ box riêng.
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
| 020 | Gương xe chủ | R-18 | Không gán gì | Cityscapes: ego vehicle là void |
| 021 | Kẹt xe máy | R-16 | Vật tách được → box riêng; khối phía sau → ignore | Tiêu chí tách được mép |
| 023 | Xích lô có người đạp | ignore (A) | Xích lô → ignore; người đạp → `rider` | Không thuộc lớp xe nào |
| 024 | Mô hình Pacer vàng trên nóc xe | R-02 | Không gán + ghi `image_issues` | Nhỏ hơn xe thật |
| 025 | Xe máy kéo thùng bên | motorcycle | 1 `motorcycle` + `rider` | Hai bánh chính, thùng gắn bên |
| 029 | Xe máy điện chở ống thép; người đứng cạnh xe đạp | R-12, R-07a | `motorcycle` (không tính ống) + `rider`; `person` + `bicycle` | Xe có động cơ; người đứng thì là person |
| 032 | ~30 người chờ buýt | R-16 | Người tách được → `person`; nhóm chồng lấn → ignore | Chốt ngưỡng tách |
| 035 | Xe lam chở bao tải | 3w, R-12 | `three_wheeler` gồm cả bao; tài xế sau kính → không gán | IDD autorickshaw |
| 037 | Pickup có nắp thùng; buýt | truck, bus | `truck`; `bus` | Cityscapes: pickup thuộc truck |
| 040 | Gương xe máy ở đáy; tuk-tuk; buýt; sticker | R-18, R-02 | Người → `rider` truncated; gương không gán; `three_wheeler`; `bus`; sticker không gán | Không nhận ra xe từ gương |

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
2. Reviewer nhóm chỉ định chốt các ca ở mục 4, ghi mã vào `calibration_decisions.csv`. Người kiểm lại khác người sửa.
3. Đổi lớp/cách box → `semantic`: tăng phiên bản, `pham_vi=lo`, rà lại cả lô. Chỉ sửa câu chữ → `editorial`.

**Lịch sử phiên bản**

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | 6 lớp: car, truck, van, motorcycle, person, ignore_region | — |
| v0.1 | Định nghĩa lớp, ngưỡng che, R-02, quy tắc ignore | 005, 017, 020, 033, 038 |
| v0.2 | Thêm `rider` (theo Cityscapes, BDD100K) | Rider chiếm phần lớn người trong ảnh VN/Thái: 005, 006, 021, 026 |
| v0.2 | Thêm `bus` (theo Cityscapes, BDD100K) | 006, 009, 021, 031, 037, 040 |
| v0.2 | Thêm `bicycle` (theo Cityscapes, BDD100K) | 003, 006, 010, 029, 034, 039 |
| v0.2 | Thêm `three_wheeler` (theo IDD autorickshaw) | 004, 016, 035, 040 |
| v0.2 | Xích lô / xe lạ → ignore (theo IDD vehicle fallback, KITTI DontCare) | 023, 039 |
