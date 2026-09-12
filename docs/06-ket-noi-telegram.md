# Kết nối Telegram

Telegram dùng làm kênh phê duyệt riêng tư trong mọi thiết lập, và có thể vừa dùng làm kênh phê duyệt vừa làm kênh cộng đồng công khai nếu muốn (tạo một bot Telegram thứ hai cho việc đó, tách biệt với bot phê duyệt).

## Tạo bot Telegram

1. Mở Telegram, tìm @BotFather, gõ `/newbot`, đặt tên và username (kết thúc bằng chữ bot).
2. Lưu lại HTTP API Token @BotFather gửi về.
3. Tìm @userinfobot, nhấn Start, lấy dòng Id, đây là TELEGRAM_ADMIN_ID.

Cả hai bước này cần người dùng tự thao tác trên tài khoản Telegram cá nhân, agent không tự làm thay được.

## Dùng chung một bot Telegram phê duyệt cho nhiều kênh cộng đồng

Nếu người dùng chạy cả Zalo, Messenger, Viber cùng lúc, dùng chung một TELEGRAM_TOKEN và TELEGRAM_ADMIN_ID cho tất cả các file kết nối kênh là hợp lý, không cần tạo nhiều bot Telegram. Điều bắt buộc: mỗi kênh phải dùng một tiền tố request ID riêng (TIEN_TO_YEU_CAU trong code, ví dụ ZL cho Zalo, MS cho Messenger, VB cho Viber), nếu không hai kênh có thể sinh ra cùng một mã số và người quản trị bấm nhầm nút duyệt sang câu hỏi của kênh khác.

Logic xử lý callback_query (nút Duyệt / Tự viết) ở mỗi file kênh (`docs/05`, `07`, `08`) đã được viết độc lập theo từng file, mỗi file tự lắng nghe `teleBot.on("callback_query")` và `teleBot.on("message")` riêng. Nếu chạy nhiều file cùng lúc và cùng dùng một TELEGRAM_TOKEN, chỉ nên để một trong các file đó thật sự dùng `polling: true`, các file còn lại nên tắt polling và chỉ dùng `sendMessage`/`editMessageText` (gửi thông báo một chiều), để tránh xung đột polling trên cùng một bot. Cách đơn giản hơn cho người mới: mỗi kênh cộng đồng dùng một bot Telegram phê duyệt riêng, chấp nhận có vài bot Telegram thay vì một, đổi lấy việc không phải xử lý xung đột polling.
