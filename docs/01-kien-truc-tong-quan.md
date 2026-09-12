# Kiến trúc tổng quan

## Bốn phần cấu thành hệ thống

1. Bộ não tri thức, chạy bằng Dify: nơi nạp tài liệu, kiến thức, dữ liệu, và System Prompt của bạn. Một Dify duy nhất có thể chứa nhiều App khác nhau, mỗi App là một "nhân cách" bot riêng với kho dữ liệu và System Prompt riêng. Không cần cài nhiều Dify cho nhiều bot.
2. Một hoặc nhiều kênh cộng đồng: nơi người dùng thật nhắn tin cho bot. Có thể là Zalo, Telegram, Messenger, Viber, hoặc kết hợp nhiều kênh cùng lúc cho cùng một App Dify.
3. Một kênh phê duyệt riêng tư, thường là Telegram: khi câu hỏi vượt tri thức có sẵn hoặc chạm vùng nhạy cảm, bot giữ câu trả lời lại và gửi về đây để người quản trị duyệt, sửa, hoặc tự viết trước khi đến tay người hỏi. Nhiều kênh cộng đồng có thể dùng chung một kênh phê duyệt.
4. Một "người quản gia" chạy nền, dùng PM2: giữ mọi tiến trình chạy liên tục, tự khởi động lại khi lỗi hoặc khi mở máy.

## So sánh nhanh các kênh nhắn tin

| Kênh | Tính chính thức | Chi phí | Yêu cầu hạ tầng | Rủi ro chính |
|---|---|---|---|---|
| Zalo | Không chính thức (mô phỏng tài khoản cá nhân qua zca-js) | Miễn phí | Không cần máy chủ công khai, chỉ cần máy chạy 24/7 | Tài khoản có thể bị khóa bất cứ lúc nào |
| Telegram | Chính thức (Bot API) | Miễn phí | Không cần máy chủ công khai, dùng polling | Thấp, nền tảng ổn định |
| Facebook Messenger | Chính thức (Messenger Platform) | Miễn phí | Bắt buộc có URL HTTPS công khai để nhận webhook | Cần một Trang Facebook, không dùng được với tài khoản cá nhân |
| Viber | Chính thức nhưng chỉ theo diện thương mại từ 05/02/2024 | 115 euro/tháng cho mỗi bot, cộng phí tin nhắn khởi tạo ngoài phiên hội thoại | Bắt buộc HTTPS công khai với chứng chỉ hợp lệ | Chỉ hợp với tổ chức có ngân sách và mục đích thương mại rõ ràng |

Zalo và Telegram không cần máy chủ có địa chỉ công khai vì cả hai đều hoạt động theo kiểu ứng dụng của bạn chủ động lắng nghe hoặc hỏi thăm máy chủ của họ. Messenger và Viber hoạt động ngược lại, họ chủ động gọi đến máy của bạn mỗi khi có tin nhắn mới [webhook], nên máy của bạn phải có một địa chỉ HTTPS mà internet nhìn thấy được. Nếu chạy trên một máy Mac tại nhà, cần thêm một dịch vụ tạo đường hầm công khai [tunnel] như Cloudflare Tunnel hoặc ngrok, xem chi tiết ở tài liệu kênh Messenger.

## Chọn kênh nào

- Cộng đồng Việt Nam, nhóm chat có sẵn: Zalo là lựa chọn thực tế nhất hiện nay dù không chính thức.
- Cần một kênh ổn định, chính thức, dễ vận hành nhất: Telegram, dù cộng đồng của bạn có thể chưa quen dùng.
- Có Trang Facebook đang hoạt động, muốn tận dụng lượng người theo dõi sẵn có: Messenger.
- Có ngân sách vận hành thật và khách hàng chủ yếu ở khu vực Viber phổ biến (một số nước Đông Âu, Đông Nam Á): mới cân nhắc Viber.

## Loại bot thường gặp và kiến trúc dữ liệu tương ứng

| Loại bot | Đặc điểm dữ liệu | Kiến trúc gợi ý | Xem thêm |
|---|---|---|---|
| Trợ lý cá nhân hoặc thương hiệu, trả lời theo giọng văn và kiến thức riêng | Tài liệu, bài viết, sách, workbook, phần lớn phi cấu trúc | Knowledge Base thường, App Chatbot đơn giản | docs/02, docs/03 |
| Tư vấn sản phẩm hoặc khóa học, cần tra đúng giá, tồn kho, lịch học, link đăng ký | Dữ liệu bán cấu trúc hoặc có cấu trúc, số lượng vừa phải | Metadata filtering trong Knowledge Base, hoặc Tool đọc trực tiếp từ Google Sheet | docs/04 |
| Tư vấn dựa trên hệ thống dữ liệu lớn, thay đổi liên tục (giá, tồn kho thật) | Dữ liệu có cấu trúc, quy mô lớn, cập nhật thường xuyên | Tool truy vấn database thật | docs/04 |

Xem chi tiết cách chọn và thiết lập ở `docs/04-kien-truc-du-lieu-cho-bot.md`.

## Ước tính chi phí vận hành tổng thể

Ngoài chi phí kênh nhắn tin ở bảng trên, còn ba khoản khác cần tính đến khi lên kế hoạch ngân sách:

- Chi phí gọi mô hình ngôn ngữ [LLM]: tính theo lượng token sử dụng thực tế, thay đổi theo nhà cung cấp và tần suất bot được hỏi, không có mức cố định.
- Máy chủ chạy hệ thống: nếu dùng máy Mac hoặc máy tính cá nhân đã có sẵn, chi phí phát sinh chủ yếu là điện năng cho máy chạy 24/7, không phải thuê thêm.
- Đường hầm công khai [tunnel] cho Messenger hoặc Viber: Cloudflare Tunnel miễn phí và ổn định cho vận hành lâu dài, ngrok miễn phí nhưng chỉ phù hợp thử nghiệm ngắn hạn.

Với Zalo, Telegram, và phần lớn trường hợp Messenger, chi phí vận hành gần như chỉ còn lại chi phí LLM. Viber là kênh duy nhất có phí cố định hàng tháng bất kể mức sử dụng, nên cân nhắc kỹ trước khi chọn.
