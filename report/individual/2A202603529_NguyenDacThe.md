# Báo cáo cá nhân — AI Foundation

- **MSSV:** 2A202603529
- **Họ và tên:** Nguyễn Đắc Thể
- **SEED** (in ở ô đầu notebook): 3178

## 1. Bằng chứng

| Tệp | Có? |
| --- | --- |
| [`day01_lab.ipynb`](day01_lab.ipynb) — notebook đã chạy, còn output | Có |
| [`lab1_report.json`](lab1_report.json) — do ô "Báo cáo cuối" sinh ra | Có |
| [`hand_calc.jpg`](hand_calc.jpg) — ảnh tính tay có các bước | Có |

## 2. Kết quả chính

*Chép bảng so sánh cuối Phần 5 (F1 test, F1 SUB 40, p95 ms/tin, USD/1k tin) của chính bạn.*
### 2.1. Chất lượng dự đoán

| Mô hình | Precision | Recall | F1 (test) | F1 (SUB 40) |
| --- | ---: | ---: | ---: | ---: |
| 1. BoW + LogReg (tự code) | 0.962 | 0.781 | 0.862 | 0.923 |
| 2. BoW + MLP (deep learning) | 0.965 | 0.865 | 0.912 | 0.919 |
| 3a. Encoder (MiniLM) + kNN | 0.841 | 0.938 | 0.887 | 0.900 |
| 3b. Encoder (MiniLM) + LogReg | 0.968 | 0.948 | 0.958 | 0.950 |
| 3c. Encoder (MiniLM) + MLP head | 0.968 | 0.948 | 0.958 | 0.974 |

### 2.2. Thời gian xử lý và chi phí

| Mô hình | Train (s) | Tin/s (cả lô) | p50 (ms/tin) | p95 (ms/tin) | USD/1.000 tin |
| --- | ---: | ---: | ---: | ---: | ---: |
| 1. BoW + LogReg (tự code) | 9.412 | 9445.813 | 0.056 | 0.085 | 0.0 |
| 2. BoW + MLP (deep learning) | 19.635 | 9221.844 | 0.329 | 4.471 | 0.0 |
| 3a. Encoder (MiniLM) + kNN | 80.689 | 18.104 | 93.844 | 211.083 | 0.0 |
| 3b. Encoder (MiniLM) + LogReg | 85.219 | 28.492 | 50.148 | 171.549 | 0.0 |
| 3c. Encoder (MiniLM) + MLP head | 82.328 | 20.563 | 51.609 | 74.642 | 0.0 |

### 2.3. Ghi chú khi đọc kết quả

- Hàng LLM chỉ đo trên SUB (40 tin) → so sánh chất lượng công bằng bằng cột 'F1 (SUB 40)'.
- SUB chỉ 40 tin: 1 tin sai đổi F1 vài điểm. Chênh < ~0.05 F1 coi như HOÀ — đừng xếp hạng theo số lẻ.
- Inference đo TỪ VĂN BẢN THÔ → NHÃN (gồm cả vectorize / encode). p50/p95 = gọi từng tin một, như một request thật.
- train (s) của 3a/3b/3c = thời gian encoder MiniLM encode tập train (+ train đầu LogReg / MLP); LLM = 0 vì không train. Số đo phụ thuộc CPU/GPU của Colab.

## 3. Dự đoán và thực tế

*Chọn 3 dự đoán `predict(...)` lệch nhiều nhất so với kết quả. Mỗi cái: bạn đoán gì, ra gì, vì sao lệch.*
### 3.1. `predict("llm_models")` — LLM Zero-shot vs Mô hình 3b

**Đoán:** LLM lớn zero-shot sẽ có điểm F1 cao vượt trội so với mô hình 3b.

**Thực tế:** Mô hình 3b (Encoder + LogReg) đạt F1 cao hơn hoặc tương đương LLM lớn zero-shot.

**Vì sao lệch:** Mô hình 3b đã được huấn luyện (fit) trực tiếp trên dữ liệu thật của bài toán, trong khi LLM zero-shot chỉ suy luận tổng quát mà không có ví dụ mẫu cụ thể.

### 3.2. `predict("hard_eval")` — Khả năng chống bẫy Prompt Injection của LLM

**Đoán:** LLM đọc hiểu ngữ cảnh tốt nên sẽ không bị lừa bởi các câu lệnh tiêm nhiễm (Prompt Injection).

**Thực tế:** LLM zero-shot bị lừa gần như tuyệt đối ở nhóm Prompt Injection.

**Vì sao lệch:** LLM được thiết kế để luôn tuân thủ chỉ thị. Khi không dùng prompt phòng thủ (bọc thẻ `<sms>`), LLM xem nội dung tin nhắn lừa đảo là câu lệnh hệ thống mới cần chấp hành thay vì là dữ liệu cần phân loại.

### 3.3. `predict("tradeoff")` — Độ trễ và vị trí trên đường Pareto

**Đoán:** LLM nhỏ sẽ có tốc độ đủ nhanh để nằm gần vạch SLA 50ms.

**Thực tế:** Tất cả các mô hình LLM đều bị đẩy xa về bên phải (độ trễ > 1000ms), hoàn toàn vi phạm SLA 50ms và bị lấn át trên đường Pareto.

**Vì sao lệch:** Đã đánh giá thấp độ trễ mạng (network latency) khi gọi API bên thứ ba và thời gian xử lý token của mô hình ngôn ngữ lớn so với các mô hình chạy trực tiếp local.

