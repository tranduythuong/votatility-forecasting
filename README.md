# Data và Feature Engineering

## Vai trò các file

- `scripts/download_data.py`: tải lịch sử giá cổ phiếu và lưu CSV vào `data/raw/`.
- `src/data/loader.py`: đọc CSV dữ liệu thô.
- `src/data/validator.py`: chuẩn hóa tên cột; kiểm tra ngày, OHLCV, giá trị thiếu và ngày trùng.
- `src/data/preprocessing.py`: gọi loader và validator, lưu dữ liệu OHLCV sạch vào `data/processed/`.
- `src/features/builder.py`: tạo các biến đầu vào từ dữ liệu đã làm sạch.
- `src/features/target.py`: tạo nhãn độ biến động tương lai để huấn luyện/đánh giá; không dùng nhãn này làm feature đầu vào.

## Các biến sau khi build

`build_features()` giữ các cột gốc `date`, `open`, `high`, `low`, `close`, `volume` và mặc định bổ sung:

- `return_1d`, `return_5d`, `return_10d`, `return_20d`: log return theo 1, 5, 10 hoặc 20 phiên, nhân 100 để biểu diễn gần theo phần trăm. Cho biết mức và chiều biến động giá trong từng khoảng thời gian.
- `MA_5`, `MA_10`, `MA_20`, `MA_50`: giá đóng cửa trung bình trong 5, 10, 20 hoặc 50 phiên. Giúp nhận biết xu hướng ngắn và dài hạn, đồng thời làm mượt dao động giá hàng ngày.
- `rolling_volatility_5`, `rolling_volatility_20`: độ lệch chuẩn của `return_1d` trong 5 hoặc 20 phiên gần nhất. Đo mức độ biến động gần đây; giá trị cao thường biểu thị rủi ro/dao động lớn hơn.
- `volume_change`: tỷ lệ thay đổi khối lượng giao dịch so với phiên trước. Cho biết hoạt động giao dịch tăng hay giảm; giá trị đầu tiên hoặc trường hợp volume phiên trước bằng 0 sẽ là `NaN`.
- `volume_ma_20`: khối lượng giao dịch trung bình trong 20 phiên. Là mốc so sánh để nhận biết volume hiện tại cao hay thấp so với mức gần đây.
- `high_low_range`: chênh lệch giá cao nhất và thấp nhất trong phiên, chia cho giá đóng cửa. Đo biên độ dao động nội phiên tương đối.
- `open_close_range`: chênh lệch giá đóng cửa và mở cửa, chia cho giá mở cửa. Dương nghĩa là đóng cửa cao hơn mở cửa; âm nghĩa là thấp hơn.

Các rolling feature chỉ dùng dữ liệu tại ngày hiện tại và quá khứ. Những dòng đầu chưa đủ lịch sử để tính cửa sổ sẽ có `NaN`; cần xử lý các dòng này có chủ đích trước khi huấn luyện.

`add_future_volatility_target()` mặc định tạo `target_volatility_5d`, là độ lệch chuẩn return trong 5 phiên kế tiếp. Đây là biến mục tiêu tương lai, không phải đầu vào mô hình.
