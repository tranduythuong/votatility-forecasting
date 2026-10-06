==================================================
RULES: DATA + FEATURE ENGINEERING + GARCH
Dataset: VN-Index
==================================================

I. DATA RULES
--------------------------------------------------

RULE D1:
Dữ liệu đầu vào là dữ liệu lịch sử VN-Index theo ngày giao dịch.

RULE D2:
Dữ liệu tối thiểu phải có các cột:
date, open, high, low, close, volume.

RULE D3:
Cột date phải được chuyển sang kiểu datetime.

RULE D4:
Dữ liệu phải được sắp xếp theo date tăng dần trước khi thực hiện
bất kỳ feature engineering hoặc GARCH nào.

RULE D5:
Phải kiểm tra và xử lý duplicate date.

RULE D6:
Không được tự ý tạo dữ liệu cho những ngày VN-Index không giao dịch.

RULE D7:
Không được nội suy hoặc fill giá OHLC bằng 0.

RULE D8:
Các giá trị OHLC phải hợp lệ:
high >= max(open, close)
low <= min(open, close)
high >= low.

RULE D9:
Giá trị close phải > 0 trước khi tính return.

RULE D10:
Volume không được âm.

RULE D11:
Raw data phải được giữ nguyên.
Không ghi đè raw dataset sau khi preprocessing.

RULE D12:
Mọi bước xử lý dữ liệu phải có thể tái lập từ raw data.

RULE D13:
Phải kiểm tra missing values trước khi feature engineering.

RULE D14:
Không được dùng dữ liệu tương lai để xử lý dữ liệu tại thời điểm t.

RULE D15:
Mỗi feature tại thời điểm t chỉ được sử dụng thông tin có thể
quan sát được tại thời điểm t hoặc trước đó.


==================================================
II. RETURN RULES
--------------------------------------------------

RULE R1:
GARCH không được áp dụng trực tiếp lên giá VN-Index.
GARCH phải được xây dựng trên return.

RULE R2:
Ưu tiên sử dụng log return:

return_t = ln(close_t / close_(t-1))

RULE R3:
Không được sử dụng close_(t+1), return_(t+1) hoặc bất kỳ dữ liệu
tương lai nào để tính return_t.

RULE R4:
Có thể nhân log return với 100 để biểu diễn theo phần trăm,
nhưng phải sử dụng nhất quán trong toàn bộ pipeline.

RULE R5:
Phải kiểm tra các giá trị inf và NaN sau khi tính return.

RULE R6:
Dòng đầu tiên sau khi tính return sẽ có NaN do không có
close_(t-1). Không được tự ý thay bằng 0.


==================================================
III. FEATURE ENGINEERING RULES
--------------------------------------------------

RULE F1:
Feature engineering phải được thực hiện theo thứ tự thời gian.

RULE F2:
Mọi rolling feature tại thời điểm t chỉ được sử dụng dữ liệu
từ t trở về quá khứ.

RULE F3:
Không được sử dụng centered rolling window.

RULE F4:
Không được sử dụng shift âm trong feature engineering.

RULE F5:
Không được sử dụng dữ liệu tương lai để tạo feature.

RULE F6:
Moving Average phải được tính từ giá lịch sử:

MA_n(t) = mean(close[t-n+1 : t])

RULE F7:
Rolling volatility phải được tính từ return, không phải trực tiếp
từ giá.

RULE F8:
Rolling volatility n ngày:

volatility_n(t) = std(return[t-n+1 : t])

RULE F9:
Các feature cơ bản có thể bao gồm:

return_1d
return_5d
return_10d
return_20d
MA_5
MA_10
MA_20
MA_50
rolling_volatility_5
rolling_volatility_20
volume_change
volume_ma_20
high_low_range
open_close_range

RULE F10:
High-low range:

high_low_range =
(high - low) / close

RULE F11:
Open-close range:

open_close_range =
(close - open) / open

RULE F12:
Volume change:

volume_change =
(volume_t - volume_(t-1)) / volume_(t-1)

RULE F13:
Feature có rolling window phải chấp nhận rằng một số dòng đầu
sẽ không đủ dữ liệu.

RULE F14:
Không được fill các rolling volatility NaN bằng 0 vì 0 có ý nghĩa
là không có biến động.

RULE F15:
Các dòng warm-up không đủ dữ liệu phải được loại bỏ hoặc xử lý
một cách có chủ đích.

RULE F16:
Không tạo quá nhiều feature trùng lặp chỉ để tăng số lượng cột.

RULE F17:
Mỗi feature phải có tên rõ ràng và thể hiện window/time horizon,
ví dụ MA_20, volatility_20, return_5d.

RULE F18:
Feature phải có ý nghĩa tài chính và phải giải thích được công thức.


==================================================
IV. GARCH RULES
--------------------------------------------------

RULE G1:
GARCH được sử dụng để mô hình hóa conditional volatility/
conditional variance của return VN-Index.

RULE G2:
Không fit GARCH trực tiếp trên close price.

RULE G3:
Input chính của GARCH là chuỗi return đã được sắp xếp theo thời gian.

RULE G4:
Mô hình cơ sở phải bắt đầu bằng GARCH(1,1).

RULE G5:
GARCH(1,1) có dạng:

sigma_t^2 =
omega
+ alpha * epsilon_(t-1)^2
+ beta * sigma_(t-1)^2

RULE G6:
Phải kiểm tra kết quả fit GARCH có hội tụ hay không.

RULE G7:
Nếu GARCH không hội tụ hoặc tham số không hợp lệ,
phải báo lỗi/thông báo thay vì âm thầm sử dụng kết quả.

RULE G8:
Không được fit GARCH trên toàn bộ dataset rồi sau đó sử dụng
kết quả đó như feature cho các thời điểm trong quá khứ.

