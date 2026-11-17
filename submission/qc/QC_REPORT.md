# Báo cáo QC — Nhóm 101, 102, 103

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả, gán và QC bounding box cho người và phương tiện trong 40 ảnh đường phố 2D. Bộ ảnh được tuyển chọn để luyện ca biên. Độ đại diện cho dữ liệu vận hành chưa được đánh giá.
- Nguồn và phép sử dụng dữ liệu: [ATTRIBUTION.csv](../../data/ATTRIBUTION.csv) ghi tác giả, giấy phép và URL nguồn cho từng ảnh. Áp dụng điều kiện ghi công và chia sẻ tương tự theo giấy phép của từng ảnh.
- Manifest và checksum nhãn cuối: [batch_manifest.json](batch_manifest.json) ghi SHA-256 của [labels_calibrated.json](labels_calibrated.json) là `1d8cc2db085808da75b02ae994642028fc89e37fa3ed721ec6dbfd11169c5761`. Bản hiện tại có 40 ảnh và 411 annotation.
- Guideline và taxonomy khi gán / khi bàn giao: Gán theo [guideline v1.0](../guidelines/guideline_v1.0_agreed.md), chuẩn bị bàn giao theo bản nháp [v1.1](../guidelines/guideline_v1.1_calibrated.md). Thay đổi v1.1 thuộc loại editorial theo [guideline_change.json](../guideline_change.json). Taxonomy giữ v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt gán trên 40 ảnh, mỗi ảnh hai người.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: Kết quả trước khi đồng bộ nhãn là 296 cặp ghép, IoU TB 0.807, cùng lớp 93.9%, chưa đạt ngưỡng 0.85 và 95%. [Bảng hiện tại](iaa_summary.md), sau khi đồng bộ và chỉnh box trong 101 và 102 theo mục tiêu IoU, ghi 367 cặp, IoU TB 0.901, cùng lớp 99.7%. Các chỉ số tính trên box ghép 1–1 với IoU từ 0.50. Chỉ số sau chỉnh phản ánh kết quả đối chiếu trên nhãn đã sửa.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Bảng ban đầu ghi 110/79 box chưa ghép theo hai phía và 2 box bỏ qua do ignore, với 56 ca loại bất đồng ignore. Bảng hiện tại ghi 25/8, 2 box và 17 ca tương ứng. Xem [danh sách ca](calibration_candidates.csv).
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Ở `img_002.jpg`, hai người có thể cùng bỏ sót một ô tô đủ rõ trên giàn chở. Theo R-04, đối chiếu lần lượt từng xe thật nhìn thấy, đo phần nhìn thấy từ 20 px và kiểm xem mỗi xe đủ điều kiện có box riêng. Đây là điểm cần kiểm trên ảnh.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: [calibration_decisions.csv](calibration_decisions.csv) có 70 ca trên 12 ảnh. C013 đã đối chiếu ảnh gốc và sửa lớp `car` thành `truck` theo R-01 vì pickup có thùng hàng riêng. Nhóm chưa ghi mã người xác nhận cuối trong phiếu QC.
- Tác động v1.1 lên nhãn và ảnh: Bản v1.1 chỉnh câu chữ ở mục Mục tiêu. Phạm vi calibration là 12 ảnh. Dùng nhãn 101 cho `img_008.jpg` và nháp đồng thuận cho 11 ảnh còn lại, sau đó sửa ca C013. Nhãn giữ cùng taxonomy, nên dùng tên lớp để đối chiếu ID COCO.
- Tham chiếu bằng chứng khi Rework: [Bản vòng 1 trước khi sửa C013](archive/round-01-before-pickup-correction/qc_report_auto.md) lưu kết quả của bản cũ. [Vòng 1 hiện tại](reviews/round-01/qc_report_auto.md) gắn với bản đã sửa và có quyết định Hold.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: Phiếu chưa có mã reviewer. Ngày lập báo cáo 17/11/2026.
- Đã duyệt / cả lô: 0/40 ảnh có kết quả duyệt trong phiếu. Danh sách QC vòng 1 đã bao phủ 40/40 ảnh.
- Ảnh lỗi / ảnh đã duyệt: 0/0, chưa tính được tỷ lệ.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/0, chưa tính được tỷ lệ.
- Vật bỏ sót, ảnh và vùng: 0 lỗi bỏ sót được ghi trong phiếu hiện tại. Cần kiểm ảnh gốc để xác định số thực tế.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Đã duyệt 0/12 ca khó và 0/28 ảnh còn lại. Chưa đủ dữ liệu để kết luận lỗi lặp. Ba box còn `can_review` gồm `img_008.jpg` #63, `img_029.jpg` #279 và `img_040.jpg` #407. Xem [phiếu QC vòng 1](reviews/round-01/qc_review.csv).
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script phát hiện box có tọa độ nằm ngoài ảnh. Người đọc ảnh cần phân biệt pickup có thùng riêng với ô tô con và kiểm xem box ôm đúng phần nhìn thấy.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Phiếu chưa ghi nhận ảnh đã duyệt và còn ba box cần xác minh. Nhóm và coach có thể dùng gói hiện tại để tiếp tục rà soát bài thực hành.
- Nếu còn việc: Trưởng nhóm 101 điều phối việc chốt ba box còn cờ, cập nhật COCO và phiếu calibration, tạo vòng mới nếu nhãn thay đổi, rồi phân công reviewer đối chiếu 40 ảnh gốc và ghi kết quả vào phiếu.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: C013 trong [phiếu calibration](calibration_decisions.csv) dẫn tới `img_012.jpg`, R-01 và box #126. [Bản chụp nhãn vòng 1](reviews/round-01/labels_snapshot.json) ghi lớp `truck` cho box này. [qc_result.json](reviews/round-01/qc_result.json) ghi Hold.
