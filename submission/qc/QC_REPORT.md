# Báo cáo QC — Nhóm PhatQuang

Lô gốc `day07-street-40` (40 ảnh), 2 thành viên (`063`, `200`). **Lô bàn giao qua pipeline QC là 8 ảnh** có 2 lượt gán độc lập đã kiểm chứng (giải thích ở mục 2). 32 ảnh còn lại chỉ có 1 lượt gán và **không** nằm trong phạm vi bàn giao của báo cáo này.

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn bounding box 2D (9 lớp: car, truck, van, bus, motorcycle, bicycle, tricycle, person, ignore_region) cho mô hình phát hiện người và phương tiện thật trên ảnh đường phố. Ảnh Wikimedia chọn có chủ đích nhiều ca biên (tượng, đồ chơi, phản chiếu, xe trưng bày, đám đông), không đại diện cho phân bố camera giao thông thật; lô bàn giao chỉ 8 ảnh; chỉ dùng cho mục đích học tập.
- Nguồn và phép sử dụng dữ liệu: Ảnh Wikimedia Commons theo CC BY / CC BY-SA / CC0 / public domain, ghi công đầy đủ trong `data/ATTRIBUTION.csv`; chia sẻ lại phải giữ ghi công, ảnh CC BY-SA phải chia sẻ tương tự. Không sửa, không thêm bớt ảnh.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; `submission/qc/labels_calibrated.json` SHA-256 `42a87315d57cd7d05a1a675e8d8cf0fa99b4d6d59d5ce37175a6fc658077f35a` (8 ảnh, 37 annotation); khớp `labels_sha256` trong `reviews/round-02/qc_result.json`.
- Guideline và taxonomy khi gán / khi bàn giao: Gán theo `guideline_v1.0_agreed.md` + `taxonomy.json` v1.0 (khóa trong suốt Pha 3). Bàn giao theo `guideline_v1.1_calibrated.md`, thay đổi `semantic` (`guideline_change.json`: thêm R-21, R-22, R-23; đặt mã R-01…R-20); taxonomy giữ v1.0, không đổi lớp hay thuộc tính.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 16/16 lượt trên lô so sánh 8 ảnh (`img_001, 002, 004, 016, 017, 026, 028, 033`, mỗi ảnh 2 người). 32 ảnh còn lại chỉ có 1 lượt gán độc lập (xem giải thích dưới), nên nằm ngoài phép đo IAA.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 34 cặp box ghép; IoU TB = 0.938 (ngưỡng 0.85, đạt); cùng lớp = 32/34 = 94.1 % (ngưỡng 95 %, CHƯA ĐẠT); cùng `occluded_level` = 22/34 = 64.7 %; cùng `truncated` = 33/34 = 97.1 %.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 0/0 box chưa ghép; 0 box loại do ignore; không có bất đồng vùng ignore trong 8 ảnh.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: 5 ca cùng sai, IAA không bắt được vì hai người giống nhau, tìm ra khi rà nhãn đồng thuận theo R-xx: N01 `img_002` (cả hai vẽ van xa 17 px, trái R-17), N02 `img_001` (truck bị mép phải cắt nhưng cả hai để truncated=no, trái R-15), N03 `img_016` (pickup cả hai gán van, trái R-04), N04 `img_004` và N05 `img_016` (cả hai bỏ sót người trong tuk-tuk, trái R-09); thêm ca thứ 6 lộ ra ở QC vòng 1: Q04 `img_026` (cả hai để người lái occluded_level=0 dù bị người ngồi sau che phần lớn, trái R-21). Ca tham chiếu của coach: `img_017` (tượng) cả hai để trống, đúng R-01; `img_002` (xe chở xe) cả hai gán 1 truck + từng car, đúng R-12; `img_026` (người + xe máy) cả hai gán 2 person + 1 motorcycle, đúng R-08; `img_005` không có trong lô so sánh.

**Vì sao lô so sánh chỉ có 8 ảnh.** Nhóm đã chia 40 ảnh thành hai nửa (063: ảnh 1–20, 200: ảnh 21–40), sau đó gán chéo một số ảnh của người kia. Trước khi chạy IAA, nhóm so toạ độ từng box giữa các file export. Ảnh nào còn box trùng khít toạ độ với nhãn của người kia (tới 0.01 px) thì loại, vì đó là box nạp lại từ file chứ không phải vẽ độc lập. Chỉ giữ ảnh mà mọi box đều do người đó tự vẽ:
- `img_001`, `img_002`: hai người đã gán độc lập từ đầu (0 box trùng toạ độ);
- `img_004`, `img_016`, `img_017` (200 gán chéo), `img_026`, `img_028`, `img_033` (063 gán chéo): 0 box trùng.

Ảnh bị loại vì còn box nạp lại: `023, 024, 025, 029, 035, 037, 040` (phía 063) và `005, 007, 011, 012, 019` (phía 200). Vì vậy IAA ở trên chỉ phản ánh 8 ảnh, chưa đủ để kết luận cho cả lô 40 ảnh.