RULE G9:
GARCH phải tuân thủ nguyên tắc information cutoff:
tại thời điểm t chỉ được sử dụng return <= t.

RULE G10:
Khi tạo GARCH forecast tại thời điểm t, forecast phải đại diện
cho volatility có thể dự đoán từ thông tin đến thời điểm t.

RULE G11:
Không được sử dụng realized future volatility để tạo
GARCH forecast.

RULE G12:
Có thể lưu các output của GARCH:

garch_conditional_variance
garch_conditional_volatility
garch_forecast_1d

RULE G13:
Conditional volatility được tính:

garch_volatility_t = sqrt(garch_variance_t)

RULE G14:
Nếu sử dụng forecast nhiều bước, phải ghi rõ horizon:

garch_forecast_1d
garch_forecast_5d
...

Không được gọi chung là garch_forecast nếu không xác định horizon.

RULE G15:
Có thể thử các distribution khác nhau như:
normal
student-t

Nhưng GARCH(1,1) + một distribution cơ bản phải được triển khai
trước khi mở rộng.

RULE G16:
Nếu return có dấu hiệu fat-tail, Student-t distribution nên được
xem xét.

RULE G17:
Không được tự ý thay đổi GARCH order chỉ để cải thiện kết quả.
Mọi thay đổi từ GARCH(1,1) phải được ghi nhận rõ ràng.

RULE G18:
Nếu sử dụng asymmetric GARCH như EGARCH hoặc GJR-GARCH,
phải xem đây là mô hình mở rộng và không được trộn lẫn với
GARCH cơ bản.


==================================================
V. DATA LEAKAGE RULES
--------------------------------------------------

RULE L1:
Đây là quy tắc quan trọng nhất:
KHÔNG ĐƯỢC ĐỂ THÔNG TIN TƯƠNG LAI XUẤT HIỆN TRONG FEATURE.

RULE L2:
Feature tại ngày t chỉ được sử dụng:
date <= t.

RULE L3:
Không sử dụng:
close_(t+1)
return_(t+1)
high_(t+1)
low_(t+1)
volume_(t+1)
future volatility

để tạo feature tại t.

RULE L4:
Không được fit rolling statistics bằng toàn bộ dataset rồi gán ngược
cho toàn bộ thời gian.

RULE L5:
Không được fit GARCH bằng dữ liệu tương lai khi tạo historical
GARCH feature.

RULE L6:
Mọi transformation phải bảo toàn temporal order.

RULE L7:
Nếu code có nguy cơ data leakage, AI phải cảnh báo thay vì
tự động thực hiện.


==================================================
VI. OUTPUT DATASET RULES
--------------------------------------------------

RULE O1:
Sau Data + Feature Engineering + GARCH, dataset có thể có dạng:

date
open
high
low
close
volume

return_1d
return_5d
return_10d
return_20d

MA_5
MA_10
MA_20
MA_50

rolling_volatility_5
rolling_volatility_20

volume_change
volume_ma_20

high_low_range
open_close_range

garch_conditional_variance
garch_conditional_volatility
garch_forecast_1d

RULE O2:
Phải giữ date làm index hoặc column rõ ràng.

RULE O3:
Không được có duplicate date trong processed dataset.

RULE O4:
Sau feature engineering phải kiểm tra:
NaN
inf
duplicate
wrong datatype
invalid numerical values.

RULE O5:
Phải ghi lại số dòng trước và sau preprocessing.

RULE O6:
Phải ghi rõ số dòng bị loại do rolling/GARCH warm-up.

RULE O7:
Processed dataset phải được lưu riêng với raw dataset.

RULE O8:
Tên file phải thể hiện trạng thái dữ liệu, ví dụ:

vnindex_raw.csv
vnindex_features.csv
vnindex_garch.csv


==================================================
VII. CODE QUALITY RULES
--------------------------------------------------

RULE C1:
Không viết toàn bộ pipeline thành một block code duy nhất.

RULE C2:
Tách tối thiểu thành các bước:

load_data()
clean_data()
calculate_returns()
create_features()
fit_garch()
create_garch_forecast()
save_processed_data()

RULE C3:
Mỗi function chỉ nên có một nhiệm vụ chính.

RULE C4:
Các window như 5, 20, 50 phải được khai báo bằng parameter,
không hard-code rải rác trong code.

RULE C5:
GARCH order phải là parameter:

p = 1
q = 1

RULE C6:
Phải có validation/check trước khi chạy GARCH.

RULE C7:
Không được silently ignore warning/error từ GARCH.

RULE C8:
Kết quả trung gian quan trọng phải có thể kiểm tra.

RULE C9:
Code phải tái chạy được trên dataset mới có cùng schema.

RULE C10:
Không tạo feature hoặc mô hình mà không thể giải thích
nguồn dữ liệu và công thức.


==================================================
VIII. NGUYÊN TẮC TỔNG QUÁT
--------------------------------------------------

RULE FINAL 1:
Data → Return → Feature Engineering → GARCH.

RULE FINAL 2:
Không đảo ngược temporal order.

RULE FINAL 3:
Không nhìn tương lai.

RULE FINAL 4:
Không fit mô hình bằng dữ liệu tương lai.

RULE FINAL 5:
Raw data và processed data phải tách biệt.

RULE FINAL 6:
Mọi feature phải có công thức rõ ràng.

RULE FINAL 7:
GARCH phải được xem là mô hình volatility,
không phải mô hình dự đoán giá VN-Index.

RULE FINAL 8:
Mục tiêu của phần này là tạo ra một dataset sạch,
không leakage, có feature hợp lý và có thông tin volatility
từ GARCH để module ML sử dụng.