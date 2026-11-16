# Guideline gán nhãn — v1.0 — 101

- Phiên bản: v1.0, bản đề xuất cá nhân dựa trên guideline v0
- Ngày: 16/11/2026
- Người viết / duyệt: thành viên 101 

## 1. Mục tiêu

Bộ nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện trong ảnh đường phố 2D. Mỗi đối tượng trong phạm vi có một bounding box chữ nhật trong CVAT, xuất theo COCO 1.0. Bỏ sót người bị che, gộp người với xe hoặc gán tượng thành đối tượng thật có thể khiến mô hình học sai.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con có thân xe chủ yếu phục vụ chở người. | Sedan, hatchback, SUV. | Pickup có thùng hàng riêng thuộc `truck`. |
| `truck` | Xe tải hoặc pickup có khoang hàng tách rõ với cabin. | Xe chở ô tô, xe tải thùng, pickup. | Van có thân và khoang hàng liền khối. |
| `van` | Xe cỡ nhỏ có khoang kín nằm trong thân liền khối, với thân sau dạng hộp. | Van chở hàng hoặc chở người thân liền. | Xe tải có cabin và thùng kín tách rời, xe buýt cỡ lớn. |
| `motorcycle` | Xe cơ giới hai bánh kiểu xe máy hoặc mô tô. | Xe số, scooter, mô tô. | Xe đạp, xích lô và tuk-tuk ba bánh. |
| `person` | Người thật có phần cơ thể nhìn trực tiếp trong ảnh, kể cả người đang ngồi trên xe. | Người đi bộ, người lái xe, người ngồi sau. | Tượng, ma-nơ-canh và hình người trên biển quảng cáo. |
| `ignore_region` | Vùng giới hạn mà các đối tượng trong phạm vi quá nhỏ hoặc quá dày để gán đáng tin cậy từng box. | Cụm xe máy ở xa không tách được từng xe. | Vùng chứa đối tượng đủ rõ để vẽ box riêng. |

**R-01. Phạm vi.** Bỏ qua vật nhận diện rõ nhưng ngoài taxonomy, ví dụ xe đạp, tuk-tuk ba bánh, xe buýt cỡ lớn và động vật. Vẫn gán người thật trên các phương tiện đó nếu đủ điều kiện. Nếu chưa xác định được vật có thuộc taxonomy hay không và chưa có box đáng tin cậy, ghi vấn đề cấp ảnh để reviewer xem.

### Thuộc tính

| Thuộc tính | Mức cố định | Cách xác định |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng phần diện tích hình chiếu của vật bị vật khác che trong phạm vi ảnh. `0` = 0%, `1` = trên 0% đến dưới 25%, `2` = từ 25% đến dưới 50%, `3` = từ 50% trở lên nếu vẫn nhận ra vật và vẽ được box. Phần nằm ngoài ảnh chỉ tính vào `truncated`. |
| `truncated` | `no`, `yes` | `yes` khi mép ảnh cắt mất một phần đối tượng, `no` khi đối tượng nằm trọn trong khung ảnh. Box sát mép vẫn có thể là `no` nếu vật chưa bị cắt. |
| `can_review` | Bỏ tick, tick | Tick khi đã có box nhưng còn phân vân về lớp, ranh giới đối tượng, mức che hoặc mép cắt. Nếu không có box đủ tin cậy, ghi mã ảnh, vùng và lý do vào ghi chú vấn đề cấp ảnh. |

## 3. Quy tắc vẽ box

