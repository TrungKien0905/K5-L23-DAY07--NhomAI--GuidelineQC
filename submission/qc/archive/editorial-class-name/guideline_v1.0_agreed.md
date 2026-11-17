# Guideline gán nhãn — v1.0 — Nhóm 101, 102, 103

- Phiên bản: v1.0
- Ngày: 16/11/2026
- Thành viên: 101, 102, 103
- Người duyệt: 101

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
| `ignore_region` | Vùng bỏ qua khi một cụm quá nhỏ hoặc quá dày để gán từng vật, hoặc khi quy tắc đám đông R-07 áp dụng. | Cụm người chen sát có hơn 5 người ở `img_021`. | Người hoặc xe đứng tách khỏi cụm, đủ rõ và đạt kích thước tối thiểu. |
| `bus` | Xe buýt hoặc xe khách cỡ lớn, nhận diện qua thân dài, khoang hành khách lớn và cửa lên xuống. | Xe buýt đô thị, xe khách liên tỉnh. | Xe van thân liền dạng hộp cỡ nhỏ hoặc trung bình. |
| `tuk_tuk` | Xe cơ giới ba bánh kiểu tuk-tuk hoặc xe lam, có khoang hoặc sàn chở khách/hàng riêng. | Tuk-tuk ở `img_004` và `img_035`. | Xe máy hai bánh, xích lô đạp. |

**R-01. Phạm vi.** Mỗi vật thật trong ảnh thuộc tối đa một lớp đối tượng. Bỏ qua vật ngoài taxonomy như xe đạp, xích lô đạp và động vật. Người thật ngồi trên hoặc điều khiển phương tiện ngoài taxonomy vẫn được gán `person` nếu đủ điều kiện. Phân biệt các lớp xe theo cấu tạo quan sát được trong ảnh. Nếu còn phân vân nhưng đã vẽ được box, tick `can_review`. Nếu chưa có box đáng tin cậy, ghi vấn đề cấp ảnh.

### Thuộc tính

| Thuộc tính | Mức cố định | Cách xác định |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng tỷ lệ diện tích hình chiếu trong khung ảnh bị vật khác che. `0` = 0%, `1` = trên 0% đến 30%, `2` = trên 30% đến 70%, `3` = trên 70% nếu vẫn nhận diện được lớp và vẽ được box. Phần nằm ngoài ảnh chỉ tính vào `truncated`. |
| `truncated` | `no`, `yes` | `yes` khi mép ảnh cắt mất một phần đối tượng, `no` khi đối tượng nằm trọn trong khung ảnh. Box sát mép vẫn có thể là `no` nếu vật chưa bị cắt. |
| `can_review` | Bỏ tick, tick | Tick khi đã có box nhưng còn phân vân về lớp, ranh giới đối tượng, mức che hoặc mép cắt. Nếu không có box đủ tin cậy, ghi mã ảnh, vùng và lý do vào ghi chú vấn đề cấp ảnh. |

## 3. Quy tắc vẽ box

