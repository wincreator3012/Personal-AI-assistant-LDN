# Vận hành 24/7 và xử lý sự cố

## Khóa chế độ ngủ của máy Mac

```bash
sudo pmset -a disksleep 0
sudo pmset -a sleep 0
```

Hai lệnh này cần mật khẩu hệ thống, người dùng tự nhập.

## Chạy mọi tiến trình bằng PM2

```bash
npm install -g pm2

cd ~/Desktop/ten-du-an/kenh-zalo && pm2 start bot-zalo.js --name "kenh-zalo"
cd ~/Desktop/ten-du-an/kenh-messenger && pm2 start bot-messenger.js --name "kenh-messenger"
cd ~/Desktop/ten-du-an/kenh-viber && pm2 start bot-viber.js --name "kenh-viber"

# Nếu dùng Cloudflare Tunnel cho Messenger hoặc Viber, cũng đưa vào PM2:
pm2 start cloudflared --name "tunnel-messenger" -- tunnel --url http://localhost:3000

pm2 save
pm2 startup
```

Lệnh `pm2 startup` in ra một dòng bắt đầu bằng `sudo env PATH=...`, sao chép và chạy dòng đó, nhập mật khẩu khi được hỏi.

## Sao lưu dữ liệu Dify

Toàn bộ tri thức, System Prompt, và lịch sử hội thoại của Dify nằm trong các Docker volume, không nằm trong một file thường trên máy. Nếu Docker bị gỡ hoặc volume bị xóa nhầm, dữ liệu mất hoàn toàn nếu chưa sao lưu. Nên sao lưu định kỳ, hàng tuần là hợp lý cho quy mô cá nhân hoặc cộng đồng nhỏ:

```bash
docker volume ls | grep docker
```

Lệnh trên liệt kê các volume Dify đang dùng (tên cụ thể tùy phiên bản Dify). Với mỗi volume cần sao lưu:

```bash
docker run --rm -v ten_volume:/data -v ~/dify-backups:/backup alpine \
  tar czf /backup/ten_volume-$(date +%F).tar.gz -C /data .
```

Thay `ten_volume` bằng tên volume thật lấy được ở lệnh trước. Có thể đưa việc này vào một cron job chạy hàng tuần để không phải nhớ làm thủ công.

## Bảng lệnh quản trị nhanh

| Thao tác | Lệnh |
|---|---|
| Xem trạng thái tất cả tiến trình | `pm2 status` |
| Xem log một kênh | `pm2 logs kenh-zalo` (đổi tên theo kênh) |
| Khởi động lại một kênh | `pm2 restart kenh-zalo` |
| Tạm dừng một kênh | `pm2 stop kenh-zalo` |

## Xử lý sự cố thường gặp

- Bot Zalo không phản hồi: `pm2 logs kenh-zalo`, thường do API Key sai hoặc phiên đăng nhập hết hạn, chạy lại `node login.js` trong đúng thư mục kênh đó.
- Bot Messenger/Viber không nhận được tin: kiểm tra `cloudflared` hay `ngrok` có đang chạy không (`pm2 status`), kiểm tra URL webhook đã cập nhật đúng URL hiện tại chưa, vì URL từ ngrok bản miễn phí đổi mỗi lần khởi động lại.
- Webhook Messenger báo lỗi xác minh: kiểm tra VERIFY_TOKEN trong code có khớp với giá trị đã nhập trên Meta for Developers không.
- Webhook Viber báo lỗi khi set_webhook: kiểm tra chứng chỉ SSL của URL có hợp lệ không, Viber từ chối chứng chỉ tự ký.
- Không nhận được thông báo Telegram: kiểm tra TELEGRAM_TOKEN và TELEGRAM_ADMIN_ID, và người quản trị đã từng nhắn ít nhất một tin cho bot Telegram đó chưa.
- Dify báo lỗi kết nối liên tục: kiểm tra Docker Desktop có chạy không, `docker ps` xem các container Dify có ở trạng thái Running không.
- Tool Google Sheet báo lỗi không có quyền truy cập: kiểm tra sheet đã được chia sẻ [Share] cho đúng email của service account chưa, và quyền chia sẻ (Viewer/Editor) có đủ cho việc bot cần làm không.
