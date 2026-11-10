# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Phương tiện giao thông cơ giới đường bộ cỡ nhỏ phục vụ chở người cá nhân hoặc gia đình, thiết kế từ 4 đến 7 chỗ ngồi. Bao gồm cả xe bán tải (pickup truck). | Sedan, hatchback, SUV, crossover, taxi, xe bán tải cabin đơn/đôi. Phân biệt: Không gán cho xe tải lớn, xe khách hay xe van chuyên dụng. |
| `truck` | Phương tiện cơ giới chuyên dụng chở hàng hóa tải trọng trung bình đến nặng, có cấu trúc phân tách rõ giữa cabin lái và thùng/rơ-moóc chở hàng. | Xe tải thùng, xe ben, xe bồn, xe đầu kéo container. Phân biệt: Xe bán tải nhỏ thuộc lớp `car`; xe van thùng kín liền khối thuộc `van`. |
| `bus` | Phương tiện vận tải hành khách công cộng cỡ trung đến lớn, thiết kế chở nhiều người (thường từ 16-29 chỗ trở lên hoặc xe buýt đô thị). | Xe buýt nội đô, xe khách liên tỉnh, minibus đưa đón tuyến cố định. Phân biệt: Xe van 9-16 chỗ dạng minivan cá nhân/chở hàng thuộc `van`. |
| `van` | Xe có thân hình hộp chữ nhật đặc trưng, cabin liền khối với khoang hàng hoặc khoang chở người, thường có cửa trượt bên hông. | Xe van chở hàng (panel van), minivan gia đình (như Toyota Sienna, Transit, Starex). Phân biệt: Nhỏ hơn xe buýt tuyến (`bus`) và hình dáng hộp vuông vức hơn SUV/sedan (`car`). |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Bounding box phải ôm khít các mép ngoài cùng của phần thân xe thực tế nhìn thấy được (đầu, đuôi, nóc, lốp tiếp xúc mặt đường). Không lấy thêm khoảng trống mặt đường, bầu trời, bóng đổ (shadow) hoặc xe lân cận.
- **Vật thể bị che khuất (Occlusion):** Nếu xe bị che khuất một phần (bởi cây cối, cột đèn, dải phân cách hoặc xe khác), chỉ vẽ bounding box bao quanh phần thân xe nhìn thấy được nếu nhận diện rõ được lớp xe. Nếu bị che bởi thanh vật cản hẹp (như cột đèn), vẫn vẽ 1 box bao trọn thân xe. Hai xe đè lên nhau thì vẽ 2 box riêng biệt cho từng xe.
- **Vật thể bị cắt bởi rìa ảnh (Truncation):** Cạnh box phải chạm đúng mép viền của bức ảnh (tọa độ giới hạn ở 0.0 hoặc 1.0). Tuyệt đối không tự suy đoán hoặc kéo box vượt ra ngoài phạm vi bức ảnh.
- **Vật thể rất nhỏ/mơ hồ:** Nếu vật thể ở quá xa hậu cảnh hoặc quá mờ (kích thước < ~10 pixel) không thể phân biệt chắc chắn là xe gì thì bỏ qua, không phỏng đoán để tránh tăng FP (False Positive).

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Xe sedan màu bạc ở tiền cảnh di chuyển tốc độ cao gây nhòe chuyển động (*motion blur*) kéo dài vệt bóng phía sau trên mặt đường ướt. | Gán `car` (Class 0). Vẽ box ôm sát thân xe cơ học, bỏ qua vệt mờ nhòe kéo dài trên mặt đường. | Box phải ôm sát thân xe thật (tightness), không bao gồm bóng phản chiếu hay vệt nhòe quang học của cảm biến camera. |
| `drive_001.jpg` | Xe SUV màu trắng ở làn giữa bị thanh ngang kim loại của cột đèn/camera che ngang qua thân xe. | Gán `car` (Class 0). Vẽ 1 box duy nhất bao trùm cả xe từ đầu đến đuôi xe nhìn thấy. | Thanh che khuất có bề ngang hẹp, không chia cắt vật thể thành 2 thực thể riêng biệt; xe vẫn được nhận diện toàn vẹn là một chiếc xe con đơn lẻ. |
| `drive_001.jpg` | Các xe ô tô đỗ thành hàng dài dọc vỉa hè phía trước dãy ki-ốt mái đỏ bên kia đường. | Vẫn gán `car` (Class 0) cho từng xe riêng lẻ nhìn rõ hình dạng. | Schema quy định gán nhãn theo loại vật thể (loại xe), không phân biệt trạng thái xe đang chạy hay đang đỗ. |
| `drive_003.jpg` | Xe dạng minivan/thân hộp chở hàng loại nhỏ (microvan/kei van) khó phân biệt giữa `car` (SUV/hatchback) và `van`. | Gán `van` (Class 3) nếu dáng xe hình hộp chữ nhật vuông vức, đuôi phẳng đứng; nếu là SUV/crossover thông thường thì gán `car` (Class 0). | Phân biệt rõ ranh giới: `van` dành cho dòng xe dạng hộp chuyên chở hàng/khách khoang liền, `car` là xe du lịch cá nhân thông thường. |
