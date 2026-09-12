# Hướng dẫn vận hành dành cho AI agent

Tài liệu này quy định cách bạn (agent đang đọc repo này) làm việc với người dùng. Áp dụng cho mọi agent: Claude Code, Claude Cowork, Google Antigravity, Codex, hoặc tương đương.

## Nguyên tắc chung

- Đây là playbook, không phải code chạy sẵn. Mọi System Prompt, mọi giá trị cấu hình, đều phải được điền bằng nội dung thật của người dùng, không tự bịa thay họ.
- Luôn đọc đúng tài liệu liên quan trong `docs/` trước khi hướng dẫn hoặc sinh code, không dựa vào trí nhớ chung chung về Dify hay các API nhắn tin, vì các nền tảng này thay đổi liên tục.
- Ưu tiên hỏi trước khi đoán. Một câu hỏi rõ ràng tốt hơn một giả định sai khiến người dùng phải làm lại từ đầu.

## Bước đầu tiên với mọi người dùng mới

Hỏi ba việc, có thể gộp thành một lượt hỏi ngắn:

1. Trợ lý này phục vụ mục đích gì (tư vấn sản phẩm hoặc khóa học, hỗ trợ chung cho cộng đồng, trả lời câu hỏi thường gặp nội bộ, hay việc khác)
2. Kênh nhắn tin nào cần dùng, có thể chọn nhiều hơn một (Zalo, Telegram, Facebook Messenger, Viber)
3. Dữ liệu họ có ở dạng nào (tài liệu văn bản rời rạc, một bảng tính có cấu trúc như Google Sheet, hay một hệ thống/database đang vận hành thật)

Dựa vào câu trả lời, đọc `docs/01-kien-truc-tong-quan.md` để định hình lộ trình, và `docs/04-kien-truc-du-lieu-cho-bot.md` để chọn đúng kiến trúc dữ liệu, trước khi đi vào các bước cài đặt.

## Khi hướng dẫn cài đặt

- Đi từng bước một, chờ người dùng xác nhận đã xong bước hiện tại trước khi sang bước kế, vì đây là người không rành kỹ thuật.
- Với các thao tác cần Terminal, nếu agent có quyền chạy lệnh trực tiếp (Cowork, Antigravity, Claude Code), hãy tự chạy các lệnh không đụng đến tài khoản cá nhân hoặc mật khẩu hệ thống của người dùng. Với các thao tác sau, LUÔN dừng lại để người dùng tự làm, không thay họ:
  - Nhập mật khẩu hệ điều hành khi được yêu cầu (cài Docker, chạy lệnh có sudo)
  - Quét mã QR đăng nhập Zalo bằng điện thoại
  - Chat với @BotFather hoặc @userinfobot trên tài khoản Telegram cá nhân
  - Đăng nhập Meta for Developers hoặc Viber Partner Cabinet bằng tài khoản cá nhân của họ
  - Bất cứ bước nào tạo ra chi phí thật (đăng ký gói trả phí, đăng ký Viber thương mại)
- Nếu người dùng chưa cài Docker Desktop, Node.js, hay Git, xem `docs/02-cai-dat-moi-truong-va-dify.md`, có thể tự động hóa phần cài đặt qua Terminal khi agent có quyền, nhưng vẫn nên thông báo trước mỗi lệnh sẽ chạy.

## Khi soạn System Prompt

Dùng mẫu ở `docs/03-mau-system-prompt.md`. Không tự viết nội dung thay cho phần "vai trò, giọng văn, nguyên tắc, thuật ngữ riêng", đây phải là nội dung người dùng cung cấp qua trò chuyện. Vai trò của agent là hỏi đúng câu để lấy được nội dung đó, rồi lắp vào đúng cấu trúc mẫu. Ví dụ minh họa trong tài liệu 03 chỉ để tham khảo cách lắp ráp, không phải nội dung để sao chép cho người dùng thật.

Mã báo động [escalation tag] người dùng chọn phải được ghi nhớ và dùng nhất quán ở mọi nơi: trong System Prompt trên Dify, và trong mọi file code kết nối kênh (Zalo, Messenger, Viber...). Nếu người dùng dùng nhiều kênh, tất cả các kênh dùng chung một mã báo động, vì chúng cùng gọi vào một Dify App.

## Khi sinh code kết nối kênh

Dùng đúng khung code mẫu trong tài liệu kênh tương ứng (`docs/05` đến `docs/08`), điền các giá trị cấu hình theo thông tin người dùng cung cấp, không tự sáng tác kiến trúc khác. Nếu người dùng dùng nhiều kênh cộng đồng cùng lúc và muốn dùng chung một kênh Telegram phê duyệt, thêm tiền tố theo kênh vào mã yêu cầu [request ID] để tránh trùng số giữa các kênh, ví dụ ZL-101 cho Zalo, MS-101 cho Messenger, xem chi tiết ở `docs/06-ket-noi-telegram.md`.

## Về Viber

Không mặc định đề xuất Viber cho người dùng cá nhân hoặc dự án không có ngân sách vận hành. Từ ngày 05/02/2024, Viber chỉ cho tạo bot theo diện thương mại, có phí duy trì hàng tháng (115 euro cho mỗi bot, tính đến thời điểm biên soạn) và cần xét duyệt qua đối tác chính thức hoặc trực tiếp với Rakuten Viber. Chỉ đi vào `docs/08-ket-noi-viber.md` nếu người dùng xác nhận họ có nhu cầu và ngân sách cho việc này.

## Về tính chính xác thông tin nền tảng

Zalo, Meta (Messenger), Viber, và Dify đều là nền tảng cập nhật liên tục. Nếu agent có khả năng tìm kiếm web, nên xác minh lại các bước có thể đã thay đổi (tên nút bấm, endpoint API, chính sách xét duyệt, giá) trước khi hướng dẫn người dùng thực hiện, thay vì chỉ dựa vào nội dung tĩnh trong repo. Các con số và chi tiết trong repo này (giá Viber, yêu cầu tài nguyên Dify, tên plugin Marketplace...) đã được kiểm chứng qua tìm kiếm web vào tháng 9/2026, nhưng có thể đã thay đổi kể từ đó.
