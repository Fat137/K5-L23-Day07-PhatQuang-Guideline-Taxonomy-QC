# Guideline gán nhãn — v0.1 (bản cá nhân) — 03200 - Nguyễn Văn Quang

- Phiên bản: v0.1-03200 (sửa từ v0, trước họp nhóm)
- Ngày: 2026-11-16
- Người viết: 03200 - Nguyễn Văn Quang
- Loại nhãn: bounding box, CVAT, xuất COCO 1.0; lô `day07-street-40`

> Chỗ ghi **(chưa chốt, cần họp)** là chỗ tôi chưa chắc.

---

## 1. Mục tiêu

Mô hình phát hiện **người và phương tiện đang tham gia giao thông** trên ảnh đường phố (camera giao thông, cảnh báo va chạm).

- Sai nguy hiểm nhất: **bỏ sót người** (đi bộ, đi xe máy, đi xe đạp) và **bỏ sót xe ở gần**.
- Sai thứ hai: **báo nhầm vật không thật** (tượng, hình in, đồ chơi), làm mô hình phanh hoặc cảnh báo vô cớ.
- Vì vậy guideline ưu tiên: gán đủ người và xe thật ở gần; không gán vật không thật; vùng quá nhỏ / dày đặc thì loại khỏi việc tính lỗi bằng `ignore_region`.

## 2. Lớp và thuộc tính

### 2.1 Lớp

Tôi chia lớp theo **hình dáng nhìn từ camera**, vì mô hình chỉ nhìn thấy hình dáng.
So với v0, tôi thêm 3 lớp: `bus`, `bicycle` và `tricycle`. Lớp `tricycle` gom mọi xe 3 bánh, cả có động cơ lẫn đạp chân.

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Xe 4 bánh chở người cỡ nhỏ: sedan, hatchback, SUV, taxi, **pickup cabin kép / cabin đơn dùng như xe con** (thùng hở, không gắn thùng kín). | Taxi `img_006`, `img_040`; SUV `img_012`, `img_028`; pickup Hilux `img_012`; pickup Mighty-X `img_016`; pickup cổ `img_028` | Pickup gắn **thùng kín / nắp thùng cao hơn nóc cabin** (`img_007`, `img_037`) là `truck`. |
| `truck` | Xe chở hàng: xe tải, xe tải mini đầu bằng, xe chở xe, và pickup đã gắn thùng kín cao hơn nóc cabin. | Isuzu ELF `img_007`; xe chở xe `img_002`; Suzuki Carry đầu bằng `img_001`; Veryca mui bạt `img_036`; Hilux gắn thùng `img_007`; pickup có truck cap `img_037` | Pickup thùng hở thông thường là `car` (xem trên). |
| `van` | Xe thân hộp liền một khối, ≤ 16 chỗ. | Hiace `img_005`, `img_016`; xe cứu thương `img_011` | Coaster `img_009` là `bus`. |
| `bus` | Xe chở khách > 16 chỗ. | `img_009`, `img_031`, `img_037`, `img_040` | Van. |
| `motorcycle` | Xe 2 bánh có động cơ, kể cả xe máy điện và **mô tô có sidecar**. | `img_003`, `img_008`, `img_021`, `img_029`, `img_022` | Xe đạp; xe 3 bánh. |
| `bicycle` | Xe đạp 2 bánh. | `img_006`, `img_010`, `img_029`, `img_034` | Xích lô (3 bánh) là `tricycle`. |
| `tricycle` | Xe **3 bánh** chở người / hàng, có hoặc không có động cơ: tuk-tuk, auto-rickshaw, xích lô. | Tuk-tuk `img_004`, `img_016`, `img_040`; auto-rickshaw `img_035`; xích lô `img_023`, `img_039` | Mô tô sidecar (`img_022`) vẫn là `motorcycle`, vì phần chính là xe 2 bánh. |
| `person` | Người thật: đi bộ, đứng, ngồi, hoặc ngồi trên xe **không có mui** (xe máy, xe đạp, xích lô, xe mui trần). | `img_001`, `img_014`, `img_032`; tài xế xe mui trần `img_019` | Tượng `img_017`; ma-nơ-canh `img_033`; mascot `img_019`; người trong poster / biển `img_018`, `img_030`, `img_036`, `img_040`. |
| `ignore_region` | Vùng chứa người / xe quá nhỏ, quá dày, quá nhoè để vẽ từng box. | Khán giả `img_019`; bãi xe `img_030`; dòng xe xa `img_021` | Không dùng cho vật to, rõ. |

