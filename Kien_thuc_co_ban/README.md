# Kiến thức cơ bản của viễn thám
## 1.Khái niệm viễn thám
Viễn thám là phương pháp thu thập thông tin về các đối tượng, hiện tượng trên bề mặt và trong khí quyển Trái Đất thông qua các cảm biến đặt trên vệ tinh hoặc máy bay mà không cần tiếp xúc trực tiếp
Trong bài toán sinh báo cáo ô nhiễm không khí bằng LLM (AirReport-LLM) , viễn thám đóng vai trò cung cấp góc nhìn không gian diện rộng về các chất gây ô nhiễm, giúp bổ sung dữ liệu cho các khu vực xa xôi hoặc thiếu vắng các trạm đo mặt đất (WAQI/OpenAQ)
## 2.Vệ tinh quan sát Trái Đất (Sentinel-5P) 
Vệ tinh giám sát chất lượng không khí được trang bị các máy đo quang phổ chuyên dụng nhằm đo lường thành phần khí vi lượng. Dữ liệu từ vệ tinh viễn thám khí quyển được sử dụng để:
* Theo dõi phân bố mật độ chất ô nhiễm dạng khí (NO2,SO2,CO,O3)
* Đại diện gián tiếp cho nồng độ bụi mịn PM2.5/PM10
* Phát hiện các vệt khói bụi liên tỉnh hoặc ô nhiễm xuyên biên giới
* Làm nguồn dữ liệu kiểm chứng và bổ trợ không gian cho các trạm quan trắc mặt đất
## 3.Nguyên lý hoạt động cơ bản của hệ thống:
```
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
```
## 4.Các nguồn dữ liệu đầu vào
* Nồng độ các chất ô nhiễm mặt đất: PM2.5, PM10, NO2, SO2, CO, O3.
* Chỉ số chất lượng không khí AQI theo chuẩn trạm.
* Dữ liệu cột khí quyển từ vệ tinh đo đạc trên diện rộng.
* Yếu tố thời tiết, khí tượng ảnh hưởng trực tiếp đến sự tích tụ hay khuếch tán ô nhiễm.
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
```
Trạm quan trắc mặt đất (Chính phủ / Cảm biến)
                      ↓
           Đo đạc nồng độ chất ô nhiễm
                      ↓
             WAQI / OpenAQ v3 API
                      ↓
         Dữ liệu chuỗi thời gian ô nhiễm
                      ↓
               AirReport-LLM
```
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

## 7.Dữ liệu khí tượng EAR5
 là bộ dữ liệu tái phân tích khí hậu toàn cầu được cung cấp bởi trung tâm Dự báo Thời tiết Hạn vừa Châu Âu (ECMWF) thông qua Copernicus Climate Data Store (CDS).
 ERA5 kết hợp mô hình vật lý khí quyển với dữ liệu quan trắc toàn cầu để tái hiện các điều kiện khí tượng chi tiết theo từng giờ.
 Đối với AirReport-LLM, ERA5 là nguồn dữ liệu quan trọng để lấy các thông tin khí tượng (đặc biệt là gió, nhiệt độ, chiều cao lớp biên khí quyển) phục vụ giải thích cơ chế khuếch tán hoặc tích tụ ô nhiễm trong báo cáo.
Các trường dữ liệu quan trọng bao gồm:

            latitude
            longitude
            time
            u10 (thành phần gió đông - tây ở 10m)
            v10 (thành phần gió bắc - nam ở 10m)
            t2m (nhiệt độ không khí ở 2m)
            d2m (nhiệt độ điểm sương ở 2m)
            sp (áp suất bề mặt)
            blh (chiều cao lớp biên khí quyển - Boundary Layer Height)
            tp (tổng lượng mưa)

## 8. Fine-tuning LLM, LoRA/QLoRA
* Fine-tuning : Tinh chỉnh, huấn luyện chuyên sâu dựa trên nền tảng có sẵn
  ** SFT: tinh chỉnh có giám sát
  ** SSFT: tinh chỉnh tự giám sát
  ** RLHF: học tăng cường lấy phản hồi của con người
* LoRA (Low-Rank Adaptation): đóng băng mô hình gốc, tạo ra và huấn luyện vài ma trận trọng số mới gắn thêm vào mô hình gốc và học các điều chỉnh đầu ra của mô hình gốc để phù hợp với nhiệm vụ mới
* QLoRA (Quantized Low-Rank Adaption): Làm cho mô hình đủ nhỏ gọn trong bộ nhớ , đóng băng mô hình cũ và áp dụng LoRA - tách nhỏ các dữ liệu và nén lại tránh mất hoặc sai dữ liệu
 
