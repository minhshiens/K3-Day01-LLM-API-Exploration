# K3 — Ngày 1: Bài Tập & Phản Ánh
## Khám Phá LLM API | Phiếu Thực Hành

**Thời lượng:** 9h00–13h00
**Cách làm:** Trả lời từng câu ngay sau khi hoàn thành block tương ứng —
đừng để dồn hết về cuối buổi. Thay dòng `*Câu trả lời của bạn*` bằng câu
trả lời thật (chấm tự động sẽ đếm số câu đã trả lời).

---

## Block 1 — API Cơ Bản (trả lời sau Checkpoint 1)

### Câu 1.1 — Độ nhạy của temperature
Gọi `call_openai` với temperature 0.0, 0.5, 1.0 và 1.5 dùng prompt
**"Hãy kể cho tôi một sự thật thú vị về Việt Nam."**

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> - Temp 0.0: Phản hồi rất nhất quán, chạy vài lần đều ra câu trả lời chuẩn về hang Sơn Đoòng.
> - Temp 0.5 - 1.0: Văn phong tự nhiên, câu từ linh hoạt và phong phú hơn.
> - Temp 1.5: Bắt đầu bị ngáo, dùng từ ngẫu nhiên và câu từ bị chắp vá sai logic.
> Quy luật: Temp càng cao thì model càng ngẫu nhiên và sáng tạo hơn nhưng rủi ro trả lời sai sự thật, hallucination càng lớn.

### Câu 1.2 — Chọn temperature cho sản phẩm
**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> Nên đặt tầm 0.0 đến 0.2. Vì chatbot CSKH cần thông tin chính xác tuyệt đối về giá cả và chính sách, đặt temp thấp để tránh tình trạng bot bị ảo giác hay phán sai lệch.

### Câu 1.3 — Đánh đổi chi phí
Kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người gọi API 3 lần,
mỗi lần trung bình ~350 token đầu ra.

**Ước tính GPT-4o đắt hơn GPT-4o-mini bao nhiêu lần cho workload này? Nêu một
trường hợp GPT-4o xứng đáng với chi phí và một trường hợp nên dùng mini:**
> - GPT-4o đắt hơn mini khoảng 16.7 lần ($0.010 vs $0.0006/1k output token). Tính ra 30k lượt/ngày thì 4o tốn khoảng $105/ngày còn mini chỉ tốn $6.3/ngày.
> - GPT-4o xứng đáng khi làm bài toán suy luận khó như phân tích tài chính hay viết code hệ thống phức tạp.
> - Nên dùng mini cho tác vụ đơn giản như chatbot FAQ hoặc phân loại tin nhắn để tiết kiệm chi phí.

---

## Block 2 — System Prompt & Token (trả lời sau Checkpoint 2)

### Câu 2.1 — Sức mạnh của persona
Gọi `chat_with_system_prompt` hai lần với cùng câu hỏi
**"Giải thích blockchain là gì?"** nhưng hai system prompt khác nhau:
- "Bạn là giáo viên tiểu học, giải thích thật đơn giản cho trẻ 8 tuổi."
- "Bạn là chuyên gia tài chính, trả lời chuyên sâu bằng thuật ngữ kỹ thuật."

**Hai phản hồi khác nhau như thế nào (độ dài, từ vựng, ví dụ)? System prompt
ảnh hưởng đến hành vi model ra sao?** (3–4 câu)
> - Giáo viên tiểu học: Dùng từ siêu đơn giản, lấy ví dụ ví dụ như cuốn sổ tay lớp học cho trẻ 8 tuổi dễ hình dung.
> - Chuyên gia tài chính: Dùng nhiều thuật ngữ chuyên ngành như DLT, cơ chế đồng thuận, mã hóa.
> System prompt đóng vai trò định hình vai trò, quyết định phong cách xưng hô, vốn từ và độ sâu câu trả lời của model.

### Câu 2.2 — tiktoken vs đếm từ
Chọn một đoạn văn tiếng Việt ~100 từ. So sánh số token theo `count_tokens`
(tiktoken) với ước lượng `số từ / 0.75` mà Part 1 đã dùng.

**Hai con số chênh nhau bao nhiêu phần trăm? Vì sao tiếng Việt thường tốn
nhiều token hơn tiếng Anh cùng độ dài?**
> - Tiktoken đếm đoạn 100 từ tiếng Việt ra khoảng 190 token, chênh tầm 40% so với ước lượng thô (`100/0.75 ≈ 133` token).
> - Tiếng Việt tốn token hơn vì bảng mã BPE của OpenAI tối ưu cho tiếng Anh, các từ tiếng Việt có dấu hay bị tách thành nhiều byte sub-token lẻ.

---

## Block 3 — Streaming & Độ Bền (trả lời sau Checkpoint 3)

### Câu 3.1 — Trải nghiệm người dùng với streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì
non-streaming lại phù hợp hơn?** (1 đoạn văn)
> Streaming quan trọng nhất khi làm chatbot tương tác trực tiếp để người dùng nhìn thấy chữ ra ngay, không bị cảm giác chờ lâu. Còn non-streaming phù hợp cho các tác vụ xử lý ngầm (background job), gọi API lấy dữ liệu JSON chuẩn để code phía sau xử lý tiếp.

### Câu 3.2 — Vì sao backoff theo cấp số nhân?
**So với delay cố định (ví dụ luôn chờ 1 giây), exponential backoff có lợi
thế gì khi API bị quá tải? Điều gì xảy ra nếu hàng nghìn client cùng retry
với delay cố định giống nhau?**
> Exponential backoff giúp giãn thời gian chờ tăng dần (0.1s, 0.2s, 0.4s...) để server quá tải có thời gian hồi phục. Nếu hàng nghìn client cùng retry cố định sau 1s sẽ tạo ra đợt nghẽn trùng nhau (retry storm), khiến server sập liên tục và không thể hồi phục.

---

## Block 4 — Mini-Project (trả lời sau Checkpoint 4)

### Câu 4.1 — Thiết kế persona
**Bạn chọn persona gì cho trợ lý của mình? Viết lại system prompt đó và giải
thích 1–2 lựa chọn từ ngữ quan trọng trong prompt (ví dụ: vì sao yêu cầu
"trả lời ngắn gọn", vì sao chỉ định ngôn ngữ...):**
> System prompt: "Bạn là trợ giảng thân thiện của khóa AI, trả lời ngắn gọn bằng tiếng Việt."
> - Thêm "ngắn gọn" để giảm token output, giúp tiết kiệm tiền và trả lời nhanh hơn.
> - Chỉ định "tiếng Việt" để câu trả lời nhất quán ngôn ngữ với học viên trong lớp.

### Câu 4.2 — Hạn chế & cải thiện
**Trợ lý của bạn hiện có hạn chế lớn nhất là gì (ví dụ: history chỉ 3 lượt,
không có bộ nhớ dài hạn, không kiểm duyệt nội dung...)? Đề xuất một cải
thiện cụ thể và mô tả ngắn cách triển khai:**
> Hạn chế lớn nhất là history chỉ lưu 3 lượt gần nhất (`history[-6:]`), nói chuyện lâu xíu là bot quên mất ngữ cảnh ban đầu. Cách cải thiện là dùng kỹ thuật tóm tắt ngữ cảnh (context summarization): khi history dài thì cho LLM tóm tắt đoạn cũ thành 1 đoạn ngắn rồi đính ở đầu history.

---

## Danh Sách Kiểm Tra Nộp Bài

- [ ] `python grade.py` — xem điểm tự động, mục tiêu ≥ 75/100
- [ ] Cả 4 checkpoint pytest đều pass
- [ ] Tất cả 9 câu trong file này đã được trả lời
- [ ] Đã copy bài làm vào folder `solution/` và zip theo hướng dẫn README
