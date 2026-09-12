# Kiến trúc dữ liệu cho từng loại bot

## Vì sao không phải cứ nhét hết vào Knowledge Base là xong

Knowledge Base của Dify dùng tìm kiếm ngữ nghĩa [semantic search], giỏi trả lời câu hỏi mở nhưng không đảm bảo chính xác tuyệt đối cho câu hỏi cần lọc đúng theo điều kiện (giá dưới một mức, còn hàng hay không, đúng một con số). Với dữ liệu càng có cấu trúc và câu hỏi càng cần độ chính xác cao, nên chuyển dần sang các cách tiếp cận có cấu trúc hơn.

## Ba mức giải pháp, từ nhẹ đến nặng

### Mức 1: Metadata filtering trong cùng một Knowledge Base

Gắn các trường có cấu trúc (giá, danh mục, tình trạng, thời gian) làm metadata dạng chuỗi, số, hoặc thời gian cho từng tài liệu, rồi lọc AND/OR trong node Knowledge Retrieval, kết hợp với tìm ngữ nghĩa cho phần mô tả tự do. Không cần App dạng Workflow, không cần dữ liệu bên ngoài. Phù hợp với danh mục vừa và nhỏ, ít thay đổi (vài chục mục, cập nhật không quá thường xuyên), ví dụ danh mục khóa học.

Cách làm trong Dify: vào Knowledge Base > Metadata > Add Metadata, tạo các trường cần thiết, gán giá trị cho từng tài liệu, sau đó bật Metadata Filtering trong node Knowledge Retrieval của App.

### Mức 2: Tool đọc trực tiếp một Google Sheet

Khi dữ liệu đang được người dùng duy trì sẵn trong một Google Sheet và muốn bot luôn đọc đúng bản mới nhất, không qua bước đồng bộ lại vào Knowledge Base.

Thiết lập:
1. Tạo project trên Google Cloud Console, bật Google Sheets API.
2. Tạo một service account, tải file khóa JSON.
3. Chia sẻ [Share] sheet dữ liệu cho đúng email của service account. Nếu bot chỉ cần đọc dữ liệu, cấp quyền Viewer là đủ, chỉ cấp Editor nếu bot cần ghi ngược lại vào sheet.
4. Trong Dify, cài plugin "Google Sheets" (tác giả omluc, tìm "google_sheets" trong Marketplace), dán nội dung file khóa JSON vào cấu hình plugin.
5. Tạo App dạng Workflow hoặc Agent (không dùng được với App Chatbot đơn giản), thêm Tool node gọi Batch Get để lấy dữ liệu từ sheet mỗi khi cần trả lời.

Cấu trúc sheet nên rõ ràng: dòng đầu là tiêu đề cột, mỗi dòng sau là một mục dữ liệu (một khóa học, một sản phẩm), mỗi cột một trường cố định.

Nguyên tắc quan trọng: với các trường cần chính xác tuyệt đối (giá, link đăng ký), thiết kế câu trả lời để chèn thẳng giá trị đọc được từ Tool vào, không để mô hình ngôn ngữ diễn giải lại bằng lời của nó, vì mô hình có thể vô tình đổi một con số hoặc một ký tự trong link dù đọc rất tự nhiên.

### Mức 3: Tool truy vấn database thật

Khi dữ liệu quy mô lớn, thay đổi liên tục (giá, tồn kho của một hệ thống bán hàng thật), nên kết nối trực tiếp vào database đang vận hành thay vì sao chép ra nơi khác.

Dify Marketplace có nhiều plugin dạng truy vấn database, tên và mức độ hỗ trợ có thể khác nhau theo thời điểm vì đây là nhóm plugin cập nhật thường xuyên. Một vài lựa chọn đáng xem tại thời điểm biên soạn: "Database Query" (tác giả Junjie.M, hỗ trợ MySQL, PostgreSQL, MSSQL, Oracle), "database" (tác giả hjlarry), "HelloDB" (tác giả cdnxy). Tìm từ khóa "database" hoặc "SQL" trong Marketplace để xem danh sách mới nhất trước khi chọn. Cần App dạng Workflow hoặc Agent.

Lưu ý bảo mật bắt buộc: tạo một tài khoản chỉ đọc [read-only] riêng cho Dify kết nối vào, không dùng tài khoản có quyền ghi hoặc xóa, để tránh rủi ro khi mô hình tự sinh câu lệnh SQL.

## Kết hợp nhiều mức trong cùng một bot

Có thể phối hợp: phần mô tả dài, mang tính tư vấn, gợi ý, so sánh, vẫn để trong Knowledge Base tìm theo ngữ nghĩa; còn phần dữ kiện cần chính xác (giá, tồn kho, lịch, link) lấy qua Tool (mức 2 hoặc 3). Thêm một bước phân loại câu hỏi ở đầu Workflow: câu hỏi cần dữ kiện chính xác thì gọi Tool, câu hỏi mở thì gọi Knowledge Retrieval, rồi tổng hợp lại thành một câu trả lời.

## Bảng quyết định nhanh

| Số lượng mục dữ liệu | Tần suất thay đổi | Cần chính xác tuyệt đối không | Chọn |
|---|---|---|---|
| Vài chục, ổn định | Thấp | Không bắt buộc | Knowledge Base thường |
| Vài chục đến vài trăm | Trung bình, người dùng tự cập nhật | Có (giá, link) | Metadata filtering, hoặc Tool Google Sheet |
| Hàng trăm trở lên, thay đổi liên tục | Cao | Bắt buộc | Tool truy vấn database thật |