**Vật ngoài các lớp** (chó, ngựa, tượng ngựa, xe đẩy, cọc giao thông): không gán.

### 2.2 Thuộc tính

**`occluded_level`**: tính theo **phần tư** diện tích phần vật nằm trong khung ảnh bị vật khác che (không tính phần ra ngoài khung):

| Mức | Bị che | Ví dụ |
|---|---|---|
| `0` | < 25 % | Người đi bộ `img_014`; Veryca `img_036` |
| `1` | 25 % đến < 50 % | SUV đỏ sau pickup `img_028`; xe sau hàng rào `img_018` |
| `2` | 50 % đến < 75 % | Xe máy giữa dãy đỗ `img_008`, `img_010` |
| `3` | ≥ 75 % | Chân người giữa hai xe `img_007`; xe máy sau người lái `img_005` |

Tôi chọn mốc 25 / 50 / 75 vì dễ ước bằng mắt ("che một phần tư, một nửa, ba phần tư").

**`truncated`**: `yes` khi ước lượng **≥ 10 %** vật nằm ngoài khung ảnh. Chỉ chạm mép rất ít thì vẫn `no`.

**`can_review`**: tick khi không chắc lớp hoặc không chắc vật thật / không thật.

## 3. Quy tắc vẽ box

1. Box ôm sát **phần nhìn thấy**, không đoán phần bị che.
2. Vật bị cắt mép ảnh: box kéo tới mép ảnh.
3. **Người trên xe và xe là 2 box riêng.** Mỗi người (lái, ngồi sau) một box `person`; box xe không cố tình gồm người.
4. Người **trong xe có mui** (car, van, truck, bus, **tuk-tuk / auto-rickshaw có mái**): không gán. Người trên xe **không mui** (xe máy, xe đạp, xích lô, ô tô mui trần): gán.
5. **Hàng chở trên xe** không tính vào box xe (ống `img_001`, `img_029`; bao hàng `img_035`). Thùng xe, giàn chở xe, sidecar, thùng kéo gắn liền thì tính.
6. **Xe trên xe chở xe** (`img_002`): mỗi ô tô một box `car`. Giàn chở và cabin là một box `truck`.
7. **Vật không thật — không gán:** tượng, bia mộ hình xe, ma-nơ-canh, mascot, đồ chơi, mô hình xe, hình in / poster / tranh tường, ảnh dán trên kính.
8. **Phản chiếu — không gán:** gương xe, kính cửa hàng, mặt đường ướt.
9. **Xe chủ** (xe mang camera) — không gán.
10. **Vật nhỏ:** box cao < 20 px thì không gán riêng; nhiều vật nhỏ sát nhau thì một `ignore_region`.
11. **Dày đặc:** khi ≥ 3 vật cùng lớp đè nhau hơn một nửa và không tách được ranh giới thì một `ignore_region` cho cả cụm. Vật phía trước tách được thì gán riêng.
12. **Nhoè chuyển động:** nhận ra lớp thì gán, box theo thân vật. Bóng mờ không đếm được thì `ignore_region`.
13. **Ảnh tối:** chỉ gán vật nhận ra được lớp.

