# Guideline gán nhãn — v0.1 (bản cá nhân) — 03063 - Phạm Tiến Phát

- Phiên bản: v0.1-03063 (viết lại từ v0, trước họp nhóm)
- Ngày: 2026-11-16
- Người viết: 03063 - Phạm Tiến Phát
- Loại nhãn: bounding box trên ảnh đường phố, công cụ CVAT, xuất COCO 1.0
- Phạm vi: lô `day07-street-40` (`img_001` … `img_040`)

> Các câu đánh dấu **[chưa chốt, cần họp]** là chỗ tôi chưa chắc, cần mang ra Pha 2.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người tham gia giao thông và phương tiện thật trên đường** (camera giao thông, hỗ trợ lái xe) ở đô thị châu Á, nơi có nhiều xe máy, xe ba bánh, xe đạp và đám đông.

Nguyên tắc chung rút ra từ mục tiêu:

1. **Chỉ gán vật thật có thể di chuyển trên đường.** Tượng, đồ chơi, ma-nơ-canh, hình in trên biển quảng cáo và hình phản chiếu **không gán**. Nếu dạy mô hình coi chúng là xe/người, mô hình sẽ báo động giả.
2. **Bỏ sót người hoặc xe ở gần là lỗi nguy hiểm nhất**, nặng hơn box hơi lỏng. Khi phân vân giữa "gán" và "không gán" một vật thật ở gần thì gán và tick `can_review`.
3. Vật quá nhỏ hoặc quá dày đặc, không vẽ từng box chính xác được, thì đưa vào `ignore_region` thay vì vẽ ẩu.

## 2. Lớp và thuộc tính

### 2.1 Lớp

Phép thử: *"Mô hình có phải đoán ra điều này không?"* Nếu có, đó là **lớp**. Nếu không (chỉ để lọc hoặc QC), đó là **thuộc tính**.
v0 chỉ có 5 lớp vật. Tôi đề xuất **thêm 3 lớp** (`bus`, `three_wheeler`, `bicycle`), vì trong 40 ảnh có nhiều vật thuộc ba loại này, mà ép chúng vào lớp cũ sẽ gây nhầm lẫn (xem bảng dưới).

