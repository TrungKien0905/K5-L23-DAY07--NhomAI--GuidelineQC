# Guideline gán nhãn — v0.1 — 103

- Phiên bản: `v0.1 - 103` (Đề xuất cá nhân nâng cấp từ bản khởi đầu v0)
- Ngày: 2026-11-16
- Người viết / duyệt: Thành viên `103`

---

## 1. Mục tiêu

### 1.1. Phạm vi ứng dụng của mô hình
Tập dữ liệu gán nhãn hộp chữ nhật 2D (bounding box) này được thiết kế nhằm phục vụ huấn luyện và đánh giá các mô hình thị giác máy tính phát hiện đối tượng (Object Detection) thời gian thực phục vụ:
1. **Hệ thống xe tự lái (Autonomous Vehicles - AV) / ADAS:** Nhận diện người đi bộ, người điều khiển phương tiện và các loại phương tiện giao thông trên hành trình di chuyển để lập kế hoạch di chuyển (path planning), căn khoảng cách an toàn và kích hoạt hệ thống phanh tự động khẩn cấp (AEB).
2. **Camera giám sát giao thông thông minh (ITS):** Đếm lưu lượng phương tiện theo làn, phân loại phương tiện, phát hiện ùn tắc, nhận diện vi phạm dừng đỗ và phân tích hành vi của người tham gia giao thông.

### 1.2. Phân tích rủi ro an toàn và kỹ thuật khi gán sai nhãn
Dữ liệu gán nhãn là "chân lý mặt đất" (Ground Truth) để mô hình học. Bất kỳ sự thiếu nhất quán hay sai sót nào trong nhãn đều gây hậu quả trực tiếp đến an toàn tính mạng và hiệu năng hệ thống:
- **Bỏ sót đối tượng (False Negative):** Nguy hiểm nhất là bỏ sót `person` hoặc `motorcycle`. Nếu mô hình bỏ qua một người đi bộ băng qua đường hoặc người đi xe máy khuất tầm nhìn, xe tự hành sẽ không kịp phanh, dẫn đến tai nạn đâm va trực diện gây tử vong.
- **Nhận diện sai vật thể giả thành vật thật (False Positive):** Gán box cho người in trên biển quảng cáo, ma-nơ-canh thời trang, bóng phản chiếu trong gương kính hoặc xe đồ chơi sẽ khiến hệ thống tự động phanh gấp ngoài ý muốn (phantom braking), tiềm ẩn nguy cơ cao bị xe phía sau đâm dồn toa (rear-end collision) trên đường cao tốc.
- **Gán nhầm lớp (Misclassification):** Nhầm giữa `person` và `motorcycle` hoặc giữa `car` và `truck` sẽ làm sai lệch mô hình dự đoán quỹ đạo động học (Kinematic / Trajectory Prediction). Xe máy và người đi bộ có khả năng chuyển hướng đột ngột và gia tốc khác hẳn xe tải hay xe con.
- **Box lỏng hoặc lệch kích thước (Loose / Inaccurate Boundary):** Ước lượng khoảng cách 3D từ bounding box 2D sẽ bị sai lệch nghiêm trọng, làm hệ thống đánh giá sai khoảng cách va chạm (Time-to-Collision - TTC).

---

## 2. Lớp và thuộc tính

### 2.1. Bảng định nghĩa chi tiết các lớp đối tượng

