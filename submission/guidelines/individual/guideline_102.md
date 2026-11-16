# Guideline gán nhãn — Bản cá nhân (Pha 1)

- Phiên bản: v0-individual
- Người viết (mã thành viên): <102>
- Loại nhãn: bounding box 2D (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu mô hình 2D

Dữ liệu nhãn dùng để huấn luyện và đánh giá mô hình phát hiện vật thể 2D (**người và phương tiện giao thông cơ giới**) trong bối cảnh đường phố thực tế (camera giám sát giao thông / hỗ trợ lái xe)[cite: 10].
- **Phạm vi nhận diện:** Chỉ gán nhãn các đối tượng thật có mặt trực tiếp trong không gian vật lý 3D của khung cảnh.
- **Nguyên tắc chung:** Mọi quy tắc đều hướng tới tính nhất quán giữa các người gán; nếu mô hình 2D không cần suy luận khi chạy thực tế thì thông tin đó nằm ở thuộc tính thay vì tách lớp mới.

---

## 2. Lớp và thuộc tính

### 2.1. Định nghĩa lớp (mỗi đối tượng thuộc đúng một lớp)

| Lớp | Định nghĩa chuẩn | Ví dụ đạt chuẩn | Phản ví dụ (KHÔNG gán lớp này) |
|---|---|---|---|
| `car` | Ô tô con chở khách dưới 9 chỗ (sedan, hatchback, SUV, crossover, coupe, xe thể thao). | Ô tô con lưu thông trên đường hoặc đỗ ven đường; từng chiếc ô tô con nằm trên sàn xe tải chuyên dụng (`img_002.jpg`)[cite: 3]. | Xe bán tải (xếp vào `truck`), xe khách/xe buýt lớn (`img_009.jpg`), xe đồ chơi/mô hình (`img_038.jpg`)[cite: 3]. |
| `truck` | Xe tải chở hàng (tải nhẹ, tải nặng, xe đầu kéo, xe bồn, xe chuyên dụng chở ô tô, xe bán tải pickup). | Xe tải chở ô tô (`img_002.jpg`), xe bán tải thùng hở[cite: 3]. | Tượng xe bán tải (`img_027.jpg`), xe van thùng kín liền khối[cite: 3, 10]. |
| `van` | Xe thùng kín cỡ nhỏ/trung chuyên chở hàng hoặc chở người dưới 16 chỗ, khoang lái và khoang sau liền khối không có thùng rời. | Xe tải van giao hàng, xe khách 12–16 chỗ dáng liền khối. | Xe buýt cỡ lớn >16 chỗ (`img_009.jpg`, `img_031.jpg`), xe tải thùng rời[cite: 3]. |
| `bus` *(đề xuất bổ sung)* | Xe khách/xe buýt cỡ lớn (trên 16 chỗ, thân dài, cửa lên xuống hành khách lớn). | Xe buýt đô thị, xe khách đường dài (`img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg`)[cite: 3]. | Xe van cỡ nhỏ dưới 16 chỗ. |
| `tuk_tuk` *(đề xuất bổ sung)* | Xe cơ giới 3 bánh chở khách hoặc chở hàng (xe lam, tuk-tuk, xe ba gác máy). | Xe tuk-tuk 3 bánh trên phố (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`)[cite: 3]. | Xe đạp, xích lô đạp không động cơ (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`)[cite: 3]. |
| `motorcycle` | Xe cơ giới 2 bánh (xe máy, xe tay ga, mô tô phân khối lớn). | Xe máy đang chạy hoặc dựng trên vỉa hè. | Xe đạp thường, xe đạp điện không có kết cấu mô tô, người lái xe máy (gán riêng lớp `person`). |
| `person` | Con người thật ở mọi tư thế (đi bộ, đứng, ngồi, điều khiển phương tiện). | Người đi bộ, người ngồi trên xe máy/xe đạp, người ngồi trong ô tô nhìn rõ qua kính. | Tượng người (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh chân dung trên biển quảng cáo (`img_018.jpg`)[cite: 3]. |
| `ignore_region` | Vùng chứa đám đông quá dày đặc hoặc cụm nhiều đối tượng có chiều cao `< 20 px` không thể tách rời từng box chính xác[cite: 3]. | Đám người đứng xa ở nền ảnh, bãi đỗ xe máy san sát ở tầm rất xa. | Nhóm 2–4 người đứng gần nhau vẫn phân tách rõ ranh giới từng người. |

### 2.2. Thuộc tính với ngưỡng đo lường định lượng

*(Lưu ý thao tác CVAT: Chọn giá trị trong panel **Objects**, không dùng phím tắt `Q` của CVAT)*[cite: 3].

| Thuộc tính | Giá trị | Ngưỡng định lượng & Quy tắc áp dụng |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Đánh giá dựa trên **tỷ lệ diện tích ước tính bị vật khác che khuất** so với toàn bộ vật thể:<br>• `0`: Bị che `0%` (thấy trọn vẹn vật thể)[cite: 10].<br>• `1`: Bị che từ `1%` đến `30%` diện tích.<br>• `2`: Bị che từ `31%` đến `70%` diện tích.<br>• `3`: Bị che trên `70%` diện tích nhưng vẫn nhận diện chắc chắn lớp đối tượng. |
| `truncated` | `no`, `yes` | • `no`: Toàn bộ đối tượng nằm trọn bên trong khung ảnh.<br>• `yes`: Bất kỳ phần nào của đối tượng chạm và bị cắt bởi 1 trong 4 cạnh biên của ảnh[cite: 10]. |
| `can_review` | `false`, `true` | • Tick (`true`) khi đối tượng rơi vào trường hợp biên chưa có tiền lệ rõ ràng, mờ nhòe nặng hoặc phân vân giữa 2 lớp để đưa ra họp calibration[cite: 3, 10]. |

---

## 3. Quy tắc vẽ box và xử lý tình huống biên

1. **Nguyên tắc ôm sát phần nhìn thấy:**
   - Bounding box chỉ bao trọn **phần nhìn thấy được** của đối tượng, ôm sát đường viền ngoài cùng (sai số khoảng trống không quá 2–3 px)[cite: 3]. Không tự suy đoán kéo dài box sang phần đã bị vật khác che khuất hoặc vượt ra ngoài biên ảnh.
2. **Người đi xe máy (`person` + `motorcycle`):**
   - Tách thành **2 box độc lập**: 01 box `motorcycle` ôm sát thân xe (gồm bánh xe, khung, tay lái, gương chiếu hậu) và 01 box `person` ôm sát cơ thể người lái (tính cả mũ bảo hiểm, ba lô đeo trên lưng và phần chân/tay đang đặt trên xe).
   - Nếu có người ngồi sau: mỗi người ngồi sau là một box `person` riêng biệt.
3. **Xe chở xe (`truck` + `car`):**
   - Gán 01 box `truck` ôm trọn xe tải chuyên dụng đầu kéo/thùng chở.
   - Đồng thời gán **từng box `car` riêng biệt** cho mỗi chiếc ô tô con nằm trên sàn chở nếu thấy rõ hình dáng nhận diện được (ví dụ 8 xe con trên 2 tầng ở `img_002.jpg`), đặt `occluded_level` tương ứng với phần bị thành xe che[cite: 3].
4. **Vật thể quá nhỏ và đám đông:**
   - Đối tượng đơn lẻ có chiều cao box `< 20 px`: **bỏ qua không gán** (tránh nhiễu cho mô hình)[cite: 3].
   - Nếu có từ 5 đối tượng trở lên tụ tập dày đặc hoặc nhiều vật `< 20 px` nằm liền nhau: khoanh một vùng `ignore_region` bao trọn cụm đó[cite: 3].
5. **Đối tượng không có thật (tượng, ma-nơ-canh, tranh ảnh, phản chiếu):**
   - **Loại bỏ hoàn toàn** (không gán nhãn): tượng người/tượng xe (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), ma-nơ-canh trong tủ kính (`img_033.jpg`), xe đồ chơi mô hình (`img_038.jpg`), hình người/xe in trên áp phích/biển quảng cáo (`img_018.jpg`), và hình ảnh phản chiếu qua gương chiếu hậu hoặc mặt kính (`img_020.jpg`)[cite: 3].
6. **Phương tiện không động cơ:**
   - Xe đạp, xích lô đạp (`img_006.jpg`, `img_029.jpg`, `img_039.jpg`) **không** thuộc taxonomy phương tiện cơ giới → bỏ qua phương tiện, chỉ gán lớp `person` cho người điều khiển/người ngồi trên xe[cite: 3].

---

## 4. Bảng tổng hợp các ca khó (từ khảo sát 40 ảnh)

| STT | Mã ảnh | Tình huống thực tế | Phân loại xử lý | Quyết định đề xuất & Lý do |
|---|---|---|---|---|
| 1 | `img_002.jpg` | Xe tải chuyên dụng 2 tầng chở 8 ô tô con phía sau[cite: 3]. | Quy tắc rõ | Gán 1 box `truck` cho xe chở và 8 box `car` cho từng ô tô con trên sàn[cite: 3]. Vì các xe con là vật thể thật, rõ ràng và mô hình cần nhận diện được cả khi đang vận chuyển. |
| 2 | `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Xe ba bánh tuk-tuk chở khách/hàng[cite: 3]. | Đề xuất thêm lớp | Đề xuất bổ sung lớp `tuk_tuk` vào taxonomy thay vì ép vào `motorcycle` hay `car`, vì đặc trưng hình học 3 bánh và khoang chở khác biệt hoàn toàn[cite: 3]. |
| 3 | `img_009.jpg`, `img_031.jpg`, `img_040.jpg` | Xe buýt đô thị và xe khách cỡ lớn[cite: 3]. | Đề xuất thêm lớp | Đề xuất bổ sung lớp `bus` vì v0 chỉ có `car`, `truck`, `van`; kích thước và cấu trúc xe buýt khác xa `van` cỡ nhỏ[cite: 3, 10]. |
| 4 | `img_011.jpg`, `img_017.jpg`, `img_027.jpg` | Tượng đồng, tượng đá hình người và tượng xe bán tải[cite: 3]. | Loại theo quy tắc | Không gán nhãn[cite: 3]. Đây là vật thể tĩnh trang trí, không phải thực thể tham gia giao thông; gán nhãn sẽ gây dương tính giả (false positive) cho hệ thống. |
| 5 | `img_018.jpg`, `img_020.jpg` | Chân dung người trên biển quảng cáo (`img_018.jpg`) và hình phản chiếu trong gương (`img_020.jpg`)[cite: 3]. | Loại theo quy tắc | Không gán nhãn vì là hình ảnh 2D phẳng hoặc ảnh ảo phản xạ, không có thực thể vật lý tại vị trí box[cite: 3]. |
| 6 | `img_033.jpg`, `img_038.jpg` | Ma-nơ-canh trong tủ kính (`img_033.jpg`) và xe mô hình đồ chơi (`img_038.jpg`)[cite: 3]. | Loại theo quy tắc | Không gán nhãn (giữ ảnh ở trạng thái 0 annotation trong COCO)[cite: 3]. |
| 7 | `img_035.jpg` | Xe tuk-tuk chất hàng hóa/bao tải cao vượt quá nóc xe[cite: 3]. | Cần hỏi reviewer | Đề xuất box `tuk_tuk` chỉ ôm khung thân xe và bánh xe, **không** kéo box bao phần hàng hóa cồng kềnh chất trên nóc để tránh làm sai lệch tỷ lệ khung hình chuẩn của lớp `tuk_tuk`[cite: 3]. |
| 8 | `img_037.jpg` | Xe bán tải gắn nắp thùng cao kín (`Master 75`) và xe diễu hành[cite: 3]. | Cần hỏi reviewer | Gán `truck` (vì khung gầm gốc là xe bán tải pickup) và tick `can_review = true` để cả nhóm chốt ranh giới `truck` vs `van` trong buổi họp Pha 2[cite: 3]. |
| 9 | `img_006.jpg`, `img_029.jpg`, `img_039.jpg` | Xe đạp và xích lô di chuyển trên phố[cite: 3]. | Loại phương tiện | Chỉ gán `person` cho người đạp/người ngồi, không gán box cho xe đạp/xích lô do nằm ngoài phạm vi phương tiện cơ giới[cite: 3]. |

---

## 5. Quy trình QC và quản lý phiên bản

- **Phiên bản cá nhân (`v0-individual-<mã>`):** Dùng để đối chiếu trong buổi họp Pha 2, không dùng trực tiếp để gán nhãn chính thức.
- **Khóa đặc tả Pha 2 (`v1.0`):** Sau khi thống nhất các lớp mới (`bus`, `tuk_tuk`) và quy tắc ca biên, cập nhật `guideline_v1.0_agreed.md` đồng bộ với `submission/taxonomy.json` trước khi gán nhãn độc lập[cite: 2, 3].
- **Xử lý ca phát sinh khi gán:** Mọi trường hợp lưỡng lự trong Pha 3 đều phải đánh dấu `can_review = true` (nếu đã vẽ box) hoặc ghi vào `submission/qc/image_issues.csv` (nếu chưa vẽ được box), tuyệt đối không tự ý thay đổi quy tắc khi chưa họp calibration ở Pha 5[cite: 2, 3].