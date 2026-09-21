# Kiến thức cơ bản của viễn thám
## 1.Khái niệm viễn thám
Viễn thám là phương pháp thu thập thông tin về các đối tượng, hiện tượng trên bề mặt và trong khí quyển Trái Đất thông qua các cảm biến đặt trên vệ tinh hoặc máy bay mà không cần tiếp xúc trực tiếp
Trong bài toán sinh báo cáo ô nhiễm không khí bằng LLM, viễn thám đóng vai trò cung cấp góc nhìn không gian diện rộng về các chất gây ô nhiễm, giúp bổ sung dữ liệu cho các khu vực xa xôi hoặc thiếu vắng các trạm đo mặt đất (WAQI/OpenAQ)
## 2.Vệ tinh quan sát Trái Đất (Sentinel-5P) 
Vệ tinh giám sát chất lượng không khí được trang bị các máy đo quang phổ chuyên dụng nhằm đo lường thành phần khí vi lượng. Dữ liệu từ vệ tinh viễn thám khí quyển được sử dụng để:
* Theo dõi phân bố mật độ chất ô nhiễm dạng khí (NO2,SO2,CO,O3)
* Đại diện gián tiếp cho nồng độ bụi mịn PM2.5/PM10
* Phát hiện các vệt khói bụi liên tỉnh hoặc ô nhiễm xuyên biên giới
* Làm nguồn dữ liệu kiểm chứng và bổ trợ không gian cho các trạm quan trắc mặt đất
## 3.Nguyên lý hoạt động cơ bản của hệ thống:
'''
Trạm quan trắc (WAQI/OpenAQ) + Vệ tinh (Sentinel-5P) + Khí tượng (ERA5)
                               ↓
                   Làm sạch & Tiền xử lý số liệu
                               ↓
         Bộ trích xuất thông số & Phân tích xu hướng (Stats Engine)
                               ↓
              Khung mẫu bán cấu trúc (Prompt/Template)
                               ↓
              Mô hình LLM (Fine-tuned qua LoRA/QLoRA)
                               ↓
                  Báo cáo Tiếng Việt hoàn chỉnh
                               ↓
                   Hệ thống Web & Xuất file PDF
## 4.Các nguồn dữ liệu đầu vào
Nồng độ các chất ô nhiễm mặt đất: PM2.5, PM10, NO2, SO2, CO, O3.
Chỉ số chất lượng không khí AQI theo chuẩn trạm.
Dữ liệu cột khí quyển từ vệ tinh đo đạc trên diện rộng.
Yếu tố thời tiết, khí tượng ảnh hưởng trực tiếp đến sự tích tụ hay khuếch tán ô nhiễm.
## 5.Các chất gây ô nhiễm không khí chính
| Tên chất ô nhiễm | Ký hiệu | Đơn vị đo | Nguồn phát sinh chính | Tác động sức khỏe / Môi trường |
| :--- | :--- | :--- | :--- | :--- |
| **Bụi mịn PM2.5** | PM2.5 | µg/m³ | Khí thải giao thông, đốt rơm rạ, công nghiệp | Đi sâu vào phế nang, ảnh hưởng tim mạch và hô hấp |
| **Bụi thô PM10** | PM10 | µg/m³ | Bụi đường xá, xây dựng, gió cát | Kích ứng mắt, mũi, họng và hệ hô hấp |
| **Nitơ Dioxit** | NO₂ | ppb / µg/m³ | Động cơ đốt trong, nhà máy nhiệt điện | Gây viêm đường hô hấp, tạo mưa axit |
| **Lưu huỳnh Dioxit** | SO₂ | ppb / µg/m³ | Đốt than đá, dầu mỏ, sản xuất công nghiệp | Co thắt phế quản, khó thở, gây mưa axit |
| **Khí Ozone mặt đất** | O₃ | ppb / µg/m³ | Phản ứng quang hóa giữa NOx và VOC dưới ánh nắng | Gây khó thở, tổn thương mô phổi, suy giảm thị lực |
| **Khí Carbon Monoxide** | CO | ppm / mg/m³ | Quá trình cháy không hoàn toàn của nhiên liệu | Giảm khả năng vận chuyển oxy của máu, đau đầu, chóng mặt |
## 6.WAQI/OpenAQ
WAQI (World Air Quality Index) và OpenAQ v3 là các nền tảng mở chuyên tổng hợp và cung cấp dữ liệu chất lượng không khí từ các trạm quan trắc mặt đất trên toàn cầu. Hệ thống tích hợp dữ liệu nồng độ các chất ô nhiễm và chỉ số AQI từ nhiều nguồn trạm quan trắc chính phủ, tổ chức nghiên cứu và mạng lưới cảm biến. Đối với AirReport-LLM, WAQI/OpenAQ v3 là nguồn dữ liệu quan trọng để lấy các thông tin số liệu đo đạc thực tế tại mặt đất.

Quy trình cơ bản:
Trạm quan trắc mặt đất (Chính phủ / Cảm biến)
                      ↓
           Đo đạc nồng độ chất ô nhiễm
                      ↓
             WAQI / OpenAQ v3 API
                      ↓
         Dữ liệu chuỗi thời gian ô nhiễm
                      ↓
               AirReport-LLM

Các trường dữ liệu có thể bao gồm 
            station_id
            location_name
            latitude
            longitude
            datetime_utc
            datetime_local
            parameter (pm25, pm10, no2, so2, co, o3)
            value
            unit
            aqi