## 4. Ca khó

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| img_001 | Xe tải mini Suzuki đầu bằng chở ống thép | `truck`, box không gồm ống | Xe tải mini, cabin đầu bằng |
| img_002 | Xe chở 9 ô tô | 1 `truck` + mỗi ô tô thật một `car` | Ô tô trên giàn vẫn là ô tô thật |
| img_004 | Tuk-tuk có tài xế ngồi trong cabin | `tricycle`; tài xế không gán vì xe có mái **(chưa chốt, cần họp)** | Quy tắc 4: người dưới mái coi như trong xe |
| img_007 | Kho tối; Hilux gắn thùng kín; chân người giữa hai xe | Hilux là `truck`; ELF là `truck`; chân người là `person` mức 3 | Pickup gắn thùng kín là xe chở hàng |
| img_011 | Tượng Predator; xe máy trang trí bằng phế liệu trên thảm trưng bày | Tượng: không gán. Xe trang trí: **không gán** vì là vật trưng bày, không tham gia giao thông **(chưa chốt, cần họp)** | Mục tiêu là phương tiện trên đường |
| img_012 | Pickup Hilux thùng hở chở đồ, cắt mép dưới | `car`, `truncated = yes` | Pickup thùng hở là `car` |
| img_014 | Xe máy chạy nhoè thành bóng mờ | `ignore_region`; 2 người đi bộ rõ là `person` | Quy tắc 12 |
| img_017 | Tượng người cưỡi ngựa | Không gán | Không phải người thật |
| img_019 | Xe Dangermouse mui trần; tài xế; mascot chuột | Xe là `car` + `can_review`; tài xế là `person` (xe không mui); mascot: không | Quy tắc 4, 7 |
| img_020 | Ô tô trong gương chiếu hậu | Không gán | Phản chiếu, xe chủ |
| img_022 | Mô tô có sidecar (ảnh cũ trắng đen) | 1 `motorcycle` gồm sidecar | Phần chính là xe 2 bánh |
| img_023, img_039 | Xích lô | `tricycle`; người đạp là `person` (không mui) | Cùng dáng 3 bánh với tuk-tuk |
| img_024 | Ô tô mô hình vàng trên nóc xe thật | Xe thật là `car`; mô hình: không gán | Quy tắc 7 |
| img_027 | Bia mộ hình pickup | Không gán | Quy tắc 7 |
| img_028 | Pickup cổ chở tượng ngựa | Pickup là `car` (thùng hở); tượng: không gán; SUV đỏ phía sau che 25–50 % là mức 1 | Quy tắc 5, 7 |
| img_030 | Bãi xe máy dày đặc | Hàng đầu gán từng chiếc; phía sau là `ignore_region` | Quy tắc 11 |
| img_032 | Khoảng 30 người chờ xe buýt | Gán từng người tách được; cụm chồng lấn là `ignore_region` | Quy tắc 11 |
| img_033 | Ma-nơ-canh, poster, xe trắng phản chiếu trên kính | Không gán gì | Quy tắc 7, 8 |
| img_035 | Auto-rickshaw chở bao hàng cao | `tricycle`, box chỉ ôm thân xe; tài xế: không gán (có mái) | Quy tắc 4, 5 |
| img_037 | Pickup gắn truck cap cao | `truck` **(chưa chốt, cần họp: có thể coi là car như pickup thường)** | Thùng kín cao hơn nóc cabin |
| img_038 | Tủ kính ô tô đồ chơi | Không gán | Đồ chơi |
| img_040 | Ảnh dán trên kính taxi; người lái xe máy chỉ thấy gương | Ảnh dán: không gán; người là `person`; xe máy chỉ thấy gương: không gán | Quy tắc 7; thấy quá ít |

## 5. QC và phiên bản

- Ngưỡng theo `taxonomy.json` (mục `rules`, không đổi):
  - IoU ≥ 0.85 với box cao ≥ 50 px, ≥ 0.7 với box cao 20–50 px;
  - IAA: IoU trung bình ≥ 0.85 và cùng lớp ≥ 95 %;
  - Release khi ảnh lỗi ≤ 2 %.
- Ca mơ hồ: tick `can_review`, cả nhóm chốt ở calibration, ghi vào `calibration_decisions.csv`.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản khởi đầu | — |
| v0.1-03200 | Thêm `bus`, `bicycle`, `tricycle` | `img_004`, `img_009`, `img_023`, `img_035` |
| | Pickup thùng hở là `car`; pickup gắn thùng kín là `truck` | `img_007`, `img_012`, `img_028`, `img_037` |
| | `occluded_level` theo phần tư (25 / 50 / 75 %); `truncated` khi ≥ 10 % ra ngoài khung | v0 không có ngưỡng |
| | Không gán vật không thật, phản chiếu, xe chủ; người dưới mái xe không gán | `img_004`, `img_017`, `img_020`, `img_033` |
