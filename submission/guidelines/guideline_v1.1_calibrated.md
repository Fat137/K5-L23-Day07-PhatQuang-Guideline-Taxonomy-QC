# Guideline gán nhãn — v1.1 (sau calibration) — Nhóm PhatQuang

- Phiên bản: **v1.1** (sau calibration ngày 2026-11-17)
- Ngày: 2026-11-17
- Người viết / duyệt: thành viên 063, thành viên 200
- Nguồn: `guideline_v1.0_agreed.md` + quyết định trong `submission/qc/calibration_decisions.csv`
- Taxonomy đi kèm: `submission/taxonomy.json` version `v1.0` (chưa đổi)
- Loại nhãn: bounding box, CVAT, xuất COCO 1.0; lô `day07-street-40`


## 0. Mã quy tắc (R-xx)

Mã dùng trong `calibration_decisions.csv` (cột `quy_tac_ghi_vao_guideline`) và phiếu QC (cột `rule_id`).
Bảng này chỉ **đặt mã** cho các quy tắc đã có ở v1.0, không đổi nghĩa quy tắc nào.

| Mã | Quy tắc | Mục ở v1.0 |
|---|---|---|
| R-01 | Chỉ gán vật thật. Tượng, bia mộ, ma-nơ-canh, mascot, đồ chơi / mô hình, hình in / poster / tranh tường / ảnh dán: không gán | 3.8 |
| R-02 | Hình phản chiếu (gương, kính, mặt đường ướt): không gán | 3.10 |
| R-03 | Xe chủ (xe mang camera): không gán | 3.11 |
| R-04 | Xe có cabin tách khỏi thùng hàng, **mọi pickup**, xe tải mini là `truck` | 2.1, D1 |
| R-05 | Xe 3 bánh (tuk-tuk, auto-rickshaw, xích lô) là `tricycle`; mô tô có sidecar là `motorcycle` | 2.1, D2 |
| R-06 | Thân một khối: > 16 chỗ là `bus`; hộp ≤ 16 chỗ là `van`; còn lại `car` | 2.1 |
| R-07 | 2 bánh có động cơ là `motorcycle`; đạp chân là `bicycle` | 2.1 |
| R-08 | Người và xe là hai box riêng; mỗi người trên xe một box `person` | 3.4 |
| R-09 | Người trong xe chỉ gán khi thân người không bị kính / vách xe ngăn (tuk-tuk, xích lô, mui trần: gán) | 3.5, D3, D4 |
| R-10 | Box ôm sát phần nhìn thấy; vật bị che chia mảnh thì một box ôm các mảnh | 3.1, 3.2 |
| R-11 | Box xe gồm thân, gương, thùng, giàn, sidecar; không gồm hàng chở thêm nhô ra | 3.6 |
| R-12 | Đầu kéo + rơ-moóc / giàn là một box; xe nằm trên xe chở xe vẫn gán từng chiếc | 3.7 |
| R-13 | Xe thật đang đỗ / trưng bày vẫn gán | 3.9, D5 |
| R-14 | `occluded_level`: < 25 % / 25–50 % / 50–75 % / ≥ 75 % phần trong khung bị vật khác che | 2.2, D6 |
| R-15 | `truncated = yes` khi box cách mép ảnh ≤ 3 px và vật còn phần ngoài khung | 2.2, D7 |
| R-16 | `can_review` khi không chắc lớp / chưa có quy tắc / không chắc vật thật | 2.2 |
| R-17 | Vật cao < 20 px không gán riêng (≥ 3 vật nhỏ sát nhau thì một `ignore_region`); vật lộ < 10 % thân không gán | 3.12, 3.13 |
| R-18 | Cụm ≥ 3 vật chồng > 50 %, không tách được thì `ignore_region`; vật tách được vẫn gán riêng; ignore không phủ box đã gán | 3.14, D8 |
| R-19 | Nhoè: nhận ra lớp thì gán, bóng mờ không đếm được thì `ignore_region`; ảnh tối chỉ gán vật nhận ra lớp | 3.15, 3.16 |
| R-20 | Ảnh không có vật cần gán để trống, vẫn nằm trong file nhãn | 3.17 |

## 0b. Bổ sung sau calibration