| Lớp | Định nghĩa (chuẩn mực) | Ví dụ bao gồm | Phản ví dụ / Dễ nhầm lẫn |
|---|---|---|---|
| `car` | Ô tô con từ 4 đến 9 chỗ ngồi phục vụ chở người cá nhân hoặc gia đình, thân xe nguyên khối hoặc gầm cao. | Sedan, hatchback, SUV, crossover, coupe, xe mui trần, taxi con 4–7 chỗ. | Xe van chở hàng (`van`), xe tải (`truck`), xe bán tải thùng hở (`truck`), xe khách trên 9 chỗ (`bus`). |
| `truck` | Phương tiện cơ giới chuyên dụng chở hàng hóa, có kết cấu cabin và thùng chở hàng/sàn phẳng tách rời hoặc bán tải. | Xe tải thùng kín, xe tải bui bạt, xe ben, xe container, xe bồn, xe cứu hộ chở ô tô (car carrier), xe bán tải (pickup truck). | Xe van thân liền khối (`van`), ô tô con SUV (`car`). |
| `van` | Ô tô chở hàng hoặc chở người cỡ nhỏ/trung có thân xe dạng hộp liền khối (monocoque), không tách rời cabin và thùng. | Ford Transit, Hyundai Solati, Toyota HiAce, minivan chở hàng/khách nhỏ. | Xe tải thùng rời (`truck`), SUV 7 chỗ (`car`), xe buýt cỡ lớn (`bus`). |
| `motorcycle` | Phương tiện giao thông cơ giới hai bánh hoặc ba bánh có gắn động cơ xăng hoặc động cơ điện. | Xe máy số, xe tay ga, xe côn tay, mô tô phân khối lớn, xe máy điện có biển số. | Xe đạp cơ (loại trừ), xe đạp điện dùng bàn đạp (loại trừ), xe ba bánh tuk-tuk (đề xuất lớp riêng `tuk_tuk`). |
| `person` | Cá thể con người thực tế còn sống trong mọi tư thế (đứng, đi, ngồi, ngồi trên xe, mang vác đồ). | Người đi bộ, cảnh sát, công nhân, người đang điều khiển hoặc ngồi sau xe máy/ô tô/xe đạp. | Ma-nơ-canh cửa hàng, tượng đài, hình in quảng cáo 2D, hình người phản chiếu qua gương. |
| `ignore_region` | Vùng không gian chứa các đối tượng mờ, nhiễu, quá dày đặc hoặc quá nhỏ không gán đơn lẻ được; không tính lỗi sót ở đây. | Đám đông đi bộ chen chúc không tách nổi viền, bãi đỗ xe xa tít có xe cao < 20 px, vùng ảnh lóa/nhiễu nặng. | Người hoặc xe đứng đơn lẻ, nhìn rõ hình dạng và có chiều cao ≥ 20 px. |
| *`bus`* *(Đề xuất)* | Phương tiện giao thông công cộng cỡ lớn chở trên 9 hành khách (chiều dài và thể tích vượt trội xe van). | Xe buýt nội đô, xe khách liên tỉnh, xe đưa đón học sinh cỡ lớn. | Xe van 16 chỗ (`van`), xe tải thùng kín (`truck`). |
| *`tuk_tuk`* *(Đề xuất)* | Phương tiện 3 bánh cơ giới chở khách đặc thù có mui/khung xe che chở, phổ biến tại Nam/Đông Nam Á. | Xe lam, xe lôi máy, xe tuk-tuk chở khách du lịch. | Xe máy thông thường 2 bánh (`motorcycle`), ô tô con mini (`car`). |

> **Quy định tạm thời trước khi v1.0 chốt lớp bổ sung:**
> - Nếu gặp xe buýt lớn: Tạm thời gán vào `truck` (hoặc `van` theo thảo luận nhóm), tick `can_review`.
> - Nếu gặp xe tuk-tuk: Tạm thời gán vào `motorcycle` và tick `can_review`.

### 2.2. Xử lý các đối tượng không thuộc lớp nào (Out of Taxonomy)
- **Xe đạp (Bicycle), xích lô, xe súc vật kéo, xe đẩy tay:** Không thuộc danh mục đào tạo của mô hình. Tuyệt đối **KHÔNG** gán box cho phần phương tiện này.
- **Người ngồi/điều khiển xe đạp hoặc xích lô:** Vẫn gán 1 box `person` ôm trọn cơ thể người (từ đầu đến chân người nhìn thấy), bỏ qua phần khung xe đạp.
- **Động vật (chó, mèo, gia súc), chướng ngại vật mặt đường (nón chóp, rào chắn):** Bỏ qua, không gán.

