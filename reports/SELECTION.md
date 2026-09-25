# Vì sao chọn lô này?

Nếu chỉ có ngân sách rà năm ảnh, tôi ưu tiên `frame_0182.jpg` (hạng 1, điểm 0.9591, 72.8 giây), `frame_0369.jpg` (hạng 2, điểm 0.9324, 147.6 giây), `frame_0380.jpg` (hạng 3, điểm 0.9170, 152.0 giây), `frame_0326.jpg` (hạng 4, điểm 0.9155, 130.4 giây) và `frame_0331.jpg` (hạng 5, điểm 0.9154, 132.4 giây). Đây là năm ảnh đứng đầu bảng điểm và đều được chọn. `frame_0326.jpg` với `frame_0331.jpg` cách nhau đúng 2.0 giây, còn `frame_0369.jpg` với `frame_0380.jpg` cách nhau 4.4 giây, nên vẫn thỏa điều kiện giãn cách tối thiểu của lô.

Ba ảnh tiêu biểu trong lô là `frame_0182.jpg`, `frame_0369.jpg` và `frame_0099.jpg`. `frame_0182.jpg` có U = 0.9182, A = 1.0000, 28 box và 18 box mơ hồ; `frame_0369.jpg` có U = 0.9315, A = 0.8889, 43 box và 16 box mơ hồ; `frame_0099.jpg` có U = 0.9460, A = 0.7778, 29 box và 14 box mơ hồ. Ảnh contact sheet cho thấy cả ba đều là cảnh đêm đông xe, có nhiều xe nhỏ và ánh đèn mạnh, nên vừa có độ bất định cao vừa tốn công rà nhãn.

`frame_0372.jpg` đứng hạng 6 với điểm 0.9101 tại 148.8 giây nhưng không được chọn. Ảnh này chỉ cách `frame_0369.jpg` tại 147.6 giây có 1.2 giây, nhỏ hơn `MIN_GAP_S = 2.0`; hai ảnh gần như cùng một cảnh nên rà cả hai sẽ tăng chi phí mà ít bổ sung thông tin mới.

Điểm cao chỉ cho biết mô hình đang không chắc chắn, có nhiều box mơ hồ hoặc ảnh giúp tăng độ đa dạng theo thời gian. Điểm này không chứng minh rằng sửa ảnh rồi fine-tune sẽ làm mô hình tốt hơn; điều đó chỉ có thể đánh giá sau khi train và đo lại trên cùng tập kiểm thử tách biệt.
