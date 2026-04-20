# Lab 2 Analysis Report (IMDB)

## 1. Data audit

- **Số lượng mẫu đã dùng:** 50,000  
- **Phân bố nhãn positive / negative:** 25,000 positive – 25,000 negative (cân bằng hoàn toàn)  
- **Độ dài review điển hình (median, p95):**  
  - Trước preprocessing: median = 173 từ, p95 = 590 từ  
  - Sau preprocessing: median = 207 từ, p95 = 705 từ  
- **Có missing / empty text không?** Không (0 trường hợp)  
- **Có duplicate không?** Có (~1.65%)  

### 3 quan sát đáng chú ý về dữ liệu IMDB

1. Dataset sạch và cân bằng → giúp mô hình học ổn định, tránh bias.  
2. Review rất dài và phân bố không đồng đều → dễ xuất hiện mixed sentiment.  
3. Có duplicate (~1.65%) → có thể ảnh hưởng đến đánh giá mô hình.  

---

## 2. Preprocessing design

- **Các bước làm sạch đã dùng:**
  - Xóa HTML tags (`<br />`)
  - Chuẩn hóa khoảng trắng
  - Thay URL/email bằng token
  - Lowercase toàn bộ text
  - Giữ dấu câu mặc định

- **Bạn giữ lại dấu câu hay bỏ đi? Vì sao?**  
  Giữ lại dấu câu. Thực nghiệm cho thấy khi bỏ dấu câu (`--drop_punct`), hiệu năng giảm nhẹ (0.9068 → 0.9064). Điều này cho thấy dấu câu mang thông tin cảm xúc quan trọng.

- **Bạn có thay số bằng `<NUM>` không? Vì sao?**  
  Không. Khi bật `--replace_number`, hiệu năng giảm nhẹ. Các số như “10/10”, “1/10” mang ý nghĩa sentiment rõ ràng nên không nên thay thế.

- **Có bước nào bạn cố tình không làm?**  
  Không xử lý mạnh như bỏ dấu câu hoặc thay số toàn bộ, vì có thể làm mất tín hiệu cảm xúc.

---

## 3. Experiment comparison

| Run | Text version | Vectorizer | Model | ngram | Macro-F1 | Accuracy | Ghi chú |
|-----|-------------|------------|-------|-------|----------|----------|--------|
| 1 | cleaned | TF-IDF | LogReg | 1-2 | 0.9068 | 0.9068 | Baseline |
| 2 | cleaned | TF-IDF | LogReg + replace_number | 1-2 | 0.9064 | 0.9064 | Giảm nhẹ |
| 3 | cleaned | TF-IDF | Linear SVM | 1-2 | **0.9098** | **0.9098** | Tốt nhất |

---

## 4. Error analysis (>= 10 lỗi)

### Bảng ghi lỗi

| ID | True | Pred | Nhóm lỗi | Giải thích ngắn |
|----|------|------|----------|----------------|
| 17596 | positive | negative | Phủ định | "not so well written" |
| 31245 | negative | positive | Phủ định | "never really works" |
| 8347 | negative | positive | Mixed sentiment | "excellent cast but unremarkable" |
| 22418 | negative | positive | Mixed sentiment | "great idea but poorly executed" |
| 37061 | negative | positive | Sarcasm | "pure genius" |
| 29148 | negative | positive | Sarcasm | "swimming against the tide" |
| 36707 | negative | positive | Keyword misleading | "greatest movies" |
| 19852 | negative | positive | Keyword misleading | "amazing visuals but boring story" |
| 34543 | negative | positive | Review dài | nhiều đoạn, nhiều ý |
| 15203 | negative | positive | Mixed sentiment | "starts good but becomes terrible" |

---

### Nhóm lỗi chính

#### 1. Phủ định
- Ví dụ: "not good", "never works"  
- Nguyên nhân: TF-IDF không hiểu cấu trúc phủ định  
- Cải thiện: xử lý negation hoặc dùng model hiểu context  

#### 2. Mixed sentiment
- Ví dụ: "excellent but unremarkable"  
- Nguyên nhân: nhiều tín hiệu trái chiều  
- Cải thiện: model sequence (BERT)  

#### 3. Keyword misleading
- Ví dụ: "greatest", "amazing" nhưng thực chất tiêu cực  
- Nguyên nhân: phụ thuộc từ khóa  
- Cải thiện: contextual embedding  

#### 4. Sarcasm
- Ví dụ: "pure genius"  
- Nguyên nhân: nghĩa mỉa mai  
- Cải thiện: deep learning  

#### 5. Review dài
- Nguyên nhân: nhiều đoạn, đổi giọng  
- Cải thiện: sequence model  

---

## 5. Reflection

- **Pipeline nào tốt nhất? Vì sao?**  
  TF-IDF + Linear SVM là tốt nhất (Macro-F1 = 0.9098), do xử lý tốt dữ liệu sparse và tối ưu margin hiệu quả.

- **Accuracy và Macro-F1 có chênh nhau nhiều không?**  
  Không đáng kể vì dataset cân bằng.

- **Nếu dữ liệu lệch lớp, metric nào quan trọng hơn? Vì sao?**  
  Macro-F1 quan trọng hơn vì phản ánh công bằng giữa các lớp, không bị bias bởi lớp lớn.

- **Một cải tiến bạn muốn thử ở Lab 3 là gì?**  
  Sử dụng mô hình hiểu ngữ cảnh (BERT/Transformer) để xử lý phủ định, sarcasm và mixed sentiment.