| Lớp | Định nghĩa (một câu) | Ví dụ trong lô | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe 4 bánh chở người cỡ nhỏ, khoang hành khách và khoang hành lý liền một khối (sedan, hatchback, SUV, wagon, taxi, xe mui trần). | Mazda SUV `img_012`; taxi `img_006`, `img_040`; Honda Civic `img_023`; xe diễu hành Dangermouse có biển số `img_019`; các xe thật đặt trên giàn xe chở xe `img_002` | Xe có **thùng hàng hở tách khỏi cabin** là `truck` (pickup). Xe đồ chơi / mô hình `img_038`, xe mô hình trên nóc `img_024`: không gán. |
| `truck` | Xe chở hàng có **cabin tách khỏi thùng hàng** (thùng hở, thùng mui bạt, thùng kín, giàn chở xe), kể cả **pickup** và **xe tải mini**. | Isuzu ELF thùng kín `img_007`; xe chở xe Renault `img_002`; pickup Suzuki `img_001`, Hilux `img_007`, `img_012`, `img_016`; pickup cổ `img_028`; Veryca mui bạt `img_036`; pickup có nắp thùng cao `img_037` | Van: thân **một khối liền** từ đầu đến đuôi, không có thùng tách rời. Tượng pickup bằng đá `img_027`: không gán. |
| `van` | Xe thân hộp một khối liền (cabin và khoang sau chung một vỏ), cỡ nhỏ / trung, chở người hoặc hàng, **≤ 16 chỗ**. | Toyota Hiace `img_005`, `img_016`; xe cứu thương dạng van `img_011` | Minibus > 16 chỗ có nhiều hàng cửa sổ (Coaster `img_009`) là `bus`. SUV cao là `car`. |
| `bus` *(mới)* | Xe chở khách **> 16 chỗ** (minibus cỡ Coaster trở lên, xe buýt, xe khách). | Coaster `img_009`; xe buýt Đài Bắc `img_031`; buýt GO `img_037`; buýt Bangkok `img_040` | Van ≤ 16 chỗ (Hiace) là `van`. |
| `motorcycle` | Xe 2 bánh **có động cơ** (xe máy, xe tay ga, mô tô, xe máy điện), **kể cả sidecar hoặc xe kéo gắn liền**. | `img_003`, `img_006`, `img_008`, `img_021`, `img_030`; xe máy điện chở ống `img_029`; mô tô sidecar `img_022`; xe máy kéo thùng hàng `img_025` | Xe đạp (không động cơ) là `bicycle`. Xe ba bánh có cabin (tuk-tuk) là `three_wheeler`. |
| `three_wheeler` *(mới)* | Xe **3 bánh có động cơ** chở khách / hàng: tuk-tuk, auto-rickshaw. | Tuk-tuk `img_004`, `img_016`, `img_040`; auto-rickshaw chở quá tải `img_035` | Xích lô (đạp chân) là `bicycle`. Xe máy có sidecar là `motorcycle`. |
| `bicycle` *(mới)* | Xe **đạp bằng chân**, 2 hoặc 3 bánh, kể cả **xích lô** và xe đạp chở hàng rong. | `img_003`, `img_006`, `img_029`, `img_034`, `img_039`; xe đạp bán chuối `img_010`; xích lô `img_023` | Xe máy điện không có bàn đạp là `motorcycle`. |
| `person` | **Người thật** đứng, đi, ngồi, hoặc lái / ngồi trên xe **hở** (`motorcycle`, `bicycle`, `three_wheeler`). | Người đi bộ `img_014`; người ngồi quán `img_001`; người lái xe máy `img_005`; tài xế tuk-tuk `img_004` | Tượng `img_017`, ma-nơ-canh `img_033`, mascot `img_019`, hình in trên biển `img_018`, poster `img_030`, `img_036`, ảnh dán kính `img_040`: không gán. Người ngồi **trong** car/van/truck/bus: không gán. |
| `ignore_region` | **Vùng** (không phải vật) chứa vật thuộc các lớp trên nhưng không vẽ từng box được: quá nhỏ, quá dày đặc, quá nhoè. Bỏ sót trong vùng này không tính là lỗi. | Đám đông sau rào `img_019`; dãy xe máy dày đặc phía sau `img_030`; dòng xe xa `img_021` | Không dùng `ignore_region` để "né" một vật to, rõ mà mình ngại phân lớp: trường hợp đó gán lớp gần nhất và tick `can_review`. |

**Vật không thuộc lớp nào** (chó `img_009`, ngựa / tượng ngựa `img_017`, `img_028`, cọc giao thông, xe đẩy): **không gán, không vẽ `ignore_region`**.

### 2.2 Thuộc tính

Mọi lớp trừ `ignore_region` đều có 3 thuộc tính sau.

**`occluded_level`**: tỉ lệ diện tích **phần vật nằm trong khung ảnh** bị **vật khác** che. Phần bị cắt ra ngoài khung ảnh không tính ở đây (đó là `truncated`). Cách ước lượng: hình dung toàn bộ vật trong khung, rồi ước phần trăm diện tích bị che.

| Mức | Tỉ lệ bị che | Ví dụ |
|---|---|---|
| `0` | < 10 % | Hai người đi bộ `img_014`; xe tải Veryca `img_036` |
| `1` | 10 % đến < 40 % | Xe sau hàng rào song thưa `img_018`; tài xế tuk-tuk trước `img_004` |
| `2` | 40 % đến < 70 % | SUV đỏ sau pickup `img_028`; xe máy giữa hàng xe đỗ `img_008` |
| `3` | ≥ 70 % (vẫn nhận ra lớp) | Người chỉ lộ chân giữa hai xe `img_007`; xe máy chỉ thấy bánh / đầu xe `img_030` |