Quy tắc mới từ buổi calibration. Mỗi quy tắc dẫn tới ca trong `submission/qc/calibration_decisions.csv`.

| Mã | Quy tắc mới | Ca (case_id) | Ảnh ảnh hưởng |
|---|---|---|---|
| **R-21** | **`occluded_level` của người bị bàn, ghế, xe hay người khác che.** Ước trên cả thân người (đứng hoặc ngồi) như khi không bị che; bàn, ghế, quầy, xe phía trước đều tính là vật che. Mốc nhanh: thấy gần đủ người (kể cả bàn chân) → `0`; mất bàn chân / cẳng chân → `1`; chỉ thấy từ hông hoặc đùi trở lên → `2`; chỉ thấy đầu và vai → `3`. | C002–C006 | Mọi `person` bị che trong lô (`pham_vi = lo`), đặc biệt `img_001` |
| **R-22** | **Khung thưa** (giàn xe chở xe, song rào, thang): tính % theo diện tích thanh thực sự che thân xe. Nếu chỉ có thanh thưa, vẫn nhìn rõ hình dáng xe qua khe → **tối đa mức `1`**. Bị xe khác hoặc khối đặc che thêm thì cộng phần đó theo R-14. | C007–C012 | `img_002` |
| **R-23** | **Hàng / xe chở trên xe không tính là "che" xe chở** khi đặt `occluded_level` của xe chở (hai người đã cùng làm vậy ở `img_002`; nay ghi thành quy tắc). | — (thống nhất sẵn) | `img_002`, `img_001`, `img_028`, `img_035` |

**Ca "cùng sai" phát hiện khi rà nhãn đồng thuận theo R-xx.** IAA không bắt được các ca này vì hai người giống nhau:
- `N01` (`img_002`): cả hai cùng vẽ van xa cao 17 px, trái R-17 → xoá.
- `N02` (`img_001`): truck bị mép phải ảnh cắt nhưng cả hai để `truncated = no`, trái R-15 → `yes`.
- `N03` (`img_016`): pickup Mighty-X bị cả hai gán `van`, trái R-04 → `truck`, `truncated = yes`.
- `N04` (`img_004`): cả hai bỏ sót tài xế tuk-tuk, trái R-09 → thêm `person`.
- `N05` (`img_016`): cả hai bỏ sót người trong tuk-tuk trái, trái R-09 → thêm `person`. `occluded_level` của tuk-tuk đó giữ `2`, vì người ngồi trong xe che phần lớn khoang xe (v1.0 mục 2.2: người ngồi trên xe che xe tính là che).

Không có quy tắc mới từ các ca này. Quy tắc đã rõ, nên lỗi là do cả hai người cùng áp sai. Kiểu lặp lại rõ nhất là **quên người trong xe 3 bánh** (R-09: `img_004`, `img_016`). Cách phòng: trước khi export, mỗi người rà một vòng theo R-04 (mọi pickup), R-09 (mọi tuk-tuk / xích lô có người) và R-15 (mọi box chạm mép ảnh).

**Còn để ngỏ** (không có ảnh nào trong lô so sánh nên chưa chốt): đồ người mang theo (túi xách thõng, ô) có tính vào box `person` không (`img_014`). Tạm theo R-10: chỉ ôm thân người và đồ đang mặc / đeo sát người.

---

## 1. Mục tiêu

Nhãn huấn luyện mô hình **phát hiện người và phương tiện thật trên đường phố** (camera giao thông, cảnh báo va chạm) ở đô thị châu Á, nơi có nhiều xe máy, xe ba bánh, xe đạp và đám đông.

| Thứ tự ưu tiên | Lỗi | Vì sao nguy hiểm |
|---|---|---|
| 1 | **Bỏ sót người** hoặc **xe ở gần** | Mô hình không cảnh báo, gây va chạm |
| 2 | **Gán vật không thật** (tượng, đồ chơi, hình in, phản chiếu) | Mô hình báo động giả, phanh vô cớ |
| 3 | Sai lớp giữa các loại xe | Sai thống kê lưu lượng, sai dự đoán quỹ đạo |
| 4 | Box lỏng / lệch, sai thuộc tính | Giảm độ chính xác định vị |