- **R-02. Phần nhìn thấy.** Vẽ hộp chữ nhật nhỏ nhất bao hết các phần nhìn thấy của cùng một vật. Khi vật bị che và phần nhìn thấy bị tách rời, bao các phần đó bằng một box nếu nhận ra chúng thuộc cùng một vật. Giữ box bên trong ảnh. Vật bị che hoàn toàn không có box.
- **R-03. Người và xe.** Vẽ một box cho xe và một box cho mỗi người đủ điều kiện. Box người bao cơ thể, quần áo và mũ đang đội. Box xe bao các phần cấu thành nhìn thấy như bánh, gương, tay lái và thân xe. Các box có thể chồng nhau.
- **R-04. Xe chở hàng hoặc chở xe.** Vẽ một box cho xe chở, gồm cabin và phần thùng hoặc giàn chở gắn với xe. Mỗi xe thật đang được chở có box riêng nếu nhận diện được và phần nhìn thấy cao ít nhất 20 px. Hàng rời nằm ngoài biên xe không làm box xe rộng thêm.
- **R-05. Vật giả và phản chiếu.** Bỏ qua tượng, ma-nơ-canh, xe mô hình, hình in, bóng đổ, phản chiếu trong gương hoặc mặt nước, và vùng lóe sáng. Nếu vật thật xuất hiện trực tiếp, gán phần vật thật nhìn thấy.
- **R-06. Xe ghi hình.** Bỏ qua bộ phận của xe đang mang camera, chẳng hạn mép nắp ca-pô hay gương chiếu hậu. Với xe khác nhìn trực tiếp trong ảnh, áp dụng quy tắc thông thường.
- **R-07. Vật nhỏ và đám đông.** Đo chiều cao phần nhìn thấy theo pixel ảnh gốc. Từ 20 px trở lên và tách được từng vật thì gán box riêng. Dùng `ignore_region` ôm sát vùng chứa vật dưới 20 px hoặc cụm không tách được từng đối tượng. Khoanh vùng sao cho vẫn giữ nhãn riêng cho các vật đủ rõ bên cạnh.
- **R-08. Xe cải trang.** Khi còn thấy dấu hiệu cấu tạo của xe thật như bánh, thân xe hoặc cabin, gán lớp theo xe nền. Phụ kiện trang trí rời và nhân vật giả trên xe xử lý theo R-05. Với vật trưng bày có hình xe nhưng không xác định được xe nền, ghi vấn đề cấp ảnh.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định đề xuất | Lý do |
|---|---|---|---|---|
| `img_005`, `img_026` | Người ngồi trên xe máy | R-02, R-03 | Gán từng `person` và từng `motorcycle` đủ điều kiện bằng box riêng. | Người và xe là các đối tượng độc lập, dù chồng lên nhau. |
| `img_002` | Xe tải chở nhiều ô tô | R-04 | Gán xe chở là `truck`. Gán từng ô tô nhìn đủ rõ trên giàn theo lớp riêng. | Giàn thuộc xe chở. Mỗi ô tô trên giàn là một xe thật. |
| `img_004` | Tuk-tuk ba bánh | R-01 | Bỏ qua thân tuk-tuk. Người thật đủ điều kiện vẫn có box `person`. | Taxi ba bánh nằm ngoài các lớp phương tiện đã định nghĩa. |
| `img_017`, `img_033` | Tượng cưỡi ngựa và ma-nơ-canh | R-05 | Bỏ qua tượng và ma-nơ-canh. | Đây là vật tạo hình người. |
| `img_019` | Xe vàng trang trí trong đoàn diễu hành | R-05, R-08 | Gán `car` nếu xác nhận được xe nền. Kiểm tra riêng nhân vật trên xe trước khi gán `person`. | Trang trí có thể che một phần thân xe hoặc có hình người, động vật. |
| `img_020` | Hình phản chiếu trong gương | R-05, R-06 | Bỏ qua vùng phản chiếu và gương xe ghi hình. | Gương không phải đối tượng cần phát hiện. |
| `img_021`, `img_030` | Nhiều xe máy và người chen sát nhau | R-02, R-07 | Gán riêng các vật tách được. Khoanh gọn vùng xa hoặc cụm không tách được. | Chất lượng quan sát thay đổi theo từng vùng ảnh. |
| `img_027`, `img_028` | Pickup trên bệ và pickup chở hình ngựa | R-04, R-05, R-08 | Kiểm tra ảnh gốc của `img_027` để phân biệt xe thật trưng bày với tượng xe. Ở `img_028`, gán pickup là `truck`, bỏ qua hình ngựa. | Quyết định phụ thuộc vào việc có xác định được xe thật hay không. |
| `img_038` | Xe mô hình trong tủ | R-05 | Bỏ qua các xe mô hình trong tủ. | Đây là đồ trưng bày dạng thu nhỏ. |

## 5. QC và phiên bản

- **Kiểm nhãn.** Kiểm lớp, số box, vị trí và kích thước box, thuộc tính, đối tượng bị bỏ sót và box thừa. Ngưỡng IoU vị trí là ≥ 0,85 với box cao từ 50 px và ≥ 0,70 với box thấp hơn 50 px.
- **Đồng thuận (IAA).** Kiểm đủ 100% lượt gán theo phân công. Ghép box 1–1 với IoU ≥ 0,50. Trên các cặp đã ghép, IoU trung bình cần ≥ 0,85 và đồng thuận lớp ≥ 95%. Báo số cặp dùng để tính và xem riêng các box chưa ghép.
- **Quyết định bàn giao.** Reviewer duyệt toàn bộ lô. Xét Hold khi thiếu bằng chứng hoặc còn vấn đề về guideline hay dữ liệu. Xét Rework khi hơn 2% ảnh có lỗi hoặc có lỗi lặp. Release khi số liệu đạt và reviewer xác nhận. Báo tỷ lệ lỗi theo ảnh và theo đối tượng, kèm tử số và mẫu số.
- **Ca mơ hồ.** Nhóm thống nhất ở buổi calibration. Người chốt ghi mã ảnh, Rule ID, quyết định và người kiểm lại. Nếu quy tắc đổi cách gán, cập nhật phiên bản và kiểm lại các ảnh chịu ảnh hưởng.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0.1 | Bổ sung tiêu chí lớp, ngưỡng che khuất, cách gán người và xe, cùng các ví dụ ca khó. | Giảm cách hiểu khác nhau ở `img_002`, `img_005`, `img_021` và `img_027`. |