v0 định nghĩa mức 3 là "không rõ". Định nghĩa đó trộn hai ý (bị che nhiều và không chắc lớp), nên tôi bỏ. Không chắc lớp thì dùng `can_review`.
Vật mảnh như dây điện, song chắn, dây thừng chỉ tính phần thực sự che. Hàng rào song thưa mà vẫn nhìn rõ vật thì tối đa mức `1`.

**`truncated`**: `yes` khi box **chạm mép ảnh** (cách mép ≤ 3 px) **và** vật rõ ràng còn phần nằm ngoài khung. Ngược lại là `no`.
Ví dụ `yes`: xe đỏ bên trái và SUV bạc bên phải `img_015`; người áo đỏ góc phải `img_016`; xe tải ELF `img_007`.

**`can_review`**: tick khi (a) không chắc lớp, (b) guideline chưa có quy tắc cho ca này, hoặc (c) không chắc vật thật hay không thật. Ghi thêm một dòng mô tả để mang ra calibration. **Không** tick chỉ vì box khó vẽ.

## 3. Quy tắc vẽ box

1. **Box ôm sát phần nhìn thấy** (modal box). Không đoán phần bị che. Sai số cho phép khoảng 2 px mỗi cạnh với vật cao ≥ 50 px.
2. **Vật bị che tách làm nhiều mảnh nhìn thấy:** một box ôm tất cả các mảnh của **cùng một vật**.
3. **Vật bị cắt mép ảnh:** box kéo tới đúng mép ảnh và đặt `truncated = yes`.
4. **Người và xe là hai box riêng.**
   - Người lái hoặc người ngồi sau trên `motorcycle` / `bicycle` / `three_wheeler`: mỗi người một box `person`, ôm từ mũ bảo hiểm tới chân.
   - Box xe ôm thân xe, từ bánh tới tay lái / gương, **không** cố tình tính phần người.
   - Hai người trên một xe máy (`img_010`, `img_026`) thì có **2 box `person` + 1 box `motorcycle`**.
5. **Người bên trong xe kín** (car, van, truck, bus, kể cả xe mui trần `img_019`): **không gán**, vì mô hình chỉ cần thấy xe.
   - Tài xế và khách trong **tuk-tuk / auto-rickshaw** (cabin hở hai bên) **có** gán nếu thấy rõ từ đầu tới vai trở xuống (`img_004`, `img_035`).
6. **Phụ kiện và hàng hoá:**
   - Box xe gồm thân xe, gương, giá nóc, thùng hàng, và khung giàn chở xe gắn cố định.
   - Box xe **không** gồm hàng chở thêm nhô ra ngoài thân / thùng: ống thép trên pickup `img_001`, ống nhôm `img_029`, bao hàng chồng trên nóc rickshaw `img_035`, tượng ngựa trong thùng `img_028`.
7. **Đầu kéo và rơ-moóc / sidecar:** một xe gắn liền là **một box**.
   - Xe chở xe `img_002` là một box `truck` ôm cả cabin và giàn.
   - Mô tô sidecar `img_022` là một box `motorcycle`.
8. **Xe chở xe:** xe thật nằm trên giàn (`img_002`) **vẫn gán** `car`, mỗi xe một box, đặt `occluded_level` theo phần bị khung giàn che.
9. **Vật không thật — không gán:** tượng (`img_017`, `img_027`), ma-nơ-canh (`img_033`), mascot / hình nộm (`img_019`, `img_011`), đồ chơi / mô hình (`img_038`, xe vàng trên nóc `img_024`), hình in / poster / tranh tường (`img_018`, `img_026`, `img_030`, `img_031`, `img_036`, `img_040`).
10. **Phản chiếu — không gán:** gương chiếu hậu (`img_020`), mặt đường ướt (`img_013`, `img_015`, `img_031`), kính cửa hàng (`img_033`), cửa kính xe (`img_024`).
11. **Xe chủ (ego vehicle)**, tức xe mang camera, nhìn thấy cửa / gương / nắp capo (`img_020`): **không gán**.
12. **Vật quá nhỏ:** box cao < **20 px** thì không vẽ box riêng.
    - Nếu có ≥ 3 vật nhỏ như vậy nằm sát nhau, vẽ **một `ignore_region`** bao cả cụm.
    - Vật nhỏ lẻ loi thì bỏ qua.