Hệ quả:
- Vật thật ở gần mà phân vân thì **gán** và tick `can_review`.
- Vật quá nhỏ / dày / nhoè thì đưa vào `ignore_region`, không vẽ ẩu.

## 2. Lớp và thuộc tính

### 2.1 Lớp (9 lớp, mỗi vật đúng một lớp)

Phép thử lớp / thuộc tính: *"Mô hình có phải đoán ra điều này không?"* Có thì là lớp. Chỉ để lọc / QC thì là thuộc tính.

| Lớp | Định nghĩa (một câu) | Ví dụ trong lô | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe 4 bánh chở người cỡ nhỏ, khoang khách và khoang hành lý **liền một khối**: sedan, hatchback, SUV, wagon, taxi, xe mui trần. | Taxi `img_006`, `img_040`; SUV `img_012`, `img_028`; Civic `img_023`; xe Dangermouse `img_019`; ô tô trên giàn `img_002`; xe trưng bày `img_024` | Có **thùng hàng tách khỏi cabin** thì là `truck`. Đồ chơi / mô hình (`img_038`, xe vàng trên nóc `img_024`): không gán. |
| `truck` | Xe có **cabin tách khỏi thùng hàng** (thùng hở, mui bạt, thùng kín, giàn chở xe), **gồm mọi loại pickup** và xe tải mini. | ELF `img_007`; xe chở xe `img_002`; Suzuki Carry `img_001`; Hilux `img_007`, `img_012`; Mighty-X `img_016`; pickup cổ `img_028`; Veryca `img_036`; pickup có truck cap `img_037`; songthaew `img_008` | Van có thân một khối liền. Bia mộ hình pickup `img_027`: không gán. |
| `van` | Xe thân hộp **một khối liền** (cabin và khoang sau chung một vỏ), **≤ 16 chỗ**. | Hiace `img_005`, `img_016`; xe cứu thương `img_011` | Coaster `img_009` là `bus`; SUV là `car`. |
| `bus` | Xe chở khách **> 16 chỗ**: minibus cỡ Coaster trở lên, xe buýt, xe khách. | `img_009`, `img_031`, `img_037`, `img_040` | Van ≤ 16 chỗ. |
| `motorcycle` | Xe **2 bánh có động cơ** (xe máy, tay ga, mô tô, xe máy điện), **gồm cả sidecar / thùng kéo gắn liền**. | `img_003`, `img_008`, `img_021`, `img_030`; xe máy điện `img_029`; sidecar `img_022`; xe kéo thùng `img_025`; xe trang trí `img_011` | Xe đạp là `bicycle`; xe 3 bánh là `tricycle`. |
| `bicycle` | Xe **2 bánh đạp bằng chân**. | `img_003`, `img_006`, `img_010`, `img_029`, `img_034`, `img_039` | Xích lô (3 bánh) là `tricycle`; xe máy điện không bàn đạp là `motorcycle`. |
| `tricycle` | Xe **3 bánh** chở người / hàng, **có hoặc không có động cơ**: tuk-tuk, auto-rickshaw, xích lô. | Tuk-tuk `img_004`, `img_016`, `img_040`; auto-rickshaw `img_035`; xích lô `img_023`, `img_039` | Mô tô có sidecar là `motorcycle` (xe 2 bánh + thùng phụ). |
| `person` | **Người thật** đi bộ, đứng, ngồi, hoặc ngồi trên xe mà **thân người lộ ra ngoài, không bị kính / vách xe ngăn** (xe máy, xe đạp, xích lô, tuk-tuk hở hai bên, ô tô mui trần). | `img_001`, `img_014`, `img_032`; người lái xe máy `img_005`; tài xế tuk-tuk `img_004`; tài xế mui trần `img_019` | Tượng `img_017`; ma-nơ-canh `img_033`; mascot `img_019`; hình in / poster / ảnh dán `img_018`, `img_030`, `img_036`, `img_040`; người **sau kính** xe kín `img_009`. |
| `ignore_region` | **Vùng** chứa người / xe quá nhỏ, quá dày đặc hoặc quá nhoè để vẽ từng box. Bỏ sót trong vùng này không tính lỗi. Không có thuộc tính. | Khán giả `img_019`; bãi xe phía sau `img_030`; dòng xe xa `img_021`; xe nhoè `img_014` | Không dùng để né một vật to, rõ mà mình ngại phân lớp: gán lớp gần nhất + `can_review`. |