**Đọc kết quả IAA.** IoU đạt nên vị trí box nhìn chung thống nhất. Đồng thuận lớp chưa đạt do 2/34 cặp ở `img_028`: 200 gán pickup và SUV là `motorcycle`. Đây là lỗi một người chọn nhầm nhãn trong CVAT, không phải guideline mơ hồ. Bất đồng lớn nhất là `occluded_level` (22/34), và **cả hai người cùng lệch** theo cùng kiểu ở người ngồi bị bàn che (`img_001`) và xe trên giàn (`img_002`). Đó là dấu hiệu guideline mơ hồ, nên nhóm sửa guideline (R-21, R-22) thay vì chỉ sửa người.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 26 dòng trong `submission/qc/calibration_decisions.csv`, mỗi ca một dòng: 17 ca IAA (C001–C017; R-04, R-06, R-10, R-14, R-15, R-21, R-22), 5 ca cùng sai (N01–N05; R-04, R-09, R-15, R-17), 4 ca từ QC vòng 1 (Q01–Q04; R-10, R-14, R-21); người chốt 063, 200 hoặc 063+200 theo từng dòng; người sửa 063; kiểm lại ở QC vòng 1 và vòng 2.
- Tác động v1.1 lên nhãn và ảnh: R-21 đổi cách chọn occluded_level cho người nên phạm vi là cả lô (C002 `pham_vi = lo`); sửa nhãn trên cả 8 ảnh, thay đổi chính ở `img_001` (7 person), `img_002` (6 car occ→1, xoá 1 van < 20 px), `img_004` (+2 person), `img_016` (pickup van→truck, +2 person), `img_026`, `img_028` (2 lớp, 1 box); số annotation 34 cặp → 37 box cuối; taxonomy không đổi nên không cần mapping.
- Tham chiếu bằng chứng khi Rework: Vòng 1 (`reviews/round-01/`, nhãn chụp `labels_snapshot.json`) ra REWORK với 4 lỗi; ghi Q01–Q04 vào bảng quyết định, sửa trong CVAT, export `labels_calibrated_a7881725ea0b3d95.json`, chạy lại `build_final.py`; vòng 2 (`reviews/round-02/`) duyệt lại cả 8 ảnh, không còn lỗi; giữ nguyên vòng 1 làm bằng chứng.

**Kiểu lỗi lặp lại.** Qua calibration và vòng 1, có 2 kiểu lặp: (1) **quên người trong xe 3 bánh** (N04, N05; R-09), (2) **`occluded_level` của người bị che** (C002–C006, Q03, Q04; R-21) và nhầm cắt mép với bị che (Q02; R-14 / R-15). Kiểu (2) là lý do vòng 1 ra REWORK dù chỉ có 4 lỗi.

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: 063, 2026-11-17 (cả vòng 1 và vòng 2). Người duyệt trùng người sửa nhãn, vì 200 không tham gia vòng duyệt. Phiếu do công cụ AI soạn nháp sau khi đọc từng ảnh; 063 đối chiếu ảnh và xác nhận từng dòng.
- Đã duyệt / cả lô: 8/8 (duyệt toàn bộ lô, cả hai vòng)
- Ảnh lỗi / ảnh đã duyệt: vòng 2: 0/8 = 0 %; vòng 1: 3/8 = 37.5 %
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: vòng 2: 0/37 = 0 %; vòng 1: 4/37 = 10.8 %
- Vật bỏ sót, ảnh và vùng: 0 ở cả hai vòng. Khối nhoè sát ống kính ở góc trái dưới `img_016` không nhận ra lớp nên không gán (R-19); hai ca bỏ sót người trong tuk-tuk đã được sửa ở calibration (N04, N05).
- Ca khó / phần còn lại, lỗi lặp và ví dụ: vòng 1: nhóm ca khó 2/4 ảnh lỗi, phần còn lại 1/4; 3/4 lỗi là `sai_thuoc_tinh` (lặp), ví dụ `img_026` người lái occ 0 thay vì 2 (R-21), `img_016` người áo đỏ occ 2 dù chỉ bị cắt mép (R-14); lỗi còn lại `sai_vi_tri_kich_thuoc`: `img_002` box truck hụt đuôi rơ-moóc ~19 px (R-10). Vòng 2: không còn lỗi.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: máy chặn được: `validate_labels.py` cảnh báo van 17 px ở `img_002` (R-17), và `qc_compare.py` bắt 200 gán pickup là `motorcycle` ở `img_028`; cần người đọc ảnh: pickup bị cả hai gán `van` ở `img_016` (N03) và người trong tuk-tuk bị cả hai bỏ sót (N04, N05). Hai người giống nhau nên IAA và script đều không thấy.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Theo cây Hold → Rework → Release: (1) Hold? Trong lô 8 ảnh không còn ca guideline chưa chốt, không có ảnh hỏng, đủ bằng chứng hai vòng, nên không Hold; (2) Rework? Vòng 2 duyệt 8/8 ảnh, 0/8 ảnh lỗi (≤ 2 %), không có lỗi lặp, nên không Rework; (3) Release. Người nhận: lab coach Day 07. Phạm vi: **chỉ 8 ảnh** trong `labels_calibrated.json`, dùng cho thực hành; không dùng làm nhãn cho 32 ảnh còn lại, cũng không dùng làm bằng chứng đồng thuận cho cả lô 40 ảnh.
- Nếu còn việc: (a) 32 ảnh ngoài lô cần lượt gán độc lập thứ hai rồi mới đưa vào pipeline (063 và 200); (b) vòng duyệt nên do người không sửa nhãn làm lại (200), vì vòng hiện tại người duyệt trùng người sửa; (c) ca để ngỏ "đồ người mang theo" (`img_014`) cần chốt khi đưa ảnh đó vào lô.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: ca N03 → `img_016` → R-04 → box truck `[0, 274, 331, 252]` trong `labels_calibrated.json` (lớp truck, truncated=yes); ca Q01 → `img_002` → R-10 → box truck biên phải x≈922 (khớp `reviews/round-02/qc_review.csv`, dòng `img_002.jpg`).

