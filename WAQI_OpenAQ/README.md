# 6.WAQI/OpenAQ v3
## 1.Tổng quan về OpenAQ v3 và WAQI
### 1.1.OpenAQ v3 
* OpenAQ là nền tảng mở quy mô toàn cầu chuyên tổng hợp dữ liệu chất lượng không khí theo thời gian thực và lịch sử từ các trạm quan trắc của chính phủ, viện nghiên cứu và mạng lưới cảm biến cộng đồng.
* OpenAQ API v3 cung cấp:
 ** Chuẩn hóa thực thể: Tách biệt rõ ràng giữa trạm đo ('locations'), cảm biến vật lý ('sensors'), thông số đo,bản ghi đo đạc
 ** Hỗ trợ dữ liệu lịch sử dài hạn (truy vấn theo ngày/tháng/năm)
 ** Quản lý phân quyền truy cập thông qua header 'X-API-Key'
#### 1.1.1.Mô hình dữ liệu quan hệ:
`Locations` ➔ `Sensors` ➔ `Parameters` ➔ `Measurements / Aggregations`
* Locations (Trạm quan trắc): Đại diện cho vị trí địa lý cố định hoặc di động (tọa độ coordinates, quốc gia country, độ cao, loại trạm đo của chính phủ hay cảm biến giá rẻ - low-cost sensor). Một Location có thể chứa nhiều Sensor.
* Sensors (Cảm biến vật lý): Thiết bị đo cụ thể gắn tại một location. Mỗi sensor thường chỉ đo đúng 1 tham số (parameter).
* Parameters (Thông số đo): Danh mục các chất ô nhiễm hoặc chỉ số khí tượng chuẩn hóa toàn cầu:Bụi & Khí: PM2.5, PM10 , O3 , NO2 , SO2 , CO. Thời tiết: Nhiệt độ, độ ẩm, áp suất, tốc độ gió.   Mỗi parameter có id, name, units (đã được quy chuẩn đồng nhất)
* Measurements & Aggregations (Chuỗi đo đạc): Dữ liệu chuỗi thời gian thực tế gắn liền với từng sensors_id.
#### 1.1.2.Luồng truy vấn dữ liệu chuẩn
```text
BƯỚC 1: Tìm Location
GET /v3/locations?coordinates=21.0285,105.8542&radius=10000
    │
    ▼
BƯỚC 2: Liệt kê Sensors thuộc Location
GET /v3/locations/{locations_id}/sensors
    │ (Xác định sensor_id đo PM2.5)
    ▼
BƯỚC 3: Kéo dữ liệu đo (Measurements/Aggregations)
GET /v3/sensors/{sensors_id}/measurements?date_from=2026-09-01&date_to=2026-09-20
```
#### 1.1.3. Các điểm cốt lõi của OpenAQ v3
### 📍 Địa điểm & Cảm biến
* `GET /v3/locations`: Tìm kiếm trạm đo theo quốc gia, tọa độ, nhà cung cấp (`providers_id`).
* `GET /v3/locations/{id}/latest`: Lấy giá trị đo mới nhất của trạm.
* `GET /v3/locations/{id}/sensors`: Xem toàn bộ danh sách cảm biến thuộc trạm đo.