### 2.3. Thuộc tính đối tượng (Attributes) và ngưỡng định lượng

Mọi bounding box gán cho các lớp thực (trừ `ignore_region`) đều phải có đủ 3 thuộc tính sau:

1. **`occluded_level` (Mức độ che khuất):** Đo lường tỷ lệ phần diện tích cơ thể/thân xe bị che khuất bởi vật cản khác (cột đèn, cây cối, xe khác, người khác).
   - `0` (Không bị che): Bị che đúng **0%**. Toàn bộ đường bao hình học của vật thể hiển thị trọn vẹn, không bị bất kỳ vật thể nào khác che chắn.
   - `1` (Che khuất nhẹ): Bị che từ **1% đến 30%** diện tích bề mặt nhìn thấy. Phần lớn đặc trưng nhận diện vẫn rõ ràng.
   - `2` (Che khuất trung bình): Bị che từ **31% đến 70%** diện tích bề mặt nhìn thấy. Vẫn nhận diện được hình dạng xe/người dựa trên các bộ phận lộ ra.
   - `3` (Che khuất nặng): Bị che **trên 70%** diện tích, chỉ còn thấy một phần nhỏ (ví dụ chỉ thấy đầu xe, một góc mũ bảo hiểm hoặc nửa thân trên), nhưng mắt người vẫn nhận diện chắc chắn đây là đối tượng đó.
   *(Lưu ý: Nếu vật thể bị che khuất trên 90% đến mức mắt người không thể nhận diện chắc chắn thuộc lớp nào thì KHÔNG gán).*

2. **`truncated` (Cắt mép ảnh):**
   - `yes`: Bounding box của đối tượng chạm hoặc cắt vào ít nhất một trong 4 mép viền của ảnh (trái, phải, trên, dưới) do đối tượng nằm một phần ngoài góc nhìn của camera.
   - `no`: Toàn bộ đối tượng nằm hoàn toàn bên trong khung ảnh, không có phần cơ thể/thân xe nào bị mép ảnh cắt qua.

3. **`can_review` (Đánh dấu cần xem lại):**
   - `true` (tick checkbox): Đánh dấu khi annotator còn phân vân hoặc gặp tình huống biên (edge case) chưa có tiền lệ rõ ràng trong guideline (ví dụ: ranh giới giữa car và truck, xe ba bánh, góc nhìn quá mờ, người bị che mấp mé ngưỡng).
   - `false` (mặc định không tick): Đã chắc chắn tuân thủ đúng quy tắc guideline.

---

## 3. Quy tắc vẽ box

Mọi annotator phải tuân thủ nghiêm ngặt 8 quy tắc chuẩn hóa sau (mã hóa từ `R-01` đến `R-08` để thuận tiện dẫn chứng khi QC):

- **`R-01` (Vẽ ôm sát phần nhìn thấy - Visible Part):**
  Bounding box là hộp chữ nhật ngoại tiếp 2D nhỏ nhất ôm khít toàn bộ các điểm ảnh thực tế quan sát được (visible pixels) của đối tượng. Tuyệt đối **KHÔNG** vẽ ước lượng phần bị che khuất hay phần bị cắt ngoài ảnh (không vẽ amodal box). Bốn cạnh của box phải tiếp xúc chính xác với các điểm cực viễn của đối tượng.

- **`R-02` (Tách riêng người và xe máy - Separation Rule):**
  Đối với người đang điều khiển hoặc ngồi trên xe máy:
  - Vẽ **01 box `person`** ôm trọn từ đỉnh đầu (hoặc mũ bảo hiểm) đến phần cơ thể thấp nhất nhìn thấy được của người đó.
  - Vẽ **01 box `motorcycle`** ôm trọn từ đầu xe, tay lái, thân xe đến hết bánh xe.
  - Hai box này bắt buộc chồng lấn lên nhau (overlap) tự nhiên tại khu vực người ngồi trên yên xe.
  - Nếu xe máy chở 2 hoặc 3 người: Vẽ **01 box `motorcycle`** cho xe và **từng box `person` riêng biệt** cho mỗi người trên xe.