## 6. Sửa script và giới hạn của kiểm bằng máy

Script của lab được sửa 3 chỗ. Hai chỗ trong `scripts/make_cvat_labels.py` vì CVAT bản hiện hành từ chối file nhãn sinh ra; một chỗ trong `scripts/common.py` để đọc được CSV đã mở / lưu bằng Excel:

| Commit | Lỗi | Sửa | Bằng chứng chạy lại |
|---|---|---|---|
| `495444c` | CVAT: `Attribute: "can_review": attribute values must be a non-empty array` | checkbox sinh `"values": ["false"]` thay vì `[]` | CVAT nhận bộ nhãn; `cvat/labels_generated.json` sinh lại |
| `4701297` | CVAT: `Label "car": unknown label type "undefined"` | thêm `"type": "rectangle"` cho mỗi nhãn | CVAT nhận bộ nhãn; 9 nhãn đều `rectangle` |
| commit bàn giao Pha 5–6 | `read_csv` đọc bằng `utf8`: CSV lưu từ Excel ("CSV UTF-8") có BOM, nên cột đầu thành `﻿case_id` / `﻿image` và script không tìm thấy cột | `read_csv` đọc bằng `utf-8-sig` (nhận cả file có và không có BOM) | Đọc thử CSV có BOM ra đúng `case_id`; `python -m unittest discover -s tests` 20/20 qua |

Không sửa ngưỡng (`rules` trong `taxonomy.json` giữ nguyên bản đề), không sửa artifact do script khác sinh.

Script kiểm được: tên file theo mã, lớp và thuộc tính đúng taxonomy, box nằm trong ảnh, box cao < 20 px, đủ lượt gán, độ lệch giữa hai người (IoU, lớp, thuộc tính, bỏ sót / thừa). Script **không** kiểm được: box có ôm sát vật không (box truck hụt đuôi ở `img_002` vẫn qua script); vật có thật không (tượng, phản chiếu); lớp có đúng thực tế không (pickup gán `van` vẫn qua script); vật cả hai người cùng bỏ sót; hai người cùng hiểu sai một quy tắc. Những phần này do người review đọc ảnh, đối chiếu R-xx.

**Tái lập trên máy khác.**
- Các file bị khoá SHA-256 (`iaa_summary.json`, `batch_manifest.json`, `reviews/round-*`) được sinh trên Windows. Lúc đầu git tự đổi xuống dòng CRLF ↔ LF khi commit / clone, làm lệch mã băm: một bản clone mới báo "Đầu vào IAA đã thay đổi".
  - Đã sửa bằng `.gitattributes` (`submission/**`, `cvat/**`, `data/ATTRIBUTION.csv` giữ nguyên byte).
  - Đã kiểm: clone mới với `core.autocrlf=true` và `=false` đều chạy `check_submission.py --round 2` khớp mã băm.
- **Giới hạn còn lại:** script lưu đường dẫn trong manifest bằng dấu `\` của Windows (ví dụ `submission\taxonomy.json`), nên chạy lại trên Linux / macOS sẽ không tìm thấy file. Nên mở và chạy lại bài trên Windows. Sửa tận gốc (đổi `rel()` sang `as_posix()`) đòi hỏi sinh lại toàn bộ chuỗi IAA → calibration → QC, nên nhóm chưa làm.

Ngoài ra, nhóm tự viết thêm một bước kiểm không có trong script: so khít toạ độ box giữa hai file nhãn, để phát hiện box nạp lại từ file thay vì vẽ độc lập. Bước này quyết định phạm vi lô 8 ảnh ở mục 2.