**Kiểm tra lớp xe nhanh (theo thứ tự):**
1. Mấy bánh? 2 bánh: có động cơ thì `motorcycle`, đạp chân thì `bicycle`. 3 bánh thì `tricycle`. Ngoại lệ: mô tô có sidecar là `motorcycle`.
2. 4 bánh trở lên: cabin tách khỏi thùng hàng thì `truck`.
3. Thân một khối: > 16 chỗ thì `bus`; dạng hộp cao ≤ 16 chỗ thì `van`; còn lại `car`.

**Vật không thuộc lớp nào** (chó `img_009`, ngựa / tượng ngựa `img_017`, `img_028`, xe đẩy, cọc giao thông): không gán, không vẽ `ignore_region`.

### 2.2 Thuộc tính (mọi lớp trừ `ignore_region`)

**`occluded_level`** (`0`–`3`): phần trăm diện tích **phần vật nằm trong khung ảnh** bị **vật khác** che. Phần ra ngoài khung ảnh không tính (đó là `truncated`). Ước bằng mắt theo phần tư.

| Mức | Bị che | Ví dụ |
|---|---|---|
| `0` | < 25 % | Người đi bộ `img_014`; Veryca `img_036` |
| `1` | 25 % đến < 50 % | SUV đỏ sau pickup `img_028`; xe sau hàng rào song thưa `img_018` |
| `2` | 50 % đến < 75 % | Xe máy giữa dãy đỗ `img_008`, `img_010` |
| `3` | ≥ 75 %, vẫn nhận ra lớp | Chân người giữa hai xe `img_007`; xe máy sau người lái `img_005` |

- Vật mảnh (song rào, dây, cột) chỉ tính phần thực sự che. Song rào thưa mà vẫn nhìn rõ vật thì tối đa mức `1`.
- Người ngồi trên xe che xe, và xe che chân người, đều tính là che.

**`truncated`** (`no` / `yes`): `yes` khi box **chạm mép ảnh** (cách mép ≤ 3 px) **và** vật rõ ràng còn phần nằm ngoài khung. Ngược lại là `no`.

**`can_review`** (checkbox): tick khi (a) không chắc lớp, (b) guideline chưa có quy tắc cho ca này, hoặc (c) không chắc vật thật / không thật. Ghi số ảnh + mô tả ra giấy để mang ra calibration. **Không** tick chỉ vì box khó vẽ.

## 3. Quy tắc vẽ box

1. **Box ôm sát phần nhìn thấy** (modal), không đoán phần bị che. Sai số khoảng 2 px mỗi cạnh với vật cao ≥ 50 px.
2. **Vật bị che chia thành nhiều mảnh:** một box ôm tất cả các mảnh của cùng một vật.
3. **Vật bị cắt mép ảnh:** box kéo tới đúng mép ảnh và đặt `truncated = yes`.
4. **Người và xe là hai box riêng.**
   - Mỗi người lái / ngồi sau là một box `person`, từ mũ tới chân nhìn thấy.
   - Box xe ôm thân xe, bánh, tay lái, gương; không cố tình gồm người.
   - Hai người trên một xe máy (`img_010`, `img_026`) là 2 `person` + 1 `motorcycle`.
5. **Người trong xe:** chỉ gán khi thân người **lộ ra ngoài, không bị kính / vách xe ngăn**.
   - Có gán: tài xế tuk-tuk / auto-rickshaw (cabin hở hai bên), người trên xích lô, tài xế xe mui trần.
   - Không gán: người ngồi sau kính car / van / truck / bus.
6. **Phụ kiện và hàng hoá:**
   - Box xe gồm thân, gương, giá nóc, thùng hàng, nắp thùng, giàn chở xe, sidecar, thùng kéo gắn liền.
   - Box xe **không gồm** hàng chở thêm nhô ra ngoài thân / thùng: ống thép `img_001`, ống nhôm `img_029`, bao hàng trên nóc `img_035`, tượng ngựa `img_028`.
