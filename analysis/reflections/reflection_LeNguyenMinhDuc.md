# Individual Reflection — Lab 18

**Tên:** Lê Nguyễn Minh Đức  
**MSSV:** 2A202601013  
**Module phụ trách:** Implement 5 Modules (M1, M2, M3, M4, M5)

---

## 1. Đóng góp kỹ thuật

- Module đã implement: M1 (Chunking), M2 (Hybrid Search), M3 (Reranking), M4 (RAGAS Evaluation), M5 (Enrichment).
- Các hàm/class chính đã viết:
  - M1: `chunk_semantic`, `chunk_hierarchical`, `chunk_structure_aware`
  - M2: `segment_vietnamese`, `BM25Search`, `DenseSearch`, `reciprocal_rank_fusion`
  - M3: `CrossEncoderReranker` (bge-reranker-v2-m3)
  - M4: `evaluate_ragas`, `failure_analysis`
  - M5: `summarize_chunk`, `generate_hypothesis_questions`, `contextual_prepend`, `extract_metadata`, `_enrich_single_call`
- Số tests pass: 37 / 37 (100%)

## 2. Kiến thức học được

- Khái niệm mới nhất: Chunking theo thứ bậc Parent-Child và Reciprocal Rank Fusion (RRF) kết hợp kết quả tìm kiếm BM25 + Dense vector.
- Điều bất ngờ nhất: `underthesea` dùng dấu `_` nối từ ghép tiếng Việt, cần replace `_` thành space để BM25Okapi phân tách đúng token.
- Kết nối với bài giảng: Áp dụng kiến thức RERANKING (bge-reranker-v2-m3) và RAGAS metrics (Faithfulness, Answer Relevancy, Context Precision, Context Recall).

## 3. Khó khăn & Cách giải quyết

- Khó khăn lớn nhất: Tách từ tiếng Việt cho BM25 và tích hợp Qdrant vector search với fallback an toàn.
- Cách giải quyết: Sử dụng `replace("_", " ")` sau tokenization và triển khai fallback logic cho RAGAS/OpenAI khi chạy môi trường offline.
- Thời gian debug: ~30 phút.

## 4. Nếu làm lại

- Sẽ làm khác điều gì: Thử nghiệm thêm FlashRank reranker nhẹ hơn (<5ms) và tùy chỉnh custom prompt enrichment tốt hơn.
- Module nào muốn thử tiếp: GraphRAG và Hybrid Search nâng cao với HNSW index parameters.

## 5. Tự đánh giá

| Tiêu chí        | Tự chấm (1-5) |
| --------------- | ------------- |
| Hiểu bài giảng  | 4/5           |
| Code quality    | 4/5           |
| Teamwork        | 4/5           |
| Problem solving | 4/5           |
