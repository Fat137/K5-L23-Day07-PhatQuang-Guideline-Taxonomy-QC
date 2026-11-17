# Thông tin nhóm

- Tên nhóm: PhatQuang
- Mã thành viên (ba chữ số): 063, 200
- Người điều phối: 063
- Người review bản cuối: 063 (trùng người sửa nhãn calibration vì 200 không tham gia vòng duyệt; xem `submission/qc/QC_REPORT.md` mục 4)
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Phân công

| Mã | Việc |
|---|---|
| 063 | Điều phối; guideline cá nhân; gán ảnh 1–20 và gán chéo; chạy script; sửa nhãn calibration; duyệt QC vòng 1–2; báo cáo QC |
| 200 | Guideline cá nhân; gán ảnh 21–40 và gán chéo; họp thống nhất guideline (Pha 2) và calibration (Pha 5) |

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 16/16 trên lô so sánh 8 ảnh có 2 lượt gán độc lập đã kiểm chứng; 32/40 ảnh chỉ có 1 lượt gán (nhóm chia 20/20, phần gán chéo còn box nạp lại nên bị loại)
- IAA trước calibration (kèm mẫu số): IoU TB 0.938 trên 34 cặp box (đạt ≥ 0.85); cùng lớp 32/34 = 94.1 % (chưa đạt 95 %)
- Số ảnh phải calibration: 8 / 8 ảnh của lô so sánh (4 ảnh bị cờ + R-21 phạm vi cả lô); 26 quyết định
- Vòng QC và số ảnh đã duyệt / cả lô: 2 vòng, mỗi vòng 8/8 (vòng 1 REWORK, vòng 2 RELEASE)
- Tỉ lệ lỗi: vòng 2: 0 % theo ảnh (0/8), 0 % theo đối tượng (0/37); vòng 1: 37.5 % (3/8), 10.8 % (4/37)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: **Release, chỉ cho lô 8 ảnh**: vòng 2 duyệt toàn lô không còn lỗi; 32 ảnh còn lại chưa đủ 2 lượt gán nên ngoài phạm vi bàn giao.
