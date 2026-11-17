# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 245 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 245 | 0.796 | 83.3% | 56.7% | 90.6% | 87/103 | 0 | CHƯA ĐẠT |
| 100|101 | 20 | 121 | 0.812 | 85.1% | 37.2% | 90.1% | 19/57 | 0 | CHƯA ĐẠT |
| 102|103 | 20 | 124 | 0.780 | 81.5% | 75.8% | 91.1% | 68/46 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 491 ({'bo_sot_thua': 190, 'iou_thap': 142, 'sai_thuoc_tinh': 115, 'sai_lop': 41, 'co_can_review': 3}), trên 36 ảnh cần calibration.
