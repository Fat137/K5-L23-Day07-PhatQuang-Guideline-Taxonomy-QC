# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **8**; số người gán: 2
- Lượt gán đủ **16/16**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 34 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 8 | 34 | 0.938 | 94.1% | 64.7% | 97.1% | 0/0 | 0 | CHƯA ĐẠT |
| 063|200 | 8 | 34 | 0.938 | 94.1% | 64.7% | 97.1% | 0/0 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 17 ({'sai_thuoc_tinh': 13, 'iou_thap': 2, 'sai_lop': 2}), trên 4 ảnh cần calibration.
