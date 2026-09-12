# Cài đặt môi trường và Dify

Áp dụng chung cho mọi loại bot và mọi kênh, chỉ cần làm một lần.

## Chuẩn bị

- Một máy Mac dùng làm máy chủ, chạy gần như liên tục (Windows/Linux dùng nguyên lý tương tự, khác giao diện cài đặt).
- API Key của một nhà cung cấp mô hình ngôn ngữ [LLM]: Anthropic, OpenAI, Google, hoặc một mô hình mã nguồn mở chạy cục bộ qua Ollama nếu ưu tiên riêng tư dữ liệu.
- Tài liệu hoặc dữ liệu tri thức của người dùng.

## Bước 1: Cài Docker Desktop

Docker tạo ra các "hộp cách ly" để chạy Dify mà không ảnh hưởng phần còn lại của máy.

```bash
# Nếu dùng Homebrew, agent có thể chạy trực tiếp lệnh này:
brew install --cask docker
```

Sau khi cài, mở Docker Desktop lần đầu để cấp quyền hệ thống (bước này cần người dùng tự bấm). Vào Settings > Resources: Dify yêu cầu tối thiểu 2 CPU ảo và 4 GB RAM để chạy được, nhưng nên cấp 8 GB RAM trở lên nếu máy có đủ, để hệ thống mượt hơn khi có nhiều Knowledge Base hoặc nhiều App cùng chạy.

## Bước 2: Cài Node.js và Git

```bash
brew install node git
node -v
npm -v
git --version
```

Cả ba lệnh kiểm tra đều cần in ra số phiên bản.

## Bước 3: Cài Dify

```bash
cd ~
git clone https://github.com/langgenius/dify.git
cd dify/docker
cp .env.example .env
docker compose up -d
```

Yêu cầu Docker Compose v2.24 trở lên (bản Docker Desktop mới đã đáp ứng). Chờ 3 đến 5 phút để tải image, khi mọi container ở trạng thái Running là xong.

Nếu cổng 80 trên máy đã bị chương trình khác chiếm dụng, sửa biến `EXPOSE_NGINX_PORT` trong file `.env` sang một cổng khác trước khi chạy `docker compose up -d`.

## Bước 4: Cấu hình ban đầu trên Dify

Đây là bước qua giao diện web, agent có trình duyệt tích hợp (Cowork, Antigravity) có thể tự thao tác, nếu không thì hướng dẫn người dùng từng cú click.

1. Mở http://localhost/install, tạo tài khoản Admin đầu tiên.
2. Vào Settings > Model Provider, chọn nhà cung cấp mô hình đã chuẩn bị, dán API Key.
3. Vào Knowledge > Create Knowledge, tải tài liệu tri thức lên (Markdown hoặc PDF), chọn chế độ chia đoạn Automatic nếu không chắc, chọn Retrieval Setting là Hybrid Search.
4. Vào Studio > Create from Blank, chọn loại App:
   - Chatbot: đủ dùng cho trợ lý cá nhân/thương hiệu dựa trên tài liệu phi cấu trúc.
   - Workflow hoặc Agent: bắt buộc nếu cần thêm Tool node (truy vấn Google Sheet, database, hay bất kỳ API ngoài nào), xem `docs/04-kien-truc-du-lieu-cho-bot.md`.
5. Gắn Knowledge Base vào App (mục Context), chọn Model, đặt Temperature khoảng 0.2 đến 0.4 để bám sát tài liệu.
6. Dán System Prompt theo mẫu ở `docs/03-mau-system-prompt.md`.
7. Nhấn Publish > Update, sau đó vào API Access > API Key > New Secret Key, lưu lại mã dạng app-xxxxxxxx, đây là khóa các file kết nối kênh (docs/05 đến 08) sẽ dùng.

Lặp lại bước 3 đến 7 cho mỗi App bot khác nhau bạn muốn tạo, tất cả chạy chung trên một Dify vừa cài.

Nên sao lưu dữ liệu Dify định kỳ ngay từ đầu, không đợi đến khi có sự cố mới nghĩ đến, xem hướng dẫn ở `docs/09-van-hanh-va-xu-ly-su-co.md`.