13. **Đám đông / dãy xe dày đặc:**
    - Vật cao ≥ 20 px mà **tách được ranh giới** với vật bên cạnh thì gán từng box.
    - Khi ≥ 5 vật cùng lớp chồng lấn tới mức không xác định được ranh giới từng vật, vẽ **một `ignore_region`** cho phần đó. Vật phía trước đã tách được thì vẫn gán riêng.
    - Áp dụng cho `img_019` (khán giả sau rào), `img_021` (dòng xe phía xa), `img_030` (bãi xe phía sau), `img_003` / `img_036` (dãy xe đỗ xa).
14. **Nhoè do chuyển động:**
    - Vẫn nhận ra lớp và ước được thân vật thì gán; box ôm phần thân đậm nhất, không ôm vệt mờ kéo dài.
    - "Bóng ma" bán trong suốt, không nhận ra là mấy xe / mấy người, thì vẽ `ignore_region`: `img_014` (dòng xe máy nhoè), `img_015` (xe đen nhoè chồng lên xe trắng).
15. **Ảnh tối:** nhận ra lớp thì gán bình thường (`img_007`, `img_013`, `img_031`). Không nhận ra thì không gán và không vẽ ignore.
16. **Ảnh không có vật cần gán** (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`): để trống, vẫn tính là đã gán.

## 4. Ca khó

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| img_001 | Pickup Suzuki mini chở ống thép; người ngồi quán bị bàn che; biển quảng cáo hot dog | Pickup là `truck` (ống thép không tính vào box). Người ngồi quán là `person`, `occluded_level` theo bàn / người che. Biển: không gán | Pickup có thùng tách cabin; hàng nhô ra không phải thân xe |
| img_002 | Xe chở 9 ô tô | 1 box `truck` ôm cabin + giàn; mỗi ô tô thật trên giàn một box `car` (bị giàn che thì mức 1–2); người đứng sau xe là `person` | Ô tô trên giàn là vật thật; mô hình phải nhận ra chúng |
| img_003, img_010 | Xe đạp chở hàng rong; người đội nón lá gánh hàng | Xe đạp là `bicycle`; người là `person`; gánh hàng không tính vào box người | v0 không có lớp xe đạp |
| img_004, img_016, img_040 | Tuk-tuk | `three_wheeler`; tài xế trong cabin hở là `person` | Hình dáng khác hẳn xe máy và ô tô; gộp vào lớp nào cũng làm nhiễu |
| img_005, img_021 | Người ngồi sau che người lái; xe máy chỉ lộ một phần | Mỗi người một box; xe máy bị người che ≥ 70 % là mức 3 | Theo quy tắc 4 và thang che |
| img_007 | Kho tối; Hilux gắn thùng kín; người lộ chân giữa hai xe; sàn ướt phản chiếu | Hilux là `truck`; ELF là `truck` (`truncated = yes`); chân người cao ≥ 20 px là `person` mức 3; phản chiếu: không | Quy tắc 6, 10 |
| img_008 | Songthaew (pickup cải tạo chở khách) ở xa; xe máy đỗ dày | Songthaew là `truck` **[chưa chốt, cần họp]**; xe máy phía trước gán từng chiếc, dãy phía xa là `ignore_region` | Songthaew vẫn là pickup có thùng tách, nhưng chức năng là chở khách |
| img_009 | Minibus Coaster; khách thấy qua cửa sổ; người trên ban công; chó | Coaster là `bus`; khách trong xe: không gán; người ban công cao ≥ 20 px là `person`; chó: không | Quy tắc 5; vật ngoài taxonomy |
| img_011 | Tượng Predator khổng lồ; mô tô trang trí bằng phế liệu trưng bày; đám đông xa | Tượng: không gán. Mô tô trang trí là `motorcycle` + `can_review` **[chưa chốt, cần họp: xe thật hay tác phẩm điêu khắc?]**; xe cứu thương là `van`; đám đông xa là `ignore_region` | Quy tắc 9; không chắc vật thật |
| img_012 | Thùng pickup chở hàng ở tiền cảnh, chiếm nửa dưới ảnh | `truck`, `truncated = yes` (không phải xe chủ: người chụp đứng sau xe) | Xe chủ chỉ là xe mang camera |
| img_013 | Đêm; người và xe phản chiếu trên đường ướt; tượng rồng vàng | Người thật là `person`; phản chiếu: không; tượng: không; xe xa < 20 px gom thành `ignore_region` | Quy tắc 10, 12 |
| img_014 | Hai người đi bộ rõ nét; xe máy chạy nhoè thành bóng mờ | 2 `person`; vùng nhoè là `ignore_region` | Quy tắc 14 |
| img_015 | Xe cắt mép trái / phải; xe nhoè chồng nhau; phản chiếu nền ướt | Xe đỏ và SUV bạc là `car`, `truncated = yes`; xe nhoè bán trong suốt là `ignore_region`; xe rõ phía sau là `car` | Quy tắc 3, 14 |
| img_017 | Tượng người cưỡi ngựa đội nón giao thông | Không gán gì | Tượng không phải người |
| img_018 | Biển bầu cử in hình người; xe đỗ sau hàng rào; chụp qua kính xe | Hình người in: không gán; các xe là `car` / `truck` (pickup), `occluded_level = 1` do song rào; xe bị biển "Ben Car" che ~40 % là mức 2 | Quy tắc 9; thang che với vật mảnh |
| img_019 | Xe Dangermouse diễu hành; mascot chuột; tài xế; đám đông sau rào | Xe là `car` (xe thật, có biển số); mascot: không; tài xế trong xe mui trần: không gán; đám đông là `ignore_region`; người áo phản quang bên phải là `person` | Xe giả tưởng nhưng chạy thật trên đường |
| img_020 | Ô tô trong gương chiếu hậu; thân xe chủ | Không gán gì | Phản chiếu + xe chủ |
| img_022 | Ảnh trắng đen; mô tô có sidecar, người lái bị kính chắn che | 1 `motorcycle` (gồm sidecar); người lái lộ đầu là `person` mức 2–3 nếu cao ≥ 20 px | Quy tắc 7 |
| img_023, img_039 | Xích lô | `bicycle`; người đạp là `person`; khách ngồi trước (nếu có) là `person` | Xích lô không có động cơ |
| img_024 | Bảo tàng ô tô; xe mô hình vàng trên nóc xe đen; phản chiếu trên kính | Xe thật đỗ trưng bày là `car`; xe vàng trên nóc (mô hình thu nhỏ): không gán **[chưa chốt, cần họp]**; phản chiếu: không | Quy tắc 9 |
| img_025 | Xe máy kéo thùng bán hàng; người nằm ghế xa bờ biển | Một box `motorcycle` gồm thùng kéo; người xa < 20 px gom thành `ignore_region` | Quy tắc 7, 12 |
| img_026 | Hai người trên xe máy, đêm, nhoè nhẹ; tranh tường cảng biển | 2 `person` + 1 `motorcycle`; tàu / xe trong tranh tường: không gán | Quy tắc 4, 9 |
| img_027 | Bia mộ hình xe pickup | Không gán | Tượng / bia mộ |
| img_028 | Pickup cổ chở tượng ngựa; SUV đỏ phía sau | Pickup là `truck` (không tính tượng ngựa); SUV là `car`, bị che ~50 % là mức 2 | Quy tắc 6 |
| img_029 | Xe máy điện chở ống nhôm dài; xe đạp | Xe máy điện là `motorcycle` (box không gồm ống); xe đạp là `bicycle`; 2 người lái là `person` | Có động cơ, không có bàn đạp |
| img_030 | Bãi xe máy dày đặc; poster người; bảo vệ | Xe máy hàng trước tách được thì gán từng chiếc; phần phía sau là `ignore_region`; poster: không; bảo vệ là `person` | Quy tắc 13 |
| img_031 | Đêm mưa; xe buýt; mascot vẽ trên tường; phản chiếu | Xe buýt là `bus`; tranh vẽ: không; phản chiếu: không | Quy tắc 9, 10 |
| img_032 | Khoảng 30 người xếp hàng ở trạm buýt, bị dải phân cách che chân | Người tách được (cao ≥ 20 px) gán từng người, mức che theo dải bê tông (thường 1); đoạn chồng lấn không tách được là `ignore_region` **[chưa chốt, cần họp: gán từng người hay cả cụm?]** | Đám đông nhưng người khá lớn |
| img_033 | Ma-nơ-canh; người trong poster; xe trắng có gương, có vẻ là phản chiếu trên kính | Không gán gì; xe trắng nếu là phản chiếu thì không gán + `can_review` | Quy tắc 9, 10 |
| img_035 | Auto-rickshaw chở bao hàng cao gấp đôi xe | `three_wheeler`, box chỉ ôm thân xe (không gồm bao); tài xế là `person` | Quy tắc 6 |
| img_037 | Pickup có nắp thùng kín cao (truck cap) | `truck` **[chưa chốt, cần họp: nhìn từ sau rất giống van]** | Vẫn là pickup có cabin tách thùng |
| img_038 | Tủ kính đồ chơi ô tô | Không gán | Đồ chơi |
| img_040 | Người đội mũ trên xe máy, chỉ thấy gương xe; ảnh dán trên kính taxi | Người là `person` (`truncated = yes`); xe máy chỉ lộ gương (< 10 % thân xe): không gán; ảnh dán: không; taxi là `car` | Quy tắc 9; vật lộ quá ít |

## 5. QC và phiên bản

- Ngưỡng QC lấy theo `submission/taxonomy.json`, mục `rules` (không đổi):
  - IoU từng box ≥ 0.85 với box cao ≥ 50 px, ≥ 0.7 với box cao 20–50 px;
  - IAA: IoU trung bình ≥ 0.85 và đồng thuận lớp ≥ 95 %;
  - tỉ lệ ảnh lỗi ≤ 2 % mới được Release.
- Ca mơ hồ khi gán: tick `can_review`, ghi ảnh + mô tả. Cả nhóm chốt ở buổi calibration; người chốt ghi vào `calibration_decisions.csv`.
- Mọi thay đổi làm đổi nhãn thì tăng phiên bản guideline; đổi lớp / thuộc tính thì tăng `version` trong `taxonomy.json`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1-03063 | Thêm lớp `bus`, `three_wheeler`, `bicycle`; pickup và xe tải mini xếp vào `truck` | `img_004`, `img_009`, `img_016`, `img_023`, `img_029`, `img_035`, `img_036`, `img_037` |
| | Định nghĩa `occluded_level` bằng % diện tích bị che (0 / <10, 1 / 10–40, 2 / 40–70, 3 / ≥70); tách khỏi `truncated` | Mức 3 "không rõ" của v0 không đo được |
| | Quy tắc không gán vật không thật, phản chiếu, xe chủ | `img_017`, `img_020`, `img_027`, `img_033`, `img_038` |
| | Quy tắc người–xe: 2 box riêng; không gán người trong xe kín | `img_005`, `img_009`, `img_019`, `img_026` |
| | Quy tắc vật nhỏ < 20 px, đám đông, nhoè dùng `ignore_region` | `img_014`, `img_019`, `img_021`, `img_030` |
