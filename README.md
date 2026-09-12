# Playbook: dựng trợ lý AI đa kênh trên nền Dify

Biên soạn bởi: Thạc sĩ Lương Dũng Nhân (M.Ed, PCC, Humanistic AI Practitioner)

Repo này là một bộ hướng dẫn để một AI agent (Claude Code, Claude Cowork, Google Antigravity, Codex, hoặc tương đương) đọc và tự hướng dẫn bạn, từng bước, dựng một trợ lý AI mang kiến thức và giọng nói riêng của bạn, hoạt động trên các kênh nhắn tin bạn chọn (Zalo, Telegram, Facebook Messenger, Viber), có kênh phê duyệt riêng tư khi cần can thiệp thủ công.

## Cách dùng

Dán đường dẫn hoặc link repo này vào agent bạn đang dùng, rồi mô tả nhu cầu, ví dụ:

- "Đọc repo này, giúp mình dựng một bot tư vấn khóa học trên Zalo, dữ liệu khóa học mình để trong Google Sheet."
- "Mình đã có bot Zalo rồi, giờ muốn thêm kênh Messenger cho cùng trợ lý đó."
- "Mình chưa biết bắt đầu từ đâu, hỏi mình vài câu rồi hướng dẫn mình từng bước."

Agent nên đọc `AGENTS.md` trước tiên để biết cách vận hành đúng, sau đó mới vào các tài liệu trong `docs/`.

## Repo này dành cho ai

Người không rành kỹ thuật, muốn có một trợ lý AI riêng cho cộng đồng, khách hàng, hoặc học viên của mình, sẵn sàng dùng một chiếc máy Mac (hoặc máy tính tương đương) làm nơi chạy hệ thống, và có thời gian ngồi cùng agent để cá nhân hóa nội dung.

## Trước khi bắt đầu

Chuẩn bị sẵn ba thứ sau sẽ giúp quá trình dựng bot suôn sẻ hơn:

- Một máy Mac (hoặc Windows/Linux) có thể chạy gần như liên tục [24/7], vì hệ thống cần luôn hoạt động để trả lời tin nhắn.
- Một API Key của nhà cung cấp mô hình ngôn ngữ [LLM] bạn định dùng (Anthropic, OpenAI, Google, hoặc một mô hình mã nguồn mở chạy cục bộ).
- Tài liệu hoặc dữ liệu tri thức bạn muốn trợ lý dùng để trả lời (bài viết, sách, workbook, bảng giá, lịch học...).

## Mục lục

1. [Kiến trúc tổng quan](docs/01-kien-truc-tong-quan.md), đọc trước tiên để hiểu bức tranh chung và chọn hướng phù hợp
2. [Cài đặt môi trường và Dify](docs/02-cai-dat-moi-truong-va-dify.md)
3. [Mẫu System Prompt](docs/03-mau-system-prompt.md)
4. [Kiến trúc dữ liệu cho từng loại bot](docs/04-kien-truc-du-lieu-cho-bot.md)
5. [Kết nối Zalo](docs/05-ket-noi-zalo.md)
6. [Kết nối Telegram](docs/06-ket-noi-telegram.md)
7. [Kết nối Facebook Messenger](docs/07-ket-noi-messenger.md)
8. [Kết nối Viber](docs/08-ket-noi-viber.md)
9. [Vận hành 24/7 và xử lý sự cố](docs/09-van-hanh-va-xu-ly-su-co.md)
10. [Chú giải thuật ngữ](docs/10-chu-giai-thuat-ngu.md)

Không phải ai cũng cần đọc hết. Tài liệu 5 đến 8 là các kênh nhắn tin, chỉ cần đọc kênh bạn thực sự dùng.

---

Cập nhật lần cuối: tháng 9/2026. Các chi tiết kỹ thuật về Dify, Zalo (zca-js), Facebook Messenger, và Viber đã được kiểm chứng qua tìm kiếm web tại thời điểm biên soạn, nhưng các nền tảng này thay đổi liên tục, xem lưu ý ở cuối `AGENTS.md`.