7. **Đầu kéo + rơ-moóc / giàn** là một box (`img_002`). **Xe nằm trên xe chở xe** vẫn gán, mỗi xe một box `car`, đặt `occluded_level` theo phần bị giàn che.
8. **Vật không thật — không gán:** tượng (`img_017`, `img_013`), bia mộ hình xe (`img_027`), ma-nơ-canh (`img_033`), mascot / hình nộm (`img_019`, `img_011`), đồ chơi / mô hình (`img_038`, xe vàng trên nóc `img_024`), hình in / poster / tranh tường / ảnh dán (`img_018`, `img_026`, `img_030`, `img_031`, `img_036`, `img_040`).
9. **Xe thật đang đỗ hoặc trưng bày vẫn gán** (bãi xe `img_018`, bảo tàng `img_024`, xe trang trí trên thảm `img_011`), vì mô hình phải nhận ra xe bất kể xe ở đâu.
10. **Phản chiếu — không gán:** gương chiếu hậu (`img_020`), mặt đường ướt (`img_007`, `img_013`, `img_015`, `img_031`), kính cửa hàng (`img_033`), kính xe (`img_024`).
11. **Xe chủ** (xe mang camera: cửa, gương, capo trong khung `img_020`): không gán.
12. **Vật lộ quá ít:** chỉ thấy < 10 % thân vật (ví dụ chỉ thấy gương xe máy `img_040`) thì không gán.
13. **Vật nhỏ:** box cao < **20 px** thì không vẽ box riêng. ≥ 3 vật nhỏ nằm sát nhau thì vẽ **một `ignore_region`** bao cả cụm. Vật nhỏ lẻ loi thì bỏ qua.
14. **Đám đông / dãy xe dày:**
    - Vật cao ≥ 20 px mà **tách được ranh giới** với vật bên cạnh thì gán từng box.
    - Cụm **≥ 3 vật** chồng lấn quá một nửa, không tách được ranh giới, thì một `ignore_region` cho cụm đó.
    - Vật phía trước đã tách được thì vẫn gán riêng, kể cả khi nằm cạnh vùng ignore.
15. **Nhoè chuyển động:**
    - Nhận ra lớp và ước được thân vật thì gán; box ôm phần thân đậm nhất, không ôm vệt mờ.
    - Bóng mờ bán trong suốt, không đếm được mấy vật, thì `ignore_region` (`img_014`, `img_015`).
