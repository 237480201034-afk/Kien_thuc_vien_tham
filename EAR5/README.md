# 5. EAR5
ERA5 là bộ dữ liệu tái phân tích khí quyển toàn cầu do Trung tâm Dự báo Thời tiết Hạn vừa Châu Âu (ECMWF) phát triển, được sử dụng trong hệ thống **AirReport-LLM** nhằm bổ sung thông tin thời tiết phục vụ việc giải thích cơ chế tích tụ hoặc khuếch tán ô nhiễm.
## 1. ERA5 và dữ liệu khí tượng tái phân tích

### 1.1. ERA5 là gì?
ERA5 là bộ dữ liệu tái phân tích khí quyển thế hệ thứ 5 của ECMWF, kết hợp các mô hình vật lý khí quyển tiên tiến với dữ liệu quan trắc thực tế toàn cầu (vệ tinh, bóng thám không, trạm đo mặt đất)[cite: 2]. 

Dữ liệu ERA5 có độ phân giải không gian khoảng $0.25^\circ \times 0.25^\circ$ (khoảng $31\text{ km}$), cung cấp dữ liệu theo từng giờ trên toàn cầu từ năm 1940 đến nay.

### 1.2. dữ liệu ERA5 để làm gì
Ô nhiễm không khí không chỉ do lượng phát thải quyết định mà chịu ảnh hưởng mạnh mẽ bởi điều kiện thời tiết:
* **Gió lặng:** Khói bụi từ giao thông, nhà máy không được đối lưu phân tán, dẫn đến nồng độ bụi tại chỗ tăng cao[cite: 2].
* **Hiện tượng nghịch nhiệt tầng thấp (Inversion):** Chiều cao lớp biên khí quyển ($blh$) hạ thấp khiến chất ô nhiễm bị giữ lại sát mặt đất.
* **Mưa:** Có tác dụng rửa trôi hạt sol khí và làm sạch bầu khí quyển.

```text
Dữ liệu đo đạc (PM2.5 tăng đột biến) + ERA5 (Gió lặng 0.5 m/s, BLH cực thấp 150m)
                                     ↓
          Quy tắc phân tích: Xác định điều kiện nghịch nhiệt tích tụ bụi
                                     ↓
                  Đưa vào Context Prompt của LLM
                                     ↓
 LLM sinh văn bản: "Nồng độ bụi PM2.5 tăng cao chủ yếu do hiện tượng nghịch nhiệt
                   và lặng gió khiến các chất ô nhiễm không thể khuếch tán."
```
## 2. Các biến khí tượng quan trọng trong hệ thống
### 2.1 Thành phần gió (u10 và v10)
* $u10$ (U-component of wind): Thành phần gió theo hướng Đông - Tây (chiều dương hướng về phía Đông).
* $v10$ (V-component of wind): Thành phần gió theo hướng Nam - Bắc (chiều dương hướng về phía Bắc).
* Từ hai thành phần này, hệ thống tính toán ra tốc độ gió và hướng gió thực tế để đánh giá nguồn phát thải lan truyền từ đâu tới.
### 2.2 Chiều cao lớp khí quyển ($blh$ - Boundary Layer Height)
* Thể hiện độ dày của tầng đối lưu sát mặt đất, nơi diễn ra hầu hết các hoạt động trao đổi nhiệt và phát tán ô nhiễm.
* Vào ban ngày trời nắng: $blh$ có thể đạt $1000 - 2000\text{ m}$ (thông thoáng, bụi phát tán lên cao).
* Vào ban đêm mùa đông: $blh$ có thể giảm xuống dưới $200\text{ m}$ (nghịch nhiệt bức xạ, giam hãm bụi mịn ở tầng thở của con người).
## 3. Thu thập và xử lý dữ liệu 
```python
import xarray as xr
import pandas as pd
import numpy as np

# Đọc file NetCDF
ds = xr.open_dataset('era5_meteorology.nc')

# Giả sử cần trích xuất tại tọa độ trạm Hà Nội (Lat: 21.0285, Lon: 105.8542)
station_data = ds.sel(latitude=21.0285, longitude=105.8542, method='nearest')

# Chuyển thành DataFrame
df_met = station_data.to_dataframe().reset_index()

# Chuyển đổi nhiệt độ từ Kelvin sang độ C
df_met['temp_c'] = df_met['t2m'] - 273.15
```
## 4. Các công thức tính toán khí tượng phục vụ Báo cáo
### 4.1. Tính tốc độ gió ($Wind\ Speed$)
* Tốc độ gió được tính bằng độ lớn của véctơ vận tốc tổng hợp:
$$Wind\ Speed = \sqrt{u10^2 + v10^2}$$
### 4.2. Tính hướng gió ($Wind\ Direction$)
* Hướng gió trong khí tượng được quy ước là góc mà gió thổi tới (tính theo độ $0^\circ - 360^\circ$ theo chiều kim đồng hồ, $0^\circ$ là hướng Bắc):
* $$Wind\ Direction = \left( 270 - \arctan2(v10, u10) \times \frac{180}{\pi} \right) \pmod{360}$$