- **`R-03` (Xe chuyên dụng chở ô tô con - Car Carrier Rule):**
  Khi gặp xe tải chuyên dụng đang chở các ô tô con trên sàn hoặc giá đỡ:
  - Gán **01 box `truck`** bao trùm toàn bộ xe chuyên dụng (tính cả đầu kéo, rơ-moóc và sàn chở).
  - Đồng thời gán **từng box `car` riêng biệt** cho MỖI chiếc ô tô con được cố định trên sàn xe (nếu ô tô con đó nhìn thấy rõ và đạt chiều cao ≥ 20 px).
  - Lý do: Giúp mô hình vừa nhận biết phương tiện vận tải lớn, vừa học được đặc trưng ô tô con ở vị trí trên cao.

- **`R-04` (Loại trừ đối tượng giả / phi sinh học - Negative Cases):**
  Tuyệt đối **KHÔNG** gán box cho các đối tượng sau:
  - Tượng người (tượng đồng, tượng đá, tượng đài trang trí ven đường).
  - Ma-nơ-canh trưng bày trong hoặc ngoài cửa hàng quần áo.
  - Đồ chơi trẻ em (xe ô tô đồ chơi, mô hình tĩnh thu nhỏ).
  - Hình in 2D trên biển hiệu quảng cáo, áp phích, băng rôn hoặc đề can trang trí trên thân xe buýt/xe tải.

- **`R-05` (Loại trừ ảnh phản chiếu qua gương / kính - Reflection Rule):**
  Hình ảnh người hoặc xe phản chiếu qua gương chiếu hậu của xe, mặt kính tòa nhà, vách kính cửa hàng hoặc vũng nước đọng trên mặt đường đều là ảnh ảo 2D -> **KHÔNG** gán box.

- **`R-06` (Loại trừ xe chủ - Ego Vehicle Rule):**
  Phần mui xe, nắp ca-pô hoặc kính chắn gió của chính chiếc xe chở camera chụp ảnh (thường nằm sát mép đáy ảnh) thuộc về phương tiện chủ (ego vehicle) -> **KHÔNG** gán box.

- **`R-07` (Ngưỡng kích thước tối thiểu - Min Size Rule):**
  Đối tượng đơn lẻ có chiều cao (box height) **nhỏ hơn 20 pixels** được coi là quá nhỏ, không thể nhận dạng chắc chắn bằng mắt thường -> **KHÔNG** gán box đơn lẻ (nếu nằm trong cụm đông thì gom vào `ignore_region`).

- **`R-08` (Sử dụng vùng bỏ qua - `ignore_region` Rule):**
  Sử dụng một box với lớp `ignore_region` để bao quanh:
  - Đám đông người đi bộ quá dày đặc, đứng chen chúc nhau ở khoảng cách xa mà ranh giới giữa các cá thể không thể phân biệt rõ ràng.
  - Cụm nhiều phương tiện san sát nhau ở hậu cảnh có kích thước mỗi xe < 20 px.
  *(Bỏ sót đối tượng trong vùng `ignore_region` không bị tính là lỗi khi nghiệm thu QC).*

---

## 4. Ca khó và tình huống biên (Edge Cases)

Bảng dưới đây thiết lập quyết định xử lý các ca khó và tình huống biên điển hình được rà soát và đối chiếu trực tiếp từ tập 40 ảnh thực tế của bài lab:

| Ảnh | Tình huống biên / Ca khó | Rule ID | Quyết định đề xuất | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_004` | Phương tiện 3 bánh chở khách (Tuk-tuk / xe lôi / xe lam). | `R-02` | Tạm gán `motorcycle`, tick `can_review`. Đề xuất nhóm lập lớp `tuk_tuk`. | v0 chưa có lớp này. Gán car thì sai kích thước/động học, gán motorcycle thì sai hình thái thùng xe. |
| `img_002` | Xe cứu hộ hoặc xe tải chuyên dụng chở 2–3 ô tô con trên sàn thùng. | `R-03` | Vẽ 01 box `truck` cho xe tải, và từng box `car` cho mỗi ô tô con trên thùng xe. | Ô tô con trên thùng xe vẫn mang đầy đủ đặc trưng thị giác 2D của ô tô; xe tải mang đặc trưng phương tiện lớn. |
| `img_033` | Ma-nơ-canh trưng bày quần áo ở mặt tiền cửa hàng sát lề đường. | `R-04` | Tuyệt đối KHÔNG gán box `person`. | Ma-nơ-canh không có hành vi sinh học, không di chuyển; gán nhãn sẽ khiến xe tự hành phanh oan (false alarm). |
| `img_017` | Tượng người hoặc tượng danh nhân đứng ở dải phân cách / công viên ven đường. | `R-04` | Tuyệt đối KHÔNG gán box `person`. | Là vật thể tĩnh cố định; tránh gây nhiễu cho mô hình phân loại người tham gia giao thông. |
| `img_020` | Xe máy hoặc người quan sát thấy qua gương chiếu hậu của ô tô. | `R-05` | Tuyệt đối KHÔNG gán box. | Ảnh ảo trong gương không đại diện cho tọa độ vật lý trước mặt camera; gán vào sẽ làm sai lệch không gian 3D. |
| `img_037` | Xe bán tải (pickup truck) có gắn nắp thùng hàng cao ngang nóc ca-bin. | `R-01` | Gán `truck`, tick `can_review` nếu hình thái giống SUV. | Dựa vào chiều dài cơ sở và kết cấu khung gầm; xe bán tải bản chất là xe tải nhẹ chở hàng. |
| `img_033` | Tấm biển quảng cáo cỡ lớn có in hình người mẫu hoặc hình chiếc ô tô thật. | `R-04` | Tuyệt đối KHÔNG gán box `person` hay `car`. | Hình in 2D trên phẳng không phải đối tượng thực thể trong không gian giao thông. |
| `img_011` | Đám đông cổ động viên / người đi bộ chen chúc nhau ở xa, viền người chồng chéo. | `R-08` | Vẽ 01 box lớn bao quanh với lớp `ignore_region`. | Không thể tách biệt biên 1-1 một cách nhất quán; vẽ cố sẽ làm giảm IAA giữa các annotator. |
| `img_038` | Đồ chơi xe ô tô mô hình trưng bày qua tấm gương / tủ kính. | `R-04`, `R-05` | Tuyệt đối **KHÔNG** gán box. | Là đồ chơi mô hình thu nhỏ và phản chiếu qua gương/kính; không phải phương tiện thật. Gán box sẽ gây báo động giả (phantom braking). |
| `img_015` | Ô tô đang chạy nhanh bị mờ nhòe, để lại một vệt chuyển động (motion blur). | `R-01` | Gán box `car` ôm sát phần thân xe thực sự xác định được (không bao vệt nhòe kéo dài), tick `can_review`. Nếu mờ hoàn toàn thì vào `ignore_region`. | Tránh box quá lỏng làm giảm IoU và sai lệch kích thước vật lý; tick `can_review` để nhóm chốt thống nhất ở Pha 2. |
| `img_018` | Xe ô tô nằm sau một hàng rào (bị che khuất bởi các song sắt/lưới rào). | `R-01` | Vẫn gán box `car` ôm sát viền ngoài của xe nhìn thấy qua hàng rào; đặt `occluded_level = 2` (hoặc `1`/`3` tùy % che khuất). | Là phương tiện thật trong không gian; mô hình xe tự lái cần học đặc trưng xe bị che khuất để kịp thời phát hiện xe đỗ/lao ra từ cổng. |
| `img_019` | Xe đồ chơi và có người mặc trang phục cosplay thú (mascot). | `R-01`, `R-04` | - Xe đồ chơi: **KHÔNG** gán box.<br>- Người cosplay: Gán 01 box `person`, tick `can_review`. | Xe đồ chơi không phải phương tiện giao thông. Người mặc đồ mascot là con người thực tế có khả năng di chuyển; ưu tiên an toàn sinh mạng bắt buộc phải phát hiện. |

---

## 5. Quy định QC và lịch sử phiên bản

### 5.1. Quy định và tiêu chí nghiệm thu chất lượng (QC)
Mọi lượt gán nhãn phải được đối chiếu theo các ngưỡng kiểm soát chất lượng quy định tại `RULES.md` và `taxonomy.json`:
1. **Ngưỡng trùng khớp hình học (IoU Threshold):**
   - Đối tượng lớn (chiều cao ≥ 50 px): Ngưỡng yêu cầu $\text{IoU} \ge 0.85$.
   - Đối tượng nhỏ (chiều cao < 50 px): Ngưỡng yêu cầu $\text{IoU} \ge 0.70$.
2. **Độ đồng thuận liên gán nhãn (Inter-Annotator Agreement - IAA):**
   - Chỉ tính trên các cặp box ghép 1–1 có $\text{IoU} \ge 0.50$.
   - Yêu cầu $\text{IoU trung bình} \ge 0.85$.
   - Yêu cầu tỷ lệ trùng khớp lớp (Class Agreement) $\ge 95\%$.
3. **Tỷ lệ lỗi tối đa cho phép của lô ảnh:**
   - Tỷ lệ ảnh có lỗi $\le 2.0\%$ (trên tập 40 ảnh, nếu có từ 1 ảnh bị lỗi nghiêm trọng thì toàn bộ lô chuyển sang trạng thái **Rework** hoặc **Hold**).
4. **Quy trình giải quyết ca mơ hồ:**
   - Trong quá trình gán nhãn độc lập: annotator tick `can_review` trên các box chưa chắc chắn và ghi chú vào nhật ký.
   - Tại buổi họp Calibration (Pha 2 và Pha 5): Toàn nhóm thảo luận dựa trên bằng chứng hình ảnh để đưa ra quyết định thống nhất cuối cùng. Mọi thay đổi về ngữ nghĩa phân loại đều phải cập nhật phiên bản guideline.

### 5.2. Lịch sử thay đổi tài liệu (Changelog)

| Phiên bản | Nội dung thay đổi chính | Lý do thay đổi / Căn cứ ca khó |
|---|---|---|
| `v0` | Bản khởi đầu sơ sài do giảng viên cung cấp; chỉ có định nghĩa 1 dòng, chưa có ngưỡng định lượng, chưa có quy tắc ca biên. | Tài liệu gốc ban đầu trong repository. |
| `v0.1 - 103` | Nâng cấp toàn diện bản v0 thành đề xuất cá nhân chi tiết: <br>1. Làm rõ mục tiêu mô hình và phân tích rủi ro an toàn (Mục 1). <br>2. Bổ sung ví dụ, phản ví dụ cho 6 lớp cũ; đề xuất bổ sung 2 lớp `bus` và `tuk_tuk` (Mục 2). <br>3. Lượng hóa chính xác 4 mức `occluded_level` theo %, tiêu chuẩn `truncated` và `can_review` (Mục 2). <br>4. Ban hành 8 quy tắc chuẩn hóa `R-01` đến `R-08`: vẽ visible box, tách người và xe máy, xe chở xe, loại trừ tượng/ma-nơ-canh/hình in/gương/xe chủ, ngưỡng < 20 px và `ignore_region` (Mục 3). <br>5. Lập khung bảng 8 ca khó để chuẩn bị đối chiếu 40 ảnh (Mục 4). <br>6. Bổ sung bộ tiêu chuẩn định lượng QC theo rubric (Mục 5). | Khắc phục triệt để tính mơ hồ của v0; chuẩn bị cơ sở đàm phán thống nhất guideline v1.0 tại cuộc họp nhóm ở Pha 2. |