### 📊 Dữ liệu đo đạc & Thống kê tính toán sẵn
* `GET /v3/sensors/{sensors_id}/measurements`: Lấy dữ liệu đo gốc (raw measurements).
* `GET /v3/sensors/{sensors_id}/hours`: Dữ liệu trung bình theo từng giờ.
* `GET /v3/sensors/{sensors_id}/days`: Dữ liệu trung bình theo từng ngày.
* `GET /v3/sensors/{sensors_id}/hours/dayofweek`: Thống kê theo giờ của các ngày trong tuần (T2 - CN).
* `GET /v3/sensors/{sensors_id}/hours/hourofday`: Phân tích thống kê theo từng khung giờ trong ngày.
### 1.2.WAQI/AQICN
WAQI/AQICN là dự án dữ liệu môi trường xã hội cung cấp chỉ số chất lượng không khí theo thời gian thực cho hơn 30.000 trạm trên toàn cầu.
* Điểm mạnh của WAQI: Đã chuẩn hóa sẵn giá trị tính toán chỉ số AQI tổng thể và AQI cho từng chất ô nhiễm riêng biệt theo trạm.
* Dữ liệu trả về qua REST API dưới dạng JSON nhanh chóng, hỗ trợ tìm kiếm trạm qua tọa độ địa lý, thành phố hoặc mã định danh `feed`.
#### 1.2.1. Cơ chế tính toán AQI của WAQI
* Quy chuẩn tính toán: WAQI áp dụng tiêu chuẩn chuẩn hóa US-EPA (United States Environmental Protection Agency) để chuyển đổi nồng độ các chất gây ô nhiễm (như $\mu g/m^3$, $ppm$) sang thang điểm không thứ nguyên (từ 0 đến 500+).
* Chỉ số AQI tổng thể của một trạm tại một thời điểm được quyết định bởi chất ô nhiễm có chỉ số AQI cao nhất:$$AQI_{overall} = \max(AQI_{PM2.5}, AQI_{PM10}, AQI_{O_3}, AQI_{NO_2}, AQI_{SO_2}, AQI_{CO})$$
#### 1.2.2.Các chỉ số thành phần
* Bụi mịn: pm25 ($PM_{2.5}$), pm10 ($PM_{10}$).Khí độc hại: o3 (Ozone), no2 (Nitrogen Dioxide), so2 (Sulfur Dioxide), co (Carbon Monoxide).
* Thông số khí tượng kèm theo: t (nhiệt độ), h (độ ẩm), p (áp suất khí quyển), w (tốc độ gió), wg (gió giật).
### 1.3.So sánh WAQI với OpenAQ v3
| Tiêu chí | WAQI (World Air Quality Index) | OpenAQ API v3 |
| :--- | :--- | :--- |
| **Mục đích cốt lõi** | Cung cấp chỉ số AQI tức thì, giao diện trực quan cho người dùng cuối và ứng dụng bản đồ. | Nền tảng khoa học dữ liệu, truy cập dữ liệu thô (*raw measurements*) có cấu trúc. |
| **Độ sâu dữ liệu** | Tập trung vào thời gian thực (*Real-time*), dữ liệu lịch sử hạn chế hoặc theo dạng biểu đồ tóm tắt. | Dữ liệu chuỗi thời gian (*time-series*) dài hạn, hỗ trợ truy vấn chi tiết từng timestamp. |
| **Xử lý AQI** | Đã tính toán và chuẩn hóa sẵn sang thang đo **US-EPA AQI**. | Trả về nồng độ vật lý gốc ($\mu g/m^3$, $ppm$); người dùng tự xây dựng công thức tính AQI nếu cần. |
| **Mức độ phức tạp API** | Đơn giản, dễ tích hợp vào frontend chỉ qua 1 request. | Cấu trúc quan hệ chặt chẽ (`locations` $\rightarrow$ `sensors` $\rightarrow$ `parameters` $\rightarrow$ `measurements`), đòi hỏi xử lý nhiều bước hơn. |
### 1.4.Mô hình luồng dữ liệu trạm quan trắc trong hệ thống LLM
```text
OpenAQ v3 API (Lịch sử chuỗi đo)  WAQI API (Real-time & AQI quy chuẩn)
               │                                      │
               └──────────────────┬───────────────────┘
                                  ↓
                  Thu thập & Kiểm tra tính hợp lệ
                                  ↓
                 Làm sạch, lọc trùng & Khử ngoại lai
                                  ↓
             Tính toán thống kê: Min, Max, Trung bình, AQI
                                  ↓
      Ghép nối với Vệ tinh Sentinel-5P & Khí tượng ERA5 theo tọa độ
                                  ↓
              Tạo cặp Input - Output cho Fine-tuning LLM
```
## 2.Các thông số quan trắc
### 2.1. Bụi mịn PM2.5
* PM2.5 là các hạt bụi lơ lửng trong khí quyển có đường kính khí động học nhỏ hơn hoặc bằng 2.5 $\mu m$. Do kích thước siêu vi, bụi PM2.5 có khả năng đi sâu vào phế nang phổi, thâm nhập vào hệ tuần hoàn máu và là tác nhân chính gây suy giảm tầm nhìn đô thị.
* PM2.5 thường là chất ô nhiễm chính (chất quyết định chỉ số AQI) tại các đô thị Việt Nam trong mùa đông
* LLM dựa vào nồng độ trung bình 24h của PM2.5 so với Quy chuẩn Kỹ thuật Quốc gia (QCVN 05:2023/BTNMT) để sinh các câu cảnh báo mức độ rủi ro sức khỏe cho người già, trẻ em và cộng đồng
### 2.2. Các thông số quan trắc khác
```text
Trạm quan trắc (Location)
├── Sensor PM2.5 (Bụi mịn - µg/m³)
├── Sensor PM10  (Bụi hô hấp - µg/m³)
├── Sensor NO2   (Khí thải giao thông - ppb hoặc µg/m³)
├── Sensor SO2   (Khí thải nhiệt điện, luyện kim - ppb hoặc µg/m³)
├── Sensor CO    (Đốt cháy không hoàn toàn - ppm hoặc mg/m³)
└── Sensor O3    (Ozone mặt đất - ppb hoặc µg/m³)
```
## 3.Thu thập dữ liệu
### 3.1.Thu thập dữ liệu từ openAQ v3
```python
import pandas as pd
import requests

OPENAQ_API_KEY = "YOUR_OPENAQ_API_KEY"
HEADERS = {"X-API-Key": OPENAQ_API_KEY}

# 1. Tìm các trạm quan trắc tại Việt Nam (Country ID = 56)
locations_url = "[https://api.openaq.org/v3/locations](https://api.openaq.org/v3/locations)"
params = {"countries_id": 56, "limit": 10}

res = requests.get(locations_url, headers=HEADERS, params=params, timeout=30)
locations = res.json().get("results", [])

# 2. Tìm sensor đo PM2.5 của trạm đầu tiên
target_sensor_id = None
if locations:
    first_loc = locations[0]
    loc_id = first_loc["id"]
    sensors_url = f"[https://api.openaq.org/v3/locations/](https://api.openaq.org/v3/locations/){loc_id}/sensors"
    s_res = requests.get(sensors_url, headers=HEADERS, timeout=30)

    for sensor in s_res.json().get("results", []):
        if sensor.get("parameter", {}).get("name") == "pm25":
            target_sensor_id = sensor["id"]
            break

# 3. Lấy chuỗi đo đạc theo giờ từ sensor PM2.5
if target_sensor_id:
    meas_url = f"[https://api.openaq.org/v3/sensors/](https://api.openaq.org/v3/sensors/){target_sensor_id}/measurements"
    m_params = {"limit": 100}
    m_res = requests.get(meas_url, headers=HEADERS, params=m_params, timeout=30)

    records = []
    for item in m_res.json().get("results", []):
        records.append(
            {
                "datetime": item.get("period", {})
                .get("datetimeFrom", {})
                .get("local"),
                "parameter": "pm25",
                "value": item.get("value"),
                "unit": item.get("unit"),
            }
        )

    df_openaq = pd.DataFrame(records)
    print(df_openaq.head())
```
### 3.2.Thu thập dữ liệu từ WAQI API
```python
import requests

WAQI_TOKEN = "YOUR_WAQI_API_TOKEN"

# Truy vấn trạm theo tọa độ Hà Nội (Lat: 21.0285, Lon: 105.8542)
feed_url = f"[https://api.waqi.info/feed/geo:21.0285;105.8542/?token=](https://api.waqi.info/feed/geo:21.0285;105.8542/?token=){WAQI_TOKEN}"
res = requests.get(feed_url, timeout=30)
data = res.json()

if data.get("status") == "ok":
    payload = data["data"]
    overall_aqi = payload.get("aqi")
    iaqi = payload.get("iaqi", {})

    station_summary = {
        "station_name": payload.get("city", {}).get("name"),
        "time": payload.get("time", {}).get("s"),
        "overall_aqi": overall_aqi,
        "pm25": iaqi.get("pm25", {}).get("v"),
        "pm10": iaqi.get("pm10", {}).get("v"),
        "no2": iaqi.get("no2", {}).get("v"),
        "temp": iaqi.get("t", {}).get("v"),
        "humidity": iaqi.get("h", {}).get("v"),
        "wind": iaqi.get("w", {}).get("v"),
    }
    print("Dữ liệu trích xuất từ WAQI:")
    print(station_summary)
```
## 4.Loại bỏ dữ liệu khuyết thiếu và trùng lặp
```python
# Loại bỏ giá trị null
df = df.dropna(subset=["value", "datetime"])

# Lọc các giá trị âm bất thường của cảm biến (sensor drift)
df = df[df["value"] >= 0]

# Loại bỏ các bản ghi trùng lặp thời gian đo
df = df.drop_duplicates(subset=["datetime"])

# Chuyển đổi định dạng thời gian
df["datetime"] = pd.to_datetime(df["datetime"])
df = df.sort_values("datetime").reset_index(drop=True)
```
## 5.Các công thức toán học
### 5.1.Giá trị trung bình 24h
* Đánh giá tác động sức khỏe của bụi PM2.5 theo quy chuẩn QCVN 05:2023/BTNMT dựa trên giá trị trung bình 24 giờ liên tục:
$$\overline{PM_{2.5}} = \frac{1}{n}\sum_{i=1}^{n} PM_{2.5, i}$$
* Trong đó $n$ phải đạt tối thiểu 75% số giờ hợp lệ trong ngày ($n \ge 18$ giá trị).
### 5.2.Chỉ số AQI cho từng chất 
* Chỉ số chất lượng không khí của một thông số ô nhiễm được tính theo công thức nội suy tuyến tính từng đoạn:
* $$I_p = \frac{I_{high} - I_{low}}{C_{high} - C_{low}} \times (C_p - C_{low}) + I_{low}$$
* $C_p$: Nồng độ quan trắc thực tế của chất $p$.
* $C_{low}, C_{high}$: Nồng độ điểm cắt dưới và điểm cắt trên ứng với dải giá trị của $C_p$.
* $I_{low}, I_{high}$: Chỉ số AQI tương ứng với các mức cắt $C_{low}, C_{high}$.
## 6.Kết luận
Dữ liệu quan trắc mặt đất từ OpenAQ v3 và WAQI là nền tảng cốt lõi của hệ thống.Việc thu thập đúng sensor, làm sạch dữ liệu chặt chẽ và tóm tắt thống kê chính xác là điều kiện tiên quyết giúp mô hình ngôn ngữ lớn sinh ra các bản báo cáo chất lượng không khí chuẩn mực, loại bỏ triệt để hiện tượng ảo giác thông tin và đáp ứng đúng quy chuẩn môi trường