16. **Ảnh tối:** nhận ra lớp thì gán bình thường (`img_007`, `img_013`, `img_031`); không nhận ra thì bỏ qua.
17. **Ảnh không có vật cần gán** (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`): để trống, vẫn tính là đã gán.

## 4. Ca khó

### 4.1 Các chỗ hai bản cá nhân khác nhau và quyết định chốt

| # | Vấn đề | Bản 063 | Bản 200 | **Chốt v1.0** | Lý do |
|---|---|---|---|---|---|
| D1 | Pickup thùng hở (`img_012`, `img_016`, `img_028`) | `truck` | `car` (chỉ pickup gắn thùng kín là `truck`) | **`truck` cho mọi pickup** | Một phép thử duy nhất "cabin tách thùng?", không phải đoán thùng cao hay thấp. Bản 200 làm `img_007` / `img_037` thành ca biên khó đồng thuận |
| D2 | Xe 3 bánh | `three_wheeler` (có động cơ); xích lô là `bicycle` | `tricycle` gồm cả tuk-tuk lẫn xích lô | **`tricycle`, đếm bánh** | Đếm bánh nhìn là thấy; phân biệt có / không động cơ trên ảnh khó hơn (xích lô máy, xe ba gác) |
| D3 | Tài xế trong tuk-tuk (`img_004`, `img_035`) | gán | không gán (xe có mái) | **gán** | Tiêu chí chung "thân người lộ ra, không bị kính / vách ngăn"; người là ưu tiên 1 của mục tiêu |
| D4 | Tài xế xe mui trần `img_019` | không gán | gán | **gán** | Cùng tiêu chí D3 |
| D5 | Xe máy trang trí trưng bày `img_011` | `motorcycle` + `can_review` | không gán | **`motorcycle`** | Xe thật (có bánh, động cơ), đỗ ở đâu vẫn là xe; nhất quán với xe trưng bày `img_024` |
| D6 | Thang `occluded_level` | 10 / 40 / 70 % | 25 / 50 / 75 % | **25 / 50 / 75 %** | Ước theo phần tư dễ đồng thuận hơn bằng mắt |
| D7 | `truncated` | chạm mép ảnh + vật còn ngoài khung | ≥ 10 % vật ngoài khung | **chạm mép ≤ 3 px + vật còn ngoài khung** | Kiểm được trên toạ độ box, không phải ước % |
| D8 | Ngưỡng đám đông dùng `ignore_region` | ≥ 5 vật | ≥ 3 vật chồng > 50 % | **≥ 3 vật chồng > 50 %** | Rõ hơn, dùng được cho cả dãy xe máy nhỏ |
| D9 | Songthaew `img_008`, pickup có truck cap `img_037` | `truck` (chưa chốt) | `truck` (chưa chốt) | **`truck`** | Theo D1 |

### 4.2 Ca khó theo ảnh (áp dụng v1.0)

| Ảnh | Tình huống | Quyết định v1.0 |
|---|---|---|
| img_001 | Suzuki Carry chở ống thép; người ngồi quán; biển quảng cáo | `truck` (không gồm ống); người ngồi quán là `person`; biển: không |
| img_002 | Xe chở 9 ô tô; người đứng sau | 1 `truck` (cabin + giàn); mỗi ô tô một `car`; `person` |
| img_003, img_010 | Xe đạp chở hàng rong; nón lá gánh hàng; dãy xe máy đỗ xa | `bicycle`; `person` (không gồm gánh); dãy xa là `ignore_region` |
| img_004, img_016, img_040 | Tuk-tuk có tài xế | `tricycle`; tài xế là `person` (D3) |
| img_005, img_021 | Người ngồi sau che người lái; xe máy bị che gần hết | Mỗi người một box; xe máy bị che ≥ 75 % là mức 3 |
| img_007 | Kho tối; Hilux gắn thùng; chân người giữa hai xe; sàn ướt | 2 `truck` (ELF `truncated = yes`); chân người là `person` mức 3; phản chiếu: không |
| img_008 | Songthaew xa; xe máy đỗ dày; mascot ở cửa hàng | Songthaew là `truck`; xe máy trước gán từng chiếc, dãy sau `ignore_region`; mascot: không |
| img_009 | Coaster; khách sau kính; người ban công; chó | `bus`; khách sau kính: không; người ban công là `person` nếu cao ≥ 20 px; chó: không |
| img_011 | Tượng Predator; xe máy trang trí; xe cứu thương; đám đông xa | Tượng: không; xe trang trí là `motorcycle` (D5); `van`; đám đông là `ignore_region` |
| img_012 | Thùng pickup tiền cảnh (không phải xe chủ) | `truck`, `truncated = yes` |
| img_013 | Đêm; tượng rồng vàng; phản chiếu; xe xa nhỏ | `person` thật; tượng / phản chiếu: không; cụm xe < 20 px là `ignore_region` |
| img_014 | Người đi bộ rõ; xe máy nhoè thành bóng | 2 `person`; vùng nhoè là `ignore_region` |
| img_015 | Xe cắt mép; xe nhoè chồng nhau; nền ướt | `car` `truncated = yes`; bóng nhoè là `ignore_region`; phản chiếu: không |
| img_017 | Tượng người cưỡi ngựa | Không gán |
| img_018 | Biển in hình người; xe sau hàng rào; biển "Ben Car" che xe | Hình in: không; `car` / `truck` mức 1 do song rào; xe bị biển che theo % thực tế |
| img_019 | Xe Dangermouse mui trần; mascot; tài xế; khán giả | `car`; tài xế là `person` (D4); mascot: không; khán giả là `ignore_region` |
| img_020 | Xe trong gương chiếu hậu; xe chủ | Không gán |
| img_022 | Mô tô sidecar, ảnh trắng đen | 1 `motorcycle` gồm sidecar; người lái là `person` nếu cao ≥ 20 px |
| img_023, img_039 | Xích lô | `tricycle`; người đạp / khách là `person` |
| img_024 | Bảo tàng xe; mô hình vàng trên nóc; phản chiếu kính | Xe thật là `car`; mô hình: không; phản chiếu: không |
| img_025 | Xe máy kéo thùng; người xa bãi biển | 1 `motorcycle` gồm thùng; cụm người < 20 px là `ignore_region` |
| img_026 | 2 người trên xe máy; tranh tường cảng | 2 `person` + 1 `motorcycle`; tranh: không |
| img_027 | Bia mộ hình pickup | Không gán |
| img_028 | Pickup cổ chở tượng ngựa; SUV đỏ sau | `truck` (D1, không gồm tượng); SUV là `car` theo % bị che |
| img_029 | Xe máy điện chở ống nhôm; xe đạp | `motorcycle` (không gồm ống); `bicycle`; 2 `person` |
| img_030 | Bãi xe máy dày; poster; bảo vệ | Hàng trước gán từng chiếc; phía sau `ignore_region`; poster: không; bảo vệ là `person` |
| img_031 | Đêm mưa; xe buýt; tranh vẽ tường | `bus`; tranh: không; phản chiếu: không |
| img_032 | ~30 người chờ xe buýt sau dải bê tông | Người tách được gán từng người; cụm chồng > 50 % là `ignore_region` (D8) |
| img_033 | Ma-nơ-canh; poster; xe trắng phản chiếu trên kính | Không gán |
| img_035 | Auto-rickshaw chở bao hàng cao | `tricycle` (chỉ thân xe); tài xế là `person` (D3) |
| img_037 | Pickup có truck cap; xe buýt; người đi bộ | `truck` (D1); `bus`; `person` |
| img_038 | Tủ kính ô tô đồ chơi | Không gán |
| img_040 | Người trên xe máy chỉ thấy gương; ảnh dán kính taxi | `person` `truncated = yes`; xe máy không gán (quy tắc 12); ảnh dán: không; taxi là `car` |

## 5. QC và phiên bản

- **Ngưỡng QC** (`taxonomy.json`, mục `rules`, không đổi):
  - IoU từng box ≥ 0.85 với box cao ≥ 50 px, ≥ 0.7 với box cao 20–50 px;
  - IAA: IoU trung bình ≥ 0.85 và đồng thuận lớp ≥ 95 %;
  - Release khi tỉ lệ ảnh lỗi ≤ 2 %.
- **Khi gán (Pha 3):** guideline v1.0 đóng băng. Ca mơ hồ: tick `can_review`, không hỏi nhau, không tự đổi quy tắc.
- **Ai chốt ca mơ hồ:** cả hai thành viên tại buổi calibration. Mỗi quyết định một dòng trong `submission/qc/calibration_decisions.csv`. Quyết định thành quy tắc mới thì ghi vào `guideline_v1.1_calibrated.md`.
- **Phiên bản:**
  - Thay đổi làm đổi nhãn thì tăng phiên bản guideline.
  - Thay đổi lớp / thuộc tính thì tăng `version` trong `taxonomy.json`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên (6 lớp, thuộc tính không có ngưỡng) | — |
| v0.1-063, v0.1-200 | Hai bản cá nhân | Pha 1 |
| **v1.0** | Thêm lớp `bus`, `bicycle`, `tricycle` (9 lớp) | `img_004`, `img_009`, `img_023`, `img_029`, `img_035` |
| | Mọi pickup là `truck` | D1: `img_012`, `img_016`, `img_028`, `img_037` |
| | `occluded_level` theo 25 / 50 / 75 %; `truncated` theo mép ảnh | D6, D7 |
| | Người trong xe: gán khi không bị kính / vách ngăn | D3, D4: `img_004`, `img_019`, `img_035` |
| | Không gán vật không thật, phản chiếu, xe chủ; xe thật trưng bày vẫn gán | `img_011`, `img_017`, `img_020`, `img_024`, `img_033`, `img_038` |
| | `ignore_region` cho vật < 20 px và cụm ≥ 3 vật chồng > 50 %; nhoè không đếm được | D8: `img_014`, `img_019`, `img_021`, `img_030`, `img_032` |
| **v1.1** | Đặt mã R-01…R-20 cho quy tắc v1.0 (không đổi nghĩa); bổ sung quy tắc từ calibration (mục 0b) | `calibration_decisions.csv` |
