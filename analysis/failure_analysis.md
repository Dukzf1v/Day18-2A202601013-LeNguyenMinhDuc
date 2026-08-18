# Failure Analysis — Lab 18: Production RAG

**Nhóm:** Cá nhân  
**Thành viên:** Lê Nguyễn Minh Đức (MSSV: 2A202601013) — Implement M1, M2, M3, M4, M5

---

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.4500 | 0.8500 | +0.4000 |
| Answer Relevancy | 0.5200 | 0.8800 | +0.3600 |
| Context Precision | 0.4800 | 0.8200 | +0.3400 |
| Context Recall | 0.5000 | 0.8600 | +0.3600 |

## Bottom-5 Failures

### #1
- **Question:** Thời gian nghỉ phép năm của nhân viên thâm niên trên 5 năm được tính như thế nào?
- **Expected:** Tăng thêm 1 ngày cho mỗi 5 năm thâm niên công tác.
- **Got:** Không tìm thấy thông tin cụ thể.
- **Worst metric:** context_recall
- **Error Tree:** Output sai → Context chưa chứa đủ thông tin → Chắc chắn do cắt chunk nhỏ làm rơi mất đoạn thâm niên.
- **Root cause:** Basic chunking cắt đoạn giữa câu quy định thâm niên và phần số ngày được cộng.
- **Suggested fix:** Chuyển sang Hierarchical Parent-Child Chunking để khi retrieve child vẫn trả lại full parent context.

### #2
- **Question:** Mật khẩu tài khoản nội bộ cần thay đổi định kỳ như thế nào?
- **Expected:** Thay đổi mỗi 90 ngày.
- **Got:** Mật khẩu phải gồm ít nhất 8 ký tự.
- **Worst metric:** context_precision
- **Error Tree:** Output sai → Retrieve trả về nhiều đoạn IT policy chung chung không liên quan đến thời hạn thay đổi.
- **Root cause:** Dense search trả về đoạn quy định độ dài mật khẩu thay vì chu kỳ đổi.
- **Suggested fix:** Thêm BM25 + RRF Hybrid search và Cross-Encoder Reranking để đẩy chunk "90 ngày" lên top 1.

### #3
- **Question:** Quy trình nộp đơn nghỉ không lương vượt quá 30 ngày?
- **Expected:** Cần được Giám đốc bộ phận phê duyệt.
- **Got:** Nghỉ không lương tối đa 30 ngày.
- **Worst metric:** answer_relevancy
- **Error Tree:** Context đúng → Prompt LLM chưa yêu cầu nêu rõ cấp phê duyệt.
- **Root cause:** System prompt ở phần LLM generation quá ngắn.
- **Suggested fix:** Cải thiện prompt template trong LLM answer generator.

### #4
- **Question:** Thời gian thử việc đối với vị trí kỹ sư là bao nhiêu?
- **Expected:** Thời gian thử việc là 60 ngày.
- **Got:** Thử việc theo quy định hợp đồng.
- **Worst metric:** faithfulness
- **Error Tree:** Context đúng → LLM suy đoán câu trả lời chung chung.
- **Root cause:** LLM hallucinate do prompt chưa thắt chặt điều kiện "Chỉ trả lời dựa trên context".
- **Suggested fix:** Hạ temperature = 0.0 và siết chặt constraint trong System Prompt.

### #5
- **Question:** Giấy xác nhận y tế khi nghỉ ốm phải nộp trong bao lâu?
- **Expected:** Trong vòng 3 ngày làm việc.
- **Got:** Không tìm thấy.
- **Worst metric:** context_recall
- **Error Tree:** Query "nghỉ ốm" không match từ khóa do BM25 tách từ ghép chưa chuẩn.
- **Root cause:** Từ ghép "nghỉ_ốm" bị tokenizer nối bằng `_`.
- **Suggested fix:** Thêm bước `segment_vietnamese` bằng `underthesea` và thay `_` bằng khoảng trắng.

## Case Study (cho presentation)

**Question chọn phân tích:** Thời gian nghỉ phép năm của nhân viên thâm niên trên 5 năm được tính như thế nào?

**Error Tree walkthrough:**
1. Output đúng? → Chưa đúng.
2. Context đúng? → Sai, do thiếu chunk chứa quy định thâm niên.
3. Query rewrite OK? → Đã OK.
4. Fix ở bước: Module 1 (Hierarchical Chunking) & Module 2 (Hybrid BM25 + Dense).

**Nếu có thêm 1 giờ, sẽ optimize:**
- Tích hợp thêm FlashRank Reranker để giảm latency reranking xuống < 5ms.
- Thử nghiệm Hypothetical Document Embeddings (HyDE) cho Module 5 Enrichment.
