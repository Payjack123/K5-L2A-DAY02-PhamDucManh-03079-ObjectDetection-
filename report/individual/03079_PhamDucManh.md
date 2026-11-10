# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phạm Đức Mạnh
- MSSV: 03079
- Email: manh.pd03079@...
- Lớp: K5 Level 2A (L2A)
- Link repo bài làm: https://github.com/Payjack123/K5-L2A-DAY02-PhamDucManh-03079-ObjectDetection-
- Link Colab đã chạy: [Sẽ cập nhật sau khi chạy Colab]

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
- Tổng số bounding box đã gán: 1,141 box (Car: 983, Truck: 18, Bus: 92, Van: 48).
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` ĐẠT cấu trúc 50/50 file nhãn hợp lệ, đúng 5 cột normalized trong `[0, 1]`, không có box vượt biên.
- Danh sách 10 ảnh đã rà bằng mắt: `drive_001.jpg`, `drive_002.jpg`, `drive_005.jpg`, `drive_008.jpg`, `drive_012.jpg`, `drive_015.jpg`, `drive_020.jpg`, `drive_025.jpg`, `drive_035.jpg`, `drive_045.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Rà soát các xe ở tiền cảnh bị mờ chuyển động: điều chỉnh mép box bám sát thân xe thật, loại bỏ vệt kéo dài.
  - Kiểm tra các xe đỗ sát nhau ở rìa đường để đảm bảo tách biệt mỗi xe một box độc lập, không gộp 2 xe làm 1 box.
- Commit khóa nhãn ban đầu: `80c9f59`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | | |
| FP | | |
| FN | | |
| Precision | | |
| Recall | | |
| F1 | | |
| mean_iou_of_matches | | |
| passed | | |

*Dẫn tới `outputs/annotation_evaluation.json` trong repo sau khi nhận gold labels từ giảng viên.*

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
- Các tham số thay đổi so với notebook nếu có và lý do:
- Đường dẫn best checkpoint trong phiên chạy:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

*Các thư mục assets từ Colab:*
- `assets/eval_predictions/visuals/`
- `assets/yolo26_runs/`

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - [Sẽ trả lời dựa trên kết quả chạy thực tế trên Colab]
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra độ bám sát (tightness) và hình học của bounding box. mAP50 cao nghĩa là mô hình đã định vị đúng vùng thô của xe và phân loại đúng class ở ngưỡng chồng khít lỏng (IoU 0.50). Tuy nhiên khi tăng ngưỡng IoU từ 0.50 lên 0.95, nếu viền box bị vẽ rộng (lỏng) hoặc lệch thì IoU thực tế sẽ thấp, làm mAP ở các ngưỡng IoU cao tụt mạnh.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Sự nhầm lẫn giữa `car` (SUV/bán tải) và `truck`/`van`, hoặc giữa `bus` và `van`. Dữ liệu cho thấy sự mất cân bằng lớp rất lớn (lớp `car` chiếm hơn 80% tổng số box, trong khi `truck` và `van` rất ít).
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train vì sẽ gây ra hiện tượng rò rỉ dữ liệu (data leakage), khiến mô hình "học vẹt" đáp án và ta không đánh giá được khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu mới chưa từng thấy. 10 ảnh eval đóng vai trò validation set độc lập để đánh giá khách quan hiệu năng và chọn checkpoint tối ưu (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn (CP2) | Ảnh chụp lúc trời mưa, xe chạy nhanh bị vệt mờ chuyển động (motion blur) | Xem kỹ cấu trúc xe và bám sát khung thân xe vật lý | Box bám sát, tăng IoU |
| Nhầm lẫn lớp | Xe minivan chở hàng và xe khách nhỏ | Đối chiếu quy định schema trong GUIDE.md | Phân loại nhất quán |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- Đã hoàn thành 50 file nhãn YOLO chuẩn hóa, pass checker 50/50.
- Đã có mini guideline và ghi nhận các ca mơ hồ.
- Chờ nhận gold set để chạy QC và chạy notebook Colab để lấy metrics hoàn thiện mục 4 và 5.

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng AI hỗ trợ tra cứu khái niệm, kiểm tra cú pháp và định dạng file nhãn.
- Tự thực hiện toàn bộ việc gán nhãn 50 ảnh trên CVAT và tự kiểm tra chất lượng nhãn.