- **R-02. Phần nhìn thấy.** Vẽ hộp chữ nhật nhỏ nhất bao hết các phần nhìn thấy của cùng một vật. Khi vật bị che và phần nhìn thấy bị tách rời, bao các phần đó bằng một box nếu nhận ra chúng thuộc cùng một vật. Giữ box bên trong ảnh. Vật bị che hoàn toàn không có box. Nếu vật bị che quá nhiều vẫn ưu tiên đánh box nếu xác định được vật.
- **R-03. Người và xe.** Vẽ một box cho xe và một box cho mỗi người đủ điều kiện. Box người bao cơ thể, quần áo và mũ đang đội. Box xe bao các phần cấu thành nhìn thấy như bánh, gương, tay lái và thân xe. Các box có thể chồng nhau.
- **R-04. Xe chở hàng hoặc chở xe.** Vẽ một box cho xe chở, gồm cabin và phần thùng hoặc giàn chở gắn với xe. Mỗi xe thật đang được chở có box riêng nếu nhận diện được và phần nhìn thấy cao ít nhất 20 px. Hàng rời nằm ngoài biên xe không làm box xe rộng thêm.
- **R-05. Vật giả và phản chiếu.** Bỏ qua tượng, ma-nơ-canh, xe mô hình, hình in, bóng đổ, phản chiếu trong gương hoặc mặt nước, và vùng lóe sáng. Nếu vật thật xuất hiện trực tiếp, gán phần vật thật nhìn thấy.
- **R-06. Xe ghi hình.** Bỏ qua bộ phận của xe đang mang camera, chẳng hạn mép nắp ca-pô hay gương chiếu hậu. Với xe khác nhìn trực tiếp trong ảnh, áp dụng quy tắc thông thường.
- **R-07. Vật nhỏ và đám đông.** Đo chiều cao phần nhìn thấy theo pixel ảnh gốc. Bỏ qua vật đơn lẻ cao dưới 20 px. Với cụm nhiều vật nhỏ hoặc cụm quá dày không tách được từng vật, vẽ một box `ignore_region` ôm sát cụm. Với đám đông chen sát có **hơn 5 người** (từ 6 người), dùng một `ignore_region` cho cả cụm và không gán box người riêng bên trong vùng đó. Người và xe đủ rõ, nằm ngoài vùng, vẫn được gán riêng.
- **R-08. Xe cải trang.** Khi nhận ra cấu tạo của xe thật, gán lớp theo xe nền. Áp dụng R-05 cho phụ kiện rời, hình in và nhân vật giả. Khi chưa xác định được đó là xe thật hay mô hình, ghi vấn đề cấp ảnh để reviewer xem.
- **R-09. Vật bị nhòe.** Bỏ qua đối tượng mờ đến mức không xác định được lớp hoặc biên box. Với đối tượng vẫn xác định được, vẽ box ôm phần thân nhìn thấy và loại phần vệt nhòe kéo dài. Tick `can_review` nếu còn phân vân về biên.
- **R-10. Xe đặt trên nóc xe trưng bày.** Ở `img_024`, bỏ qua chiếc xe màu vàng trên nóc, không vẽ box riêng và không kéo box xe bên dưới lên để bao nó. Quy tắc R-04 áp dụng cho xe tải chuyên dụng chở xe như ở `img_002`.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_005`, `img_026` | Người ngồi trên xe máy | R-02, R-03 | Gán từng `person` và từng `motorcycle` đủ điều kiện bằng box riêng. | Người và xe là các đối tượng độc lập, dù chồng lên nhau. |
| `img_002` | Xe tải chở nhiều ô tô | R-04 | Gán xe chở là `truck`. Gán từng ô tô nhìn đủ rõ trên giàn theo lớp riêng. | Giàn thuộc xe chở. Mỗi ô tô trên giàn là một xe thật. |
| `img_004`, `img_016`, `img_035` | Tuk-tuk ba bánh | R-01, R-04 | Gán `tuk_tuk` theo phần thân xe nhìn thấy, bỏ phần hàng rời nhô ra. Gán riêng người đủ điều kiện. | Hình dáng xe và khoang chở phân biệt được với xe máy hai bánh. |
| `img_009` | Xe khách trên phố | R-01 | Dựa vào thân, khoang khách và cửa để phân biệt `bus` với `van`. Tick `can_review` nếu góc chụp chưa đủ rõ. | Số chỗ thường khó xác định trong ảnh. |
| `img_014` | Vật thể bị nhòe khi di chuyển | R-09 | Bỏ qua các vật thể bị mờ, không vẽ box cho các vệt nhòe. Hai người đi bộ thấy rõ vẫn gán `person` riêng. | Có thể xác định biên hai người đi bộ nhưng không thể vẽ box đáng tin cậy cho các vật bị nhòe. |
| `img_017`, `img_033` | Tượng cưỡi ngựa và ma-nơ-canh | R-05 | Bỏ qua tượng và ma-nơ-canh. | Đây là vật tạo hình người. |
| `img_019` | Xe vàng trang trí trong đoàn diễu hành | R-05, R-08 | Gán `car` nếu xác nhận được xe nền. Kiểm tra riêng nhân vật trên xe trước khi gán `person`. | Trang trí có thể che một phần thân xe hoặc có hình người, động vật. |
| `img_020` | Hình phản chiếu trong gương | R-05, R-06 | Bỏ qua vùng phản chiếu và gương xe ghi hình. | Gương không phải đối tượng cần phát hiện. |
| `img_021` | Đám đông chen sát có hơn 5 người | R-07 | Vẽ một `ignore_region` ôm sát cụm đông ở phía xa. Gán riêng người và xe đủ rõ nằm ngoài vùng. | Vùng bỏ qua chỉ bao phần đám đông cần khoanh. |
| `img_024` | Xe màu vàng đặt trên nóc xe đang trưng bày | R-10 | Không gán xe ở trên. Gán xe phía dưới và các xe khác theo quy tắc thông thường nếu đủ điều kiện. | Cách xử lý đã chốt cho bố cục trưng bày này. |
| `img_027`, `img_028` | Pickup trên bệ và pickup chở hình ngựa | R-04, R-05, R-08 | Bỏ qua tượng pickup trên bệ ở `img_027`. Gán pickup thật ở `img_028` là `truck` và bỏ qua hình ngựa. | Hai vật có hình dáng pickup nhưng chỉ một chiếc là xe thật theo cách phân loại của ca này. |
| `img_030` | Xe máy đỗ dày đặc | R-02, R-07 | Gán riêng xe đủ rõ, dùng `ignore_region` cho cụm không tách được. | Có thể nhận diện riêng một số xe ở tiền cảnh. |
| `img_037` | Pickup có nắp thùng cao | R-01 | Gán `truck` nếu thấy kết cấu cabin và thùng tách rời. Tick `can_review` nếu chưa phân biệt được với `van`. | Xếp lớp theo kết cấu xe. |
| `img_038` | Xe mô hình trong tủ | R-05 | Bỏ qua các xe mô hình trong tủ. | Đây là đồ trưng bày dạng thu nhỏ. |
| `img_015` | Xe bị nhòe một phần | R-09 | Gán phần thân xe còn xác định được, bỏ phần vệt nhòe kéo dài. | Phần thân còn rõ đủ để đặt box. |

## 5. QC và phiên bản

- **Kiểm nhãn.** Kiểm lớp, số box, vị trí và kích thước box, thuộc tính, đối tượng bị bỏ sót và box thừa. Ngưỡng IoU vị trí là ≥ 0,85 với box cao từ 50 px và ≥ 0,70 với box thấp hơn 50 px.
- **Đồng thuận (IAA).** Kiểm đủ 100% lượt gán theo phân công. Ghép box 1–1 với IoU ≥ 0,50. Trên các cặp đã ghép, IoU trung bình cần ≥ 0,85 và đồng thuận lớp ≥ 95%. Báo số cặp dùng để tính và xem riêng các box chưa ghép.
- **Quyết định bàn giao.** Reviewer duyệt toàn bộ lô. Xét Hold khi thiếu bằng chứng hoặc còn vấn đề về guideline hay dữ liệu. Xét Rework khi hơn 2% ảnh có lỗi hoặc có lỗi lặp. Release khi số liệu đạt và reviewer xác nhận. Báo tỷ lệ lỗi theo ảnh và theo đối tượng, kèm tử số và mẫu số.
- **Ca mơ hồ.** Nhóm thống nhất ở buổi calibration. Người chốt ghi mã ảnh, Rule ID, quyết định và người kiểm lại. Nếu quy tắc đổi cách gán, cập nhật phiên bản và kiểm lại các ảnh chịu ảnh hưởng.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Tổng hợp guideline 101, 102, 103. Thống nhất lớp, thuộc tính, cách vẽ box và quy tắc QC. | Bổ sung quyết định cho `img_014`, `img_021` và `img_024`. |
