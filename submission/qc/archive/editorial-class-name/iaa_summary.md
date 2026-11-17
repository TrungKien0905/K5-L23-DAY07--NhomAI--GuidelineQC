# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 367 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 367 | 0.901 | 99.7% | 95.6% | 99.5% | 25/8 | 2 | ĐẠT |
| 101|102 | 13 | 185 | 0.900 | 100.0% | 100.0% | 100.0% | 0/0 | 0 | ĐẠT |
| 101|103 | 13 | 116 | 0.901 | 100.0% | 100.0% | 100.0% | 0/0 | 0 | ĐẠT |
| 102|103 | 14 | 66 | 0.901 | 98.5% | 75.8% | 97.0% | 25/8 | 2 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 70 ({'iou_thap': 8, 'sai_thuoc_tinh': 18, 'bo_sot_thua': 21, 'sai_lop': 1, 'co_can_review': 5, 'bat_dong_ignore': 17}), trên 12 ảnh cần calibration.
