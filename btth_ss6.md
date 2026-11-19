#  BÀI TẬP THỰC HÀNH TỔNG HỢP

---

## I. BÁO CÁO THIẾT KẾ GIẢI PHÁP (SOLUTION ARCHITECTURE)

### 1. Bối cảnh & Mục tiêu bài toán
Bài toán yêu cầu xây dựng một luồng xử lý AI khép kín phục vụ tự học và khảo thí tự động: tiếp nhận tài liệu gốc của sinh viên, kiểm tra mức độ hiểu bài thông qua phương pháp **Feynman** (giải thích lại kiến thức bằng ngôn ngữ đơn giản), sau đó tự động chấm điểm dựa trên bộ tiêu chí **Rubric** và đưa ra lộ trình cải thiện lỗ hổng kiến thức.

### 2. Kiến trúc chuỗi Prompt liên kết (Prompt Chaining Pipeline)
Thay vì nhồi nhét toàn bộ yêu cầu vào một Prompt duy nhất dễ gây quá tải ngữ cảnh (Context Overflow) và giảm độ chính xác, hệ thống được chia thành **3 mắt xích (Chains)** chạy tuần tự, đầu ra của bước trước là đầu vào chuẩn hóa (JSON) của bước sau:

```text
[Tài liệu gốc + Bài làm SV] 
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│ CHAIN 1: Kiểm duyệt dữ liệu & Trích xuất kiến thức chuẩn │
│ (Data Guardrail & Knowledge Extractor)                   │
└──────┬───────────────────────────────────────────────────┘
       │ Output: JSON {status, core_concepts, rubrics}
       ▼
┌──────────────────────────────────────────────────────────┐
│ CHAIN 2: Gia sư Feynman phân tích & Tìm lỗ hổng tư duy   │
│ (Feynman Tutor & Gap Analyzer)                           │
└──────┬───────────────────────────────────────────────────┘
       │ Output: JSON {simplicity_score, jargon_list, gaps}
       ▼
┌──────────────────────────────────────────────────────────┐
│ CHAIN 3: Hội đồng khảo thí chấm Rubric & Tổng hợp báo cáo│
│ (Rubric Grader & Final Report Generator)                 │
└──────────────────────────────────────────────────────────┘
```
### Bước 2: Chain 2 - Gia sư Feynman đánh giá mức độ thấu hiểu 
#### ROLE
Bạn là Gia sư AI chuyên sâu về phương pháp Feynman (giải thích kiến thức phức tạp bằng ngôn ngữ đơn giản cho người mới bắt đầu).

#### INPUT
- Kiến thức chuẩn (từ Chain 1): {{CHAIN_1_OUTPUT.core_concepts}}
- Bài làm của sinh viên: {{STUDENT_ESSAY}}

#### TASK
Đối chiếu bài làm của sinh viên với `core_concepts` và phân tích theo 3 góc độ:
1. **Độ chính xác:** Sinh viên có làm sai lệch ý cốt lõi nào không?
2. **Tính đơn giản (Feynman):** Sinh viên có lạm dụng thuật ngữ chuyên ngành khó hiểu hoặc chép nguyên văn sách giáo khoa không?
3. **Ví dụ ẩn dụ:** Sinh viên có đưa ra ví dụ đời thường dễ hình dung không?

#### OUTPUT FORMAT (CHỈ TRẢ VỀ JSON RÚT GỌN)
{
  "missing_or_wrong_concepts": ["Lỗ hổng kiến thức 1"],
  "academic_jargons_used": ["Thuật ngữ khó hiểu chưa giải thích 1"],
  "has_good_analogy": true/false,
  "tutor_feedback": "Nhận xét ngắn gọn mang tính sư phạm",
  "socratic_questions": ["Câu hỏi gợi mở 1", "Câu hỏi gợi mở 2"]
}