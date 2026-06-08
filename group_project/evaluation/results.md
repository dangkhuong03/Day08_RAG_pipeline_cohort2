# RAG Evaluation Results

## Framework sử dụng

**RAGAS** — chuẩn industry cho RAG evaluation, đánh giá theo 3 trục: faithfulness, relevance, context quality.

---

## Overall Scores

| Metric | hybrid_rerank | dense_only | Δ |
|--------|---------------|------------|---|
| faithfulness | 0.763 | 0.681 | -0.082 |
| answer_relevancy | 0.841 | 0.793 | -0.048 |
| context_recall | 0.724 | 0.608 | -0.116 |
| context_precision | 0.697 | 0.634 | -0.063 |
| **Average** | **0.756** | **0.679** | -0.077 |

---

## A/B Comparison Analysis

**Config A (hybrid_rerank):**
> Hybrid search = semantic (dense) + BM25 (lexical) → RRF merge → cross-encoder reranking. Full pipeline với đầy đủ components.

**Config B (dense_only):**
> Dense-only = chỉ semantic search bằng embedding cosine similarity, không có BM25 và không reranking. Baseline đơn giản.

**Kết luận:**
> Config A (hybrid_rerank) vượt trội Config B (dense_only): 0.756 vs 0.679 điểm trung bình (Δ=+0.077). Cải thiện lớn nhất ở **context_recall** (+0.116): BM25 bổ sung recall cho từ khoá pháp luật cụ thể (tên điều luật, số nghị định) mà embedding không capture được bằng semantic similarity. Reranking cải thiện precision của kết quả cuối cùng.

---

## Worst Performers (Bottom 3)

| # | Question | Faithfulness | Answer Relevancy | Context Recall | Failure Stage | Root Cause |
|---|----------|-------------|-----------------|----------------|---------------|------------|
| 1 | Cơ quan nào có thẩm quyền kiểm soát hoạt động h... | 0.512 | 0.731 | 0.333 | Retrieval | Retriever không tìm được context liên quan — câu hỏi về thẩm quyền cơ quan không có trong corpus |
| 2 | Người tự nguyện cai nghiện ma tuý tại gia đình... | 0.584 | 0.762 | 0.417 | Retrieval | Context recall thấp — chunk về hỗ trợ cai nghiện bị cắt ngang điều khoản |
| 3 | Vụ án của ca sĩ Châu Việt Cường liên quan đến ma... | 0.631 | 0.688 | 0.500 | Generation | LLM hallucinate chi tiết vụ án — thông tin báo chí không được index đầy đủ |

---

## Recommendations

### Cải tiến 1: Nâng cấp chunking strategy
**Action:** Dùng `MarkdownHeaderTextSplitter` để giữ nguyên cấu trúc điều khoản pháp luật (mỗi chunk = 1 điều, không cắt ngang điều khoản).  
**Expected impact:** Tăng context precision ~0.10–0.15 vì chunk không bị cắt giữa chừng khiến LLM mất ngữ cảnh pháp lý; trực tiếp giải quyết Worst Performer #2.

### Cải tiến 2: Vietnamese tokenizer cho BM25
**Action:** Tích hợp `underthesea` hoặc `pyvi` để word-segment tiếng Việt trước BM25, thay vì whitespace tokenization hiện tại.  
**Expected impact:** Tăng lexical search recall ~0.10 vì BM25 hiện nhầm lẫn token ghép âm tiết (VD: "ma tuý" vs "matuý", "cai nghiện" vs "cainghiện").

### Cải tiến 3: Mở rộng corpus tin tức
**Action:** Crawl thêm bài báo về các vụ án nghệ sĩ (2020–2025) từ VnExpress, Tuổi Trẻ, Thanh Niên; đảm bảo mỗi vụ án lớn có ít nhất 3 bài.  
**Expected impact:** Tăng context recall cho nhóm câu hỏi về tin tức ~0.20; giải quyết trực tiếp Worst Performer #3.
