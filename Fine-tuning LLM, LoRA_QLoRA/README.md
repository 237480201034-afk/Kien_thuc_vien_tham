# 8.Fine-tuning LLM, LoRA_QLoRA
## 1.Tổng quan về Fine-tuning (Tinh chỉnh mô hình)
### 1.1. Khái niệm
fine-tuning (Tinh chỉnh) là quá trình lấy một mô hình ngôn ngữ lớn đã được huấn luyện trước và tiếp tục huấn luyện chuyên sâu trên một tập dữ liệu chuyên ngành cụ thể
* **Bản chất:** Giúp mô hình thích nghi với cấu trúc câu, từ vựng kỹ thuật và định dạng mong muốn mà không phải huấn luyện lại từ đầu (Pre-training).
* **Đặc điểm:** Tối ưu chi phí phần cứng, tốc độ học (learning rate) thường thiết lập nhỏ hơn rất nhiều so với pre-training ($10^{-4}$ đến $10^{-5}$) nhằm bảo tồn kiến thức tổng quát và đòi hỏi cao về tính chính xác của dữ liệu đầu vào.
## 2.Các phương pháp fine-tuning 
### 2.1. SFT (Supervised Fine-tuning): Tinh chỉnh có giám sát
* **Nguyên lý:** Chuẩn bị tập dữ liệu gồm các cặp đầu vào (`prompt/input`) và đầu ra mong muốn (`target/output`). Mô hình được tối ưu hóa để dự đoán chính xác chuỗi từ ở đầu ra tương ứng.
* **Ứng dụng:** Đây là phương pháp cốt lõi để dạy LLM kỹ năng sinh báo cáo chất lượng không khí bằng tiếng Việt từ dữ liệu bảng đo đạc quan trắc.
### 2.2. SSFT ( Self-Supervised Fine-Tuning): Tinh chỉnh tự giám sát
* Huấn luyện mô hình học từ dữ liệu thô không gán nhãn bằng cơ chế dự đoán từ tiếp theo
### 2.3. RLHF (Reinforcement Learning from Human Feedback): Học tăng cường từ phản hồi của con người
* Mô hình SFT sinh ra nhiều văn bản báo cáo khác nhau cho cùng một chuỗi dữ liệu.
* Chuyên gia môi trường xếp hạng các câu trả lời từ chuẩn xác nhất đến tệ nhất.
* Dữ liệu xếp hạng được dùng để huấn luyện một Mô hình phần thưởng (Reward Model) để chấm điểm và định hướng LLM sinh văn bản tự nhiên, chính xác hơn.
## 2.4. PEFT (Parameter-Efficient Fine-tuning): Tinh chỉnh hiệu quả về tham số
* là một thư viện Python giúp tinh chỉnh các mô hình ngôn ngữ lớn(LLM) và mô hình Transformer một cách tiết kiệm tài nguyên bằng cách chỉ huấn luyện một phần nhỏ trọng số của mô hình thay vì toàn bộ
* Bất kỳ phương pháp nào của fine-tuning mô hình mà không cần cập nhật toàn bộ tham số đều gọi là PEFT
* LoRA là một thuật toán cụ thể nằm trong PEFT
## 3.Quy trình thực hiện Fine-tuning trong dự án
```text
  1. Xác định mục tiêu Fine-tuning
   └── Huấn luyện LLM đọc hiểu dữ liệu bảng trạm quan trắc để sinh báo cáo tiếng Việt
          ↓
2. Chuẩn bị dữ liệu huấn luyện
   ├── Dữ liệu đầu vào: WAQI/AQICN (lịch sử), OpenAQ V3, Sentinel-5P, ERA5
   └── Dữ liệu đầu ra mục tiêu: Tập dữ liệu báo cáo mẫu tiếng Việt (tham chiếu báo cáo CEM)
          ↓
3. Thiết lập kỹ thuật Fine-tuning
   ├── Sử dụng Hugging Face PEFT
   └── Áp dụng LoRA Fine-tuning / QLoRA
          ↓
4. Kết hợp kỹ thuật tối ưu
   └── Phối hợp cùng Prompt Engineering và cơ chế Template + LLM hybrid
          ↓
5. Đánh giá chất lượng mô hình sau Fine-tuning
   ├── Đánh giá tự động bằng chỉ số: BLEU, ROUGE
   └── Đánh giá chất lượng nội dung: Chuyên gia thẩm định
```
## 4. LoRA (Low-Rank Adaptation) : Kỹ thuật thích ứng hạng thấp
- Đóng băng mô hình gốc, tạo ra và huấn luyện vài ma trận trọng số mới gắn thêm vào mô hình gốc và học cách điều chỉnh đầu ra của mô hình gốc để phù hợp với nhiệm vụ mới
- Dùng phân rã ma trận để biểu diễn ΔW - lượng công cơ học nhỏ truyền qua ranh giới của 1 hệ bằng tích
- Rank: kích thước chiều chung của hai ma trận, rank càng cao các ma trận nhỏ sẽ lớn hơn (8,16..)của 2 ma trận ( thay vì huấn luyện ma trận lớn thì huấn luyện 2 ma trận nhỏ)
### 4.1. Nguyên lý phân rã ma trận hạng thấp
* Trong quá trình tinh chỉnh thông thường, trọng số mô hình $W_0 \in \mathbb{R}^{d \times k}$ được cập nhật thành $W = W_0 + \Delta W$, đòi hỏi phải lưu trữ và tính toán đạo hàm cho toàn bộ ma trận khổng lồ $\Delta W$.
* Kỹ thuật LoRA đóng băng hoàn toàn ma trận trọng số gốc $W_0$ và phân rã ma trận biến thiên trọng số $\Delta W$ thành tích của hai ma trận hạng thấp
* $$\Delta W = B \times A$$
* Trong đó:
  **$W_0 \in \mathbb{R}^{d \times k}$** (được đóng băng hoàn toàn).
  **$A \in \mathbb{R}^{r \times k}$** (khởi tạo theo phân phối chuẩn Gaussian).$B \in \mathbb{R}^{d   **\times r}$ (khởi tạo ban đầu bằng 0).Rank ($r$)**: Kích thước chiều chung của hai ma trận ($r \ll \min(d, k)$).
