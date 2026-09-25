# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Ngo Xuan Nam

Công cụ gán nhãn đã dùng: CVAT

## 1. Dữ liệu và cách chia tập

Pool và tập kiểm thử được chia theo trục thời gian, có vùng đệm ở giữa, vì camera đứng yên và một chiếc xe thường xuất hiện trong nhiều frame liên tiếp. Nếu chia ngẫu nhiên, các ảnh gần như trùng nhau của cùng một xe có thể vừa nằm trong tập train vừa nằm trong tập test. Đây là rò rỉ dữ liệu: mô hình được chấm trên cảnh rất giống cảnh đã học, nên AP50 và recall có xu hướng cao hơn khả năng tổng quát hóa thật. Chia theo thời gian giúp tập test độc lập hơn với các ảnh dùng để fine-tune.

## 2. Mô hình khởi đầu lạnh (cold start)

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |

Mô hình khởi đầu có AP50 = 0.7714, precision = 0.9249 và recall = 0.4888. Recall của xe nhỏ chỉ là 0.1818, thấp hơn rõ rệt xe vừa 0.5473 và xe lớn 0.5610, cho thấy mô hình bỏ sót nhiều xe ở xa. Ảnh `compare_round0.jpg` cũng cho thấy số bỏ sót tăng ở cảnh đông: `frame_0050` có 7 FN, `frame_0150` có 10 FN, `frame_0250` có 9 FN và `frame_0350` có 14 FN. Các xe nhỏ, tối, bị che hoặc chỉ còn cụm đèn dễ không khớp nhãn tham chiếu.

Một trường hợp cần người rà lại nhãn tham chiếu là xe rất xa chỉ còn hai chấm đèn, hoặc xe bị cắt ở mép ảnh. Guideline cho phép bỏ qua xe quá nhỏ, còn nhãn test hiện do mô hình tạo và chưa được người kiểm từng box. Vì vậy một FN tại các vị trí này có thể đến từ nhãn tham chiếu chưa chuẩn, không nhất thiết hoàn toàn là lỗi của mô hình được đánh giá.

## 3. Chiến lược chọn mẫu

Điểm chọn mẫu được tính theo `score = 0.5·U + 0.3·A + 0.2·D`. Thành phần U đo mức mô hình không chắc chắn, A tăng khi ảnh có nhiều box mơ hồ cần rà, còn D khuyến khích lấy ảnh cách xa về thời gian để lô đa dạng hơn. `MIN_GAP_S = 2.0` loại các frame cách một ảnh đã chọn dưới hai giây, vì camera đứng yên khiến các frame sát nhau gần như trùng lặp.

Trong `SELECTION.md`, `frame_0182.jpg` đứng hạng 1 với điểm 0.9591 và 18 box mơ hồ; `frame_0369.jpg` đứng hạng 2 với điểm 0.9324, có 43 box dự đoán; `frame_0099.jpg` đứng hạng 8 với điểm 0.9063 và U = 0.9460. Ngược lại, `frame_0372.jpg` có điểm cao 0.9101 nhưng không được chọn vì thời điểm 148.8 giây chỉ cách `frame_0369.jpg` 1.2 giây. Việc bỏ ảnh gần trùng giúp giảm công gán nhãn mà vẫn giữ độ đa dạng của lô. Điểm bất định không chứng minh ảnh sẽ cải thiện mô hình: nó chỉ là tiêu chí đề xuất ảnh đáng xem, còn hiệu quả phải được kiểm bằng kết quả sau fine-tune trên tập test độc lập.

## 4. Các vòng học chủ động (active learning)

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vong 1..1 | 12 | 321 | 0.357 | -0.415 | 1.000 | 0.040 | 0.076 | 0.000 | 0.020 | 0.244 |

Ở vòng 1, mô hình đề xuất 169 box trên 12 ảnh. Sau khi rà bằng CVAT còn 321 box: giữ nguyên 148, chỉnh 8, xóa 13 và thêm 165; accept rate là 88%. AP50 giảm từ 0.7714 xuống 0.3567, tức giảm 0.4147 so với cold start và cũng là mức giảm so với vòng trước. Recall tổng giảm từ 0.4888 xuống 0.0397. Recall xe nhỏ giảm từ 0.1818 xuống 0, xe vừa từ 0.5473 xuống 0.0203, và xe lớn từ 0.5610 xuống 0.2439. Precision đạt 1.0 vì vòng 1 chỉ phát hiện 16 TP và không có FP tại ngưỡng 0.25; con số này không bù được 387 FN.

`compare_round1.jpg` cho thấy một thay đổi xấu rõ ràng ở `frame_0050`: cold start tìm đúng 11 xe và bỏ sót 7, nhưng vòng 1 không tìm đúng xe nào và bỏ sót 18. Tương tự, `frame_0150` giảm từ 10 TP xuống 1 TP. Với chỉ 12 ảnh train, kết quả này phù hợp với khả năng mô hình bị quá khớp, lệch phân bố hoặc bị thay đổi độ tự tin sau fine-tune; cần kiểm tra chứ chưa thể kết luận học chủ động có lợi.

Ba nguồn quan sát có vai trò khác nhau. `BLIND_SCAN.md` là quan sát độc lập trước khi xem pre-label: ở `frame_0099`, tôi đếm 23 xe và ghi nhận xe bị cắt mép cùng các xe rất xa chỉ còn đèn. `REVIEW_LOG.csv` ghi thao tác sửa nhãn, gồm tách khung gộp hai xe nhỏ và thu gọn các khung bao dư. `round1_diff.md` tổng hợp toàn lô và cho thấy đã thêm tới 165 box. Các dữ liệu này nói về chất lượng nhãn train, còn `compare_round1.jpg` nói về hành vi mô hình sau khi train. Ca khó tiêu biểu là xe rất xa chỉ còn hai chấm đèn: guideline cho phép gán hoặc bỏ nếu box quá nhỏ, và 14 box tham chiếu cao dưới 16 px đã bị loại khỏi phép chấm.

## 5. Kết luận và giới hạn

Vòng 1 kém hơn cold start: AP50 giảm 0.4147 và recall giảm 0.4491. Tôi dừng để rà lại dữ liệu và cấu hình train trước khi làm vòng 2, thay vì tiếp tục đưa nhãn mới vào một quy trình đang cho kết quả giảm mạnh. Trước khi train thêm, tôi sẽ mở chồng ảnh với nhãn để kiểm class id, tọa độ chuẩn hóa, đường dẫn export CVAT, tính nhất quán của 321 box và các trường hợp thêm nhiều box; sau đó kiểm learning rate, số epoch, ngưỡng confidence và dấu hiệu quá khớp trên chỉ 12 ảnh.

Hai nhóm nên ưu tiên nếu tiếp tục là xe nhỏ ở xa chỉ còn đèn và xe tối, nhoè hoặc bị cắt ở mép ảnh. Các ca này tốn công vì phải phóng to, phân biệt xe với vệt sáng và quyết định phần thân còn nhìn thấy; ảnh quay sát thời gian cũng dễ gần trùng nên phải giữ khoảng cách tối thiểu. Kết luận hiện còn yếu vì tập test chỉ có 20 ảnh, bỏ qua 14 xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được người rà thủ công. Vì vậy mức giảm AP50 là tín hiệu cảnh báo mạnh, nhưng độ lớn chính xác của thay đổi vẫn có thể chịu ảnh hưởng của mẫu test nhỏ và lỗi nhãn tham chiếu.
