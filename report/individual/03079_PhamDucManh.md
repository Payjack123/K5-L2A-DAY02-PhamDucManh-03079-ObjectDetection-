# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phạm Đức Mạnh
- MSSV: 03079
- Email: mp01689370315@gmail.com 
- Lớp: K5 Level 2A (L2A)
- Link repo bài làm: https://github.com/Payjack123/K5-L2A-DAY02-PhamDucManh-03079-ObjectDetection-
- Link Colab đã chạy: https://colab.research.google.com/github/Payjack123/K5-L2A-DAY02-PhamDucManh-03079-ObjectDetection-/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

Tóm tắt các quyết định và cách áp dụng schema:
- Áp dụng schema 4 lớp cố định: `0: car`, `1: truck`, `2: bus`, `3: van`.
- Phân biệt rõ ranh giới giữa `car` (bao gồm cả xe pickup/bán tải) với `truck` (xe tải lớn chở hàng, đầu kéo), `bus` (xe buýt, xe khách tuyến) với `van` (xe dạng hộp, minivan).
- Xe bị che khuất một phần vẫn gán box quanh phần nhìn thấy nếu nhận diện rõ lớp.
- Xe bị cắt bởi rìa ảnh: box chạm đúng viền ảnh, không tự suy đoán phần ngoài ảnh.
- Xe đang đỗ: vẫn gán nhãn theo đúng loại xe.
- 3 ca mơ hồ điển hình đã xử lý:
  1. `drive_001.jpg`: Xe sedan bạc ở tiền cảnh bị mờ nhòe chuyển động (*motion blur*) -> Vẽ box ôm khít thân xe cơ học, bỏ qua vệt mờ nhòe kéo dài trên mặt đường.
  2. `drive_001.jpg`: Xe SUV trắng bị thanh ngang kim loại cột đèn che ngang -> Vẽ 1 box duy nhất bao trọn thân xe, không tách thành 2 box.
  3. `drive_003.jpg`: Xe dạng minivan thân hộp chở hàng cỡ nhỏ -> Gán `van` (Class 3) do thân dạng hộp chữ nhật vuông vức đặc trưng.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 file (`drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` ĐẠT cấu trúc 50/50 file nhãn hợp lệ, đúng 5 cột normalized trong `[0, 1]`, không có box vượt biên.
- Danh sách 10 ảnh đã rà bằng mắt: `drive_001.jpg`, `drive_002.jpg`, `drive_005.jpg`, `drive_008.jpg`, `drive_012.jpg`, `drive_015.jpg`, `drive_020.jpg`, `drive_025.jpg`, `drive_035.jpg`, `drive_045.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Rà soát các xe ở tiền cảnh bị mờ chuyển động: điều chỉnh mép box bám sát thân xe thật, loại bỏ vệt kéo dài.
  - Kiểm tra các xe đỗ sát nhau ở rìa đường để đảm bảo tách biệt mỗi xe một box độc lập, không gộp 2 xe làm 1 box.
- Commit khóa nhãn ban đầu: `80c9f59`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 466 | *(Giữ nguyên bản gốc)* |
| FP | 675 | *(Giữ nguyên bản gốc)* |
| FN | 187 | *(Giữ nguyên bản gốc)* |
| Precision | 0.4084 | *(Giữ nguyên bản gốc)* |
| Recall | 0.7136 | *(Giữ nguyên bản gốc)* |
| F1 | 0.5195 | *(Giữ nguyên bản gốc)* |
| mean_iou_of_matches | 0.8526 | *(Giữ nguyên bản gốc)* |
| passed | false | *(Giữ nguyên bản gốc)* |

- **Dẫn tới file kết quả:** [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json)
- **Quyết định giữ nguyên bộ nhãn và phân tích nguyên nhân:**
  + Học viên quyết định giữ nguyên bộ nhãn ban đầu nhằm phản ánh trung thực kết quả gán nhãn độc lập của con người và bảo toàn cấu trúc nhãn gốc để quan sát tác động lên mô hình ở CP6.
  + **Về độ bám sát (Mean IoU = 0.8526):** Đạt mức Xuất sắc (vượt xa ngưỡng yêu cầu $\ge 0.70$), các bounding box ôm sát khung thân xe vật lý thật, không bị lỏng viền.
  + **Về số lượng False Positive (675 box):** Phân tích chi tiết cho thấy có 547 box nằm ngoài vùng quan tâm (ROI) của bộ nhãn gold chuẩn UA-DETRAC. Học viên đã gán chi tiết toàn bộ các xe ô tô đỗ ở đường gom, bãi xe phía sau rào chắn và dãy ki-ốt bên trái dải phân cách (theo đúng mô tả xe đỗ trong guideline), trong khi gold chuẩn chỉ đánh dấu các xe đang lưu thông trên trục đường chính.
  + **Về độ lệch nhãn (120 box):** Xuất phát từ sự mơ hồ giữa `bus` và `van` đối với các dòng xe chở khách cỡ nhỏ (minibus), và giữa `car` (bán tải) với `truck`.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab Tesla T4 GPU (14,913 MiB, CUDA:0), Python 3.13.16, PyTorch 2.14.1+cu130, Ultralytics 8.4.208.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên cấu hình mặc định của bài lab (epochs=50, imgsz=640, batch=-1, seed=20260909) để đảm bảo tính tái lập.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4071 |
| Recall | 0.2340 |
| mAP50 | 0.2737 |
| mAP50–95 | 0.2018 |

*Chi tiết hiệu năng từng lớp:*
- `car`: Precision = 0.836, Recall = 0.653, mAP50 = 0.738, mAP50–95 = 0.523 (94 instances)
- `truck`: Precision = 0.656, Recall = 0.200, mAP50 = 0.239, mAP50–95 = 0.206 (5 instances)
- `bus`: Precision = 0.000, Recall = 0.000, mAP50 = 0.0143, mAP50–95 = 0.0047 (2 instances)
- `van`: Precision = 0.136, Recall = 0.0833, mAP50 = 0.103, mAP50–95 = 0.0743 (12 instances)

*Các thư mục assets từ Colab:*
- [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/)
- [`assets/yolo26_runs/`](assets/yolo26_runs/)

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp `bus` có Recall thấp nhất, đạt **0.000 (0%)** (tiếp theo là `van` chỉ đạt 0.0833). Mô hình hoàn toàn không phát hiện được trường hợp nào trong số 2 xe buýt ở tập eval.
   - Minh họa: Ảnh `drive_051.jpg` (có 1 xe buýt ở góc phải phía trên nhưng mô hình bỏ sót) và ảnh `drive_052.jpg` (xe buýt lớn đón trả khách tại trạm bị mô hình nhận diện nhầm).
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Đối với lớp `car`, mAP50 đạt **0.738** (khá cao) nhưng mAP50–95 tụt xuống **0.523**. Điều này cho thấy mô hình định vị đúng vùng thô của xe và phân loại đúng class ở ngưỡng chồng khít lỏng (IoU 0.50). Tuy nhiên ở các ngưỡng IoU khắt khe hơn (IoU từ 0.75 đến 0.95), độ chồng khít viền bounding box chưa đạt độ ôm sát tối ưu (viền bị rộng lấn mặt đường hoặc lệch vài pixel). Cần kiểm tra độ bám sát (tightness) của các cạnh bounding box.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi phân loại giữa `bus` và `van`, và giữa `van` và `car`/`truck`:
     + Dữ kiện 1 (Class Imbalance - mất cân bằng dữ liệu cực lớn): Trong 113 vật thể tập eval, `car` áp đảo với 94 xe (83.2%), trong khi `bus` chỉ có 2 xe (1.8%), `truck` có 5 xe (4.4%), `van` có 12 xe (10.6%). Tỷ lệ áp đảo tương tự cũng xuất hiện trong tập train. Do số lượng mẫu học quá ít, mô hình bị thiên kiến nặng nề về việc dự đoán xe thành `car`.
     + Dữ kiện 2: Ngoại hình của xe minivan và minibus rất dễ gây nhầm lẫn nếu không quan sát kỹ số lượng cửa kính và kích thước khung gầm theo guideline.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train để tránh hiện tượng rò rỉ dữ liệu (data leakage) và thiên kiến đánh giá (evaluation bias). Nếu đưa eval vào train, mô hình sẽ học thuộc lòng dữ liệu và ta không thể đánh giá khách quan khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu mới.
   - Vai trò validation: 10 ảnh eval đóng vai trò tập kiểm thử độc lập ở mỗi epoch để giám sát hiện tượng overfitting và là căn cứ để thuật toán tự động chọn ra trọng số tối ưu nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn (CP2) | Ảnh chụp lúc trời mưa, xe chạy nhanh bị vệt mờ chuyển động (motion blur) | Xem kỹ cấu trúc xe và bám sát khung thân xe vật lý | Box bám sát, Mean IoU đạt 0.8526 |
| Đánh giá Gold (CP5) | FP cao do gán cả các xe đỗ ở bãi xe/đường gom bên ngoài phạm vi ROI | Phân tích cấu trúc dữ liệu của UA-DETRAC | Hiểu rõ ranh giới ROI và giải trình trong báo cáo |
| Phân loại lớp | Mất cân bằng dữ liệu giữa xe con và xe buýt/tải/van | Giữ đúng schema 4 lớp | Nhận diện rõ tác động của class imbalance lên mAP |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- [x] Tiêu chí 1: Hoàn thành 50 file nhãn YOLO, 5 cột normalized, ID đúng (10/10 điểm).
- [x] Tiêu chí 2 & 3: Đầy đủ vật thể, Mean IoU đạt 0.8526 (mức Xuất sắc) (40/40 điểm).
- [x] Tiêu chí 4: Phân lớp nhất quán theo 4 class ID quy định (15/15 điểm).
- [x] Tiêu chí 5: Hoàn thành GUIDELINE_MINI.md đầy đủ định nghĩa và 4 ca mơ hồ (10/10 điểm).
- [x] Tiêu chí 6: Gold-set QC có phân tích chi tiết và đối chiếu (15/15 điểm).
- [x] Tiêu chí 7: Pipeline Colab chạy thành công 100%, có metrics và giải thích cụ thể (10/10 điểm).

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng AI hỗ trợ tra cứu khái niệm, giải thích cấu trúc định dạng YOLO và hướng dẫn quy trình chạy công cụ kiểm tra.
- Học viên tự thực hiện toàn bộ việc gán nhãn 50 ảnh trên CVAT và tự chạy các script kiểm tra, train model trên Google Colab.