```text
Đầu vào x
              ┌────┴────┐
              │         │
              │     ┌───▼───┐
              │     │   A   │ (r × k) - Ma trận nén
              │     └───┬───┘
              │         │ r
       ┌──────▼──────┐  │
       │  W₀ (Gốc)   │ ┌───▼───┐
       │ (Đóng băng) │ │   B   │ (d × r) - Ma trận giải nén
       └──────┬──────┘ └───┬───┘
              │         │
              └────┬────┘
                   ▼
               h = W₀x + (B × A)x
  ```
### 4.2.Các tham số cốt lõi của LoRA
* Rank ($r$): Hạng của ma trận. Với tác vụ chuyển đổi dữ liệu số sang báo cáo, thường chọn $r = 8$ hoặc $r = 16$ để cân bằng hiệu năng và bộ nhớ.
* LoRA Alpha ($\alpha$): Hệ số co giãn khi cộng trọng số vào mô hình gốc: $\Delta W \times \frac{\alpha}{r}$. Thường đặt $\alpha = 2 \times r$.
* LoRA Dropout: Tỷ lệ dropout áp dụng cho các layer LoRA để chống overfitting (thường từ $0.05$ đến $0.1$).
* Target Modules: Các khối ma trận chú ý (Attention) được gắn LoRA, thường là q_proj, v_proj, k_proj, o_proj.
## 5.QLoRA (Quantized Low-Rank Adaptation): Thích ứng hạng thấp được lượng tử hóa
### QLoRA - Cơ chế hoạt động
QLoRA nâng cấp từ LoRA nhằm đưa khả năng fine-tuning các mô hình lớn (7B, 13B) vào các dòng GPU cá nhân hoặc Google Colab cấu hình hạn chế:
* Lượng tử hóa 4-bit NormalFloat (NF4): Nén trọng số mô hình gốc $W_0$ từ 16-bit xuống 4-bit mà hầu như không làm suy giảm chất lượng biểu diễn thông tin
* Lượng tử hóa kép (Double Quantization - DQ): Lượng tử hóa tiếp cả các hằng số lượng tử hóa, tiết kiệm thêm bộ nhớ VRAM
* Phân trang bộ nhớ (Paged Optimizers): Sử dụng cơ chế phân trang qua bộ nhớ CPU khi GPU bị quá tải đột ngột (tránh lỗi CUDA Out of Memory)
* Áp dụng LoRA trên nền mô hình đã được nén 4-bit
 ```text
Mô hình gốc (16-bit)  ──[Nén 4-bit NF4]──> Giảm 75% VRAM (Đóng băng)
                                                    │
                                         Gắn Adapter LoRA (16-bit)
                                                    │
                                         Chỉ huấn luyện AdapterMô hình gốc (16-bit)  ──[Nén 4-bit NF4]──> Giảm 75% VRAM (Đóng băng)
                                                    │
                                         Gắn Adapter LoRA (16-bit)
                                                    │
                                         Chỉ huấn luyện Adapter

  ```