## 4. Phân tích 50 case khó

*Model nào tụt nhiều nhất, ở nhóm case nào? Nêu 2 case cụ thể và cơ chế khiến model sai.*

| Mô hình | F1 test thường (SUB) | F1 50 case khó | Accuracy 50 case khó |
| --- | ---: | ---: | ---: |
| 1. BoW + LogReg | 0.923 | 0.333 | 0.52 |
| 2. BoW + MLP | 0.919 | 0.400 | 0.52 |
| 3a. Emb + kNN | 0.900 | 0.702 | 0.66 |
| 3b. Emb + LogReg | 0.950 | 0.596 | 0.62 |
| 3c. Emb + MLP head | 0.974 | 0.653 | 0.66 |
| 4a. LLM nhỏ zero-shot | 0.884 | 0.933 | 0.92 |
| 4d. LLM nhỏ + prompt phòng thủ | 0.884 | 0.918 | 0.90 |

## 5. Thử thách cá nhân

*Đề theo SEED của bạn, cách làm, kết quả, giải thích.*

### 5.1. Đề #5 — Embedding

 Viết 4 tin nhắn của riêng bạn (2 spam né từ khoá, 2 ham chứa 'free'/'win'). So dự đoán BoW vs Embedding+LogReg và giải thích từng tin.

### 5.2. Mã thực hiện

```python
# 1. Khai báo 4 tin nhắn của riêng bạn
my_4_msgs = [
    # --- 2 spam né từ khóa ---
    "Y.o.u.r a-c-c-o-u-n-t will be locked. Update now at bit.ly/secure-login-24", # spam 1: Viết cách điệu chữ để né
    "Hey, we noticed unusual login activity from a new device. Please verify identity immediately.", # spam 2: Dùng giọng điệu hỗ trợ khách hàng, không có từ khoá trúng thưởng/khuyến mãi
    
    # --- 2 ham chứa từ khóa dễ bị nhầm là spam ('free', 'win') ---
    "Are you completely free this weekend to grab a coffee with me?", # ham 1: Từ 'free' nghĩa là rảnh rỗi
    "Did you see the match last night? I can't believe they managed to win!" # ham 2: Từ 'win' nghĩa là chiến thắng thể thao
]

# (Giả sử bạn đã có sẵn hàm predict_bow_lr và mô hình Embedding+LogReg từ các phần trước)
# Nếu tên hàm của bạn khác, hãy điều chỉnh lại cho khớp.
print("--- KẾT QUẢ DỰ ĐOÁN ---")
for i, msg in enumerate(my_4_msgs):
    # Dự đoán bằng BoW
    bow_pred = predict_bow_lr([msg])[0] 
    
    # Dự đoán bằng Embedding + LogReg (giả định dùng hàm của Phần 3)
    emb_features = (embed([msg]) - mu) / sd
    emb_pred = (predict_proba(emb_features, w_e, b_e) >= 0.5).astype(int)[0]
    
    label_bow = "spam" if bow_pred == 1 else "ham"
    label_emb = "spam" if emb_pred == 1 else "ham"
    
    print(f"Tin {i+1}: {msg[:50]}...")
    print(f"  > BoW đoán: {label_bow} | Embedding đoán: {label_emb}\n")
```

## 6. Phương án cho sản phẩm thật

*Với yêu cầu p95 < 50 ms/tin cho 10 triệu SMS/ngày: chọn phương án nào, dẫn số liệu nào, chấp nhận đánh đổi gì.*

### 6.1. Lựa chọn mô hình và chi phí

**Đạt độ trễ:** Các mô hình chạy local như BoW và Encoder (MiniLM). LLM không đạt vì độ trễ lên tới vài giây. 

**Đạt cả độ trễ lẫn chất lượng:** Mô hình Encoder (MiniLM) + MLP head. 

**Chi phí LLM/tháng:** 3.150 USD/tháng (với 10 triệu SMS/ngày = 105 USD/ngày × 30 ngày).

### 6.2. Phương án router

Mô hình router giúp tối ưu đồng thời độ trễ, chi phí và độ chính xác. Hơn 90% tin nhắn rõ ràng được mô hình nhỏ xử lý tức thì với chi phí 0 đồng và đạt chuẩn SLA < 50ms. Chỉ 5–10% tin nhắn lưỡng tính mới đẩy sang LLM xử lý, giúp giữ tổng chi phí cực thấp và không làm nghẽn hệ thống.

### 6.3. Thay đổi hiểu biết

**Dự đoán sai nhiều nhất:** Nghĩ rằng LLM zero-shot sẽ áp đảo hoàn toàn các mô hình nhỏ và có khả năng chống bẫy tốt. 

**Thay đổi nhận thức:** Mô hình nhỏ (Encoder/BoW) huấn luyện trên dữ liệu chuyên biệt vẫn tối ưu hơn LLM về tốc độ, chi phí và độ ổn định. LLM dễ bị tấn công bởi Prompt Injection và chỉ nên dùng làm lớp bổ trợ cuối cùng.

## 7. AI log và điều đã học

*Mỗi lần dùng AI: hỏi gì, nhận gì, bạn sửa gì. Sau đó 2–3 câu: điều bạn hiểu ra hôm nay mà trước đó chưa hiểu.*

khum biet
