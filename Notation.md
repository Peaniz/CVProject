# Đề tài: Xác định và phân loại phương tiện giao thông và người điều khiển phương tiện

## 1. Xác định vấn đề:

### a. Câu hỏi đặt ra:
- Hướng, góc nhìn của camera sẽ là cố định hay là flexible? Có thể tùy ý thay đổi góc nhìn nhưng vẫn nhận diện được không.
- Phát hiện phương tiện và người sử dụng các phương pháp gì? Dataset lấy ở đâu?
- Bài toán này giúp giải quyết được vấn đề gì? Hỗ trợ cho những yêu cầu nào trong quản lí giao thông?
- Có áp dụng đầy đủ các yếu tố trong môn học: enhancement, filtering, segmentation, morphology, colour và restoretation? 

### b. Hướng giải quyết vấn đề: 
- Sử dụng các mô hình detection deep learning: 
  - [YOLOv8/v7](https://github.com/ultralytics/ultralytics)
  - [Detectron2 (Facebook)](https://github.com/facebookresearch/detectron2)
  - [EfficientDet](https://github.com/xuannianz/EfficientDet)

- Sử dụng các mô hình trả bounding box cho các vật thể cần được phân loại. 
- Phân đoạn vùng xe qua thresholding/adaptive thresholding -> morphology để tạo thành vùng liên tục
- Gợi ý thực tế: dùng DL detector + postprocess bằng morphology/tracking (để ổn định) là hướng tốt nhất.
- Loại trừ false positive: Tránh detect những người đang đi bộ vào thuộc diện đang trên xe (Có thể sử dụng [ByteTrack](https://github.com/FoundationVision/ByteTrack)/ [DeepSORT](https://github.com/nwojke/deep_sort) để gắn ID qua frame và set thời gian trên xe nếu overlap liên tục)


## 2. Các implement extra (có thể thực hiện thêm)

### a. Xác định trạng thái người trên phương tiện: 
- Xác định người ở trong xe: có thể thông qua việc overlap bounding box giữa người và xe.
- Đếm số người ở trong xe: Dựa vào số lượng bounding box người overlap với xe mà đánh giá.
- Hành vi hiện tại: Hiện tại khá là khó để xác định vì chỉ thực hiện trên một đoạn video tĩnh -> tuy nhiên có thể sử dụng Mediapipe để xác định tư thế ngồi

### b. Tracking ổn định kết quả:

- Dùng tracker (ByteTrack / DeepSORT) để giữ ID vehicle & person, cho phép đếm, tần suất xuất hiện, và giúp tránh nhiễu tạm thời.
- Có thể loại bỏ background tĩnh để tăng hiệu ứng phát hiện xe chuyển động (sử dụng background subtraction)
- Ánh sáng không đều / bóng / ban đêm: dùng adaptive thresholding / histogram equalization; nếu ban đêm nặng, cân nhắc IR/thermal camera
- Biến thể kích thước & góc nhìn: DL detector + multi-scale anchor/predictor. (slides cung cấp nền tảng xử lý ảnh nhưng DL là cần thiết để robust).
- Nhiễu & mưa: dùng spatial filters (median) và restoration nếu cần (slides về restoration). 
## 3. Dữ liệu và huấn luyện:
-  cần dataset có bounding boxes cho cả vehicle classes và people; ideal: có ảnh giao thông VN (góc camera, xe máy phổ biến). Bạn có thể dùng COCO (cơ bản) → sau đó fine-tune bằng dữ liệu thực tế (camera của bạn).
- Gắn nhãn: vehicle_type, person, rider_on_vehicle (optionally). Tạo annotation cho overlap cases -> dùng để huấn luyện classifier post-process nếu overlap rule gây nhiều lỗi.