## 9. Prompt Template 
* Là phương pháp thiết kế câu lệnh được sử dụng trong AirReport-LLM nhằm chuyển đổi các thông số đo đạc thành báo cáo cảnh báo ô nhiễm không khí bằng tiếng Việt.
* Prompt Template đóng vai trò làm cầu nối kiểm soát chặt chẽ giữa dữ liệu số định lượng (từ WAQI, Sentinel-5P, ERA5) và mô hình ngôn ngữ lớn.
* Đối với AirReport-LLM, kỹ thuật này là giải pháp cốt lõi để cố định số liệu thực tế, loại trừ triệt để hiện tượng bịa đặt số liệu (hallucination) và định hướng văn phong hành chính/chuyên môn theo mẫu chuẩn của CEM
## 10. Học máy và học sâu
### 10.1. Vị trí của Học máy và học sâu
Fine-tuning LLM thực chất là một nhánh chuyên sâu của **Học sâu (Deep Learning)** thuộc lĩnh vực Xử lý ngôn ngữ tự nhiên (NLP).
```text
       Dữ liệu thô (Trạm đo, Vệ tinh, Khí tượng)
                          ↓
      [HỌC MÁY TRUYỀN THỐNG / STATISTICAL ML]
      ├── Tiền xử lý, lọc nhiễu, điền dữ liệu khuyết (KNN Imputer / MICE)
      ├── Phân cụm mức độ ô nhiễm, phân loại cảnh báo (Random Forest / LightGBM)
      └── Trích xuất đặc trưng thống kê & Xu hướng nồng độ
                          ↓
            Bảng đặc trưng định lượng (Features)
                          ↓
             [HỌC SÂU / DEEP LEARNING & LLM]
      ├── Kiến trúc Transformer (Cơ chế Self-Attention đa đầu)
      ├── Tinh chỉnh mạng nơ-ron sâu qua Backpropagation (Lan truyền ngược)
      └── Fine-tuning thích ứng ma trận hạng thấp (LoRA/QLoRA)
                          ↓
      Văn bản Báo cáo Cảnh báo Môi trường hoàn chỉnh
```
### 10.2. Học máy (ML)
* Khôi phục chuỗi thời gian bị khuyết: Do trạm quan trắc mặt đất thường mất tín hiệu đột ngột, các giải thuật như KNN hoặc Random Forest Regressor được dùng để nội suy nồng độ PM2.5/NO2 dựa trên trạm lân cận và hướng gió ERA5.
* Phân lớp rủi ro tự động: Sử dụng các thuật toán phân loại (Decision Tree, SVM, XGBoost) để xác thực ngưỡng chỉ số AQI và phân loại cấp độ cảnh báo (An toàn, Nguy hại, Khẩn cấp) theo đúng quy chuẩn CEM.
* Đặc điểm: Tối ưu hóa trên dữ liệu dạng bảng, tốc độ tính toán mili-giây, độ chính xác số học tuyệt đối và giải thích được nguyên nhân theo ngưỡng logic cố định.
### 10.3. Học sâu (DL)
* Kiến trúc Transformer: Mô hình hoạt động dựa trên hàng chục lớp nơ-ron sâu với cơ chế tự chú ý (Self-Attention), giúp mô hình hiểu được mối liên hệ phức tạp giữa biến động thời tiết (gió lặng, lớp biên thấp) và hiện tượng nồng độ bụi gia tăng.
* Cơ chế tối ưu hóa lan truyền ngược (Backpropagation): Trong quá trình SFT (Supervised Fine-Tuning), mô hình tính toán đạo hàm hàm mất mát (Cross-Entropy Loss) để điều chỉnh các ma trận trọng số thích ứng.
* Học biểu diễn ngữ cảnh chuyên ngành: Học sâu giúp mô hình chuyển đổi các vectơ đặc trưng số (Embedding) thành câu văn nhận định tiếng Việt có ngữ pháp chuẩn xác, văn phong hành chính chỉn chu và giàu tính khoa học
## 11. Kết luận
* Dữ liệu quan trắc môi trường và mô hình ngôn ngữ lớn cung cấp khả năng tự động hóa việc phân tích và cảnh báo chất lượng không khí từ dữ liệu số
* Trong hệ thống AirReport-LLM, dữ liệu quan trắc mặt đất từ WAQI/OpenAQ v3 được sử dụng làm nguồn số liệu chính xác cốt lõi, kết hợp cùng dữ liệu viễn thám Sentinel-5P để bao quát ô nhiễm diện rộng và dữ liệu khí tượng ERA5 nhằm giải thích điều kiện khuếch tán khói bụi
* Sau khi tổng hợp và chuẩn hóa số liệu, hệ thống đưa dữ liệu qua Prompt Template để mô hình LLM đã tinh chỉnh (Fine-tuned qua LoRA/QLoRA) tự động tạo báo cáo tiếng Việt theo chuẩn văn bản môi trường 
```text
Cách dữ liệu quan trắc và khí tượng được thu thập, xử lý.
Ý nghĩa của các trường dữ liệu đo đạc (PM2.5, NO2, AQI, gió, BLH).
Ý nghĩa của việc kết hợp trạm mặt đất với viễn thám Sentinel-5P.
Nguyên lý fine-tuning LLM với kỹ thuật LoRA/QLoRA.
Cách thiết kế Prompt Template để loại trừ ảo giác số liệu.
Quy trình tự động sinh và xuất báo cáo cảnh báo hoàn chỉnh.
