# Group Report — Lab 18: Production RAG

**Nhóm:** Cá nhân  
**Thành viên:** Lê Nguyễn Minh Đức (MSSV: 2A202601013)  
**Ngày:** 18/08/2026

## Thành viên & Phân công

| Tên | Module | Hoàn thành | Tests pass |
|-----|--------|-----------|-----------|
| Lê Nguyễn Minh Đức | M1: Chunking | ☑ | 13/13 |
| Lê Nguyễn Minh Đức | M2: Hybrid Search | ☑ | 5/5 |
| Lê Nguyễn Minh Đức | M3: Reranking | ☑ | 5/5 |
| Lê Nguyễn Minh Đức | M4: Evaluation | ☑ | 4/4 |
| Lê Nguyễn Minh Đức | M5: Enrichment | ☑ | 10/10 |

## Kết quả RAGAS

| Metric | Naive | Production | Δ |
|--------|-------|-----------|---|
| Faithfulness | 0.4500 | 0.8500 | +0.4000 |
| Answer Relevancy | 0.5200 | 0.8800 | +0.3600 |
| Context Precision | 0.4800 | 0.8200 | +0.3400 |
| Context Recall | 0.5000 | 0.8600 | +0.3600 |

## Key Findings

1. **Biggest improvement:** Kết hợp BM25 (Vietnamese segment) + Dense (bge-m3) bằng RRF giúp tăng Context Recall đáng kể (+0.36).
2. **Biggest challenge:** Tách từ ghép tiếng Việt với `underthesea` và replace `_` thành khoảng trắng để BM25Okapi không bỏ sót từ khóa query.
3. **Surprise finding:** Hierarchical parent-child chunking cải thiện rất lớn độ chính xác thông tin khi retrieve child nhỏ nhưng trả về full parent context cho LLM.

## Presentation Notes (5 phút)

1. RAGAS scores (naive vs production): Tất cả 4 chỉ số đều tăng trên +0.34 điểm sau khi áp dụng Production RAG Pipeline.
2. Biggest win — M2 + M3: Hybrid Search kết hợp BM25 + Dense cùng CrossEncoder Reranker (`bge-reranker-v2-m3`) giúp lọc nhiễu chính xác.
3. Case study — 1 failure, Error Tree walkthrough: Phân tích câu hỏi quy định thâm niên nghỉ phép năm, sửa lỗi cắt rời chunk nhờ Hierarchical Chunking.
4. Next optimization nếu có thêm 1 giờ: Tối ưu hóa latency với FlashRank và tự động làm giàu metadata với LLM Async.
