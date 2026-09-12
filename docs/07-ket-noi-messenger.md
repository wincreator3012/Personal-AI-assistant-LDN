# Kết nối Facebook Messenger

Messenger là kênh chính thức, dùng Messenger Platform của Meta, hoạt động theo cơ chế webhook: Meta chủ động gọi đến máy chủ của bạn mỗi khi có tin nhắn mới, khác với Zalo và Telegram là ứng dụng của bạn tự đi lấy tin.

Giao diện Meta for Developers và các yêu cầu xét duyệt thay đổi khá thường xuyên. Trước khi hướng dẫn người dùng từng bước, nên đối chiếu nhanh với tài liệu hiện hành tại developers.facebook.com/docs/messenger-platform để chắc chắn tên nút bấm và luồng thao tác vẫn khớp với mô tả dưới đây.

## Điều kiện bắt buộc

- Một Trang Facebook [Page] đang hoạt động, không dùng được với tài khoản cá nhân.
- Một tài khoản Meta for Developers.
- Một địa chỉ HTTPS công khai để Meta gọi webhook đến. Nếu chạy trên máy tại nhà (Mac Mini, v.v.), cần một trong hai cách:
  - Cloudflare Tunnel (`cloudflared`): miễn phí, ổn định, khuyến nghị cho vận hành lâu dài.
  - ngrok: dễ dùng cho thử nghiệm nhanh, bản miễn phí đổi địa chỉ mỗi lần khởi động lại, không hợp cho vận hành lâu dài.

## Thiết lập trên Meta for Developers (người dùng tự thao tác)

1. Vào developers.facebook.com, tạo App loại Business.
2. Thêm sản phẩm Messenger vào App.
3. Trong phần Messenger > Settings, tạo Page Access Token cho Trang Facebook muốn dùng.
4. Cấu hình Webhook: dán URL công khai của bạn (dạng `https://ten-mien-cua-ban/webhook`), đặt một Verify Token tự chọn (một chuỗi bí mật do bạn đặt ra để Meta xác minh webhook là của bạn), đăng ký sự kiện `messages` và `messaging_postbacks`.

## Bật đường hầm công khai (agent có thể tự chạy)

```bash
brew install cloudflared
cloudflared tunnel --url http://localhost:3000
```

Lệnh trên in ra một URL dạng `https://xxxx.trycloudflare.com`, dùng URL này làm webhook ở bước 4 trên (thêm `/webhook` vào cuối). Với vận hành lâu dài, nên đăng ký một tên miền riêng và cấu hình Cloudflare Tunnel cố định thay vì dùng URL ngẫu nhiên đổi mỗi lần khởi động lại.

## Thiết lập dự án

```bash
mkdir -p ~/Desktop/ten-du-an/kenh-messenger
cd ~/Desktop/ten-du-an/kenh-messenger
npm init -y
npm install express axios node-telegram-bot-api body-parser
```

## Code điều phối (bot-messenger.js)

```javascript
const express = require("express");
const axios = require("axios");
const TelegramBot = require("node-telegram-bot-api");

// ==================== CẤU HÌNH ====================
const PORT = 3000;
const VERIFY_TOKEN = "ĐIỀN_VERIFY_TOKEN_TỰ_CHỌN";
const PAGE_ACCESS_TOKEN = "ĐIỀN_PAGE_ACCESS_TOKEN";
const DIFY_API_URL = "http://localhost/v1/chat-messages";
const DIFY_API_KEY = "ĐIỀN_MÃ_APP_API_DIFY";
const TELEGRAM_TOKEN = "ĐIỀN_TOKEN_BOT_TELEGRAM";
const TELEGRAM_ADMIN_ID = "ĐIỀN_TELEGRAM_ID_CỦA_BẠN";
const TEN_TRO_LY = "ĐIỀN_TÊN_TRỢ_LÝ";
const MA_BAO_DONG = "ĐIỀN_MÃ_BÁO_ĐỘNG"; // phải khớp System Prompt
const TIEN_TO_YEU_CAU = "MS";
// =====================================================

const app = express();
app.use(express.json());
// Chỉ gửi thông báo, KHÔNG polling, để tránh xung đột nếu dùng chung bot Telegram với kênh khác
const teleBot = new TelegramBot(TELEGRAM_TOKEN, { polling: false });
const pendingTasks = new Map();
let requestCounter = 100;

// Bước xác minh webhook (Meta gọi 1 lần khi bạn lưu cấu hình)
app.get("/webhook", (req, res) => {
    if (req.query["hub.verify_token"] === VERIFY_TOKEN) {
        res.send(req.query["hub.challenge"]);
    } else {
        res.sendStatus(403);
    }
});

// Nhận tin nhắn thật
app.post("/webhook", async (req, res) => {
    res.sendStatus(200); // luôn trả 200 ngay để Meta không gửi lại
    const entries = req.body.entry || [];
    for (const entry of entries) {
        for (const event of entry.messaging || []) {
            if (!event.message || !event.message.text) continue;
            const senderId = event.sender.id;
            const query = event.message.text.trim();

            try {
                const dify = await axios.post(DIFY_API_URL, {
                    inputs: {}, query, response_mode: "blocking", user: senderId
                }, {
                    headers: { Authorization: `Bearer ${DIFY_API_KEY}`, "Content-Type": "application/json" },
                    timeout: 60000
                });
                let answer = dify.data.answer.trim();

                if (answer.includes(`[${MA_BAO_DONG}]`)) {
                    requestCounter++;
                    const reqId = `${TIEN_TO_YEU_CAU}-${requestCounter}`;
                    const suggestedAnswer = answer.replace(`[${MA_BAO_DONG}]`, "").trim();
                    pendingTasks.set(reqId, { senderId, query, suggestedAnswer });

                    await sendMessengerText(senderId, "Câu hỏi này cần thêm thời gian để trả lời cho thấu đáo, mình đã chuyển đến quản trị viên nhé!");
                    await teleBot.sendMessage(TELEGRAM_ADMIN_ID,
                        `⚠️ *CẦN DUYỆT [#${reqId}]* (kênh Messenger)\n❓ "${query}"\n\n🌱 _${suggestedAnswer}_`,
                        { parse_mode: "Markdown", reply_markup: { inline_keyboard: [
                            [{ text: "✅ Duyệt gửi", callback_data: `APPROVE_${reqId}` }]
                        ]}});
                } else {
                    await sendMessengerText(senderId, answer);
                }
            } catch (err) {
                console.error("Lỗi kết nối Dify:", err.message);
                await sendMessengerText(senderId, "Hiện tại nhịp kết nối tri thức đang chậm, bạn chờ mình chút nhé.");
            }
        }
    }
});

async function sendMessengerText(recipientId, text) {
    await axios.post(`https://graph.facebook.com/v21.0/me/messages?access_token=${PAGE_ACCESS_TOKEN}`, {
        recipient: { id: recipientId },
        message: { text }
    });
}

teleBot.on("callback_query", async (cb) => {
    const action = cb.data;
    if (action.startsWith("APPROVE_")) {
        const reqId = action.replace("APPROVE_", "");
        const task = pendingTasks.get(reqId);
        if (task) {
            await sendMessengerText(task.senderId, task.suggestedAnswer);
            await teleBot.editMessageText(`✅ Đã gửi [#${reqId}]`, { chat_id: TELEGRAM_ADMIN_ID, message_id: cb.message.message_id });
            pendingTasks.delete(reqId);
        }
    }
    teleBot.answerCallbackQuery(cb.id);
});

app.listen(PORT, () => console.log(`[${TEN_TRO_LY}] (Messenger) đang lắng nghe cổng ${PORT}`));
```

Ghi chú: bản trên chỉ có nút Duyệt, không có nút Tự viết như bản Zalo, để giữ ví dụ gọn. Có thể thêm logic tương tự bản Zalo nếu cần.

## Chạy cùng đường hầm

Chạy `node bot-messenger.js` và `cloudflared tunnel --url http://localhost:3000` cùng lúc (hai tiến trình riêng, đều nên đưa vào PM2, xem `docs/09-van-hanh-va-xu-ly-su-co.md`).

## Giới hạn cần biết

Với App loại Business chưa qua xét duyệt của Meta, bot chỉ nhắn được với người dùng đã tự nhắn cho Trang trước, và có thể giới hạn số người dùng thử nghiệm. Muốn mở rộng công khai cho mọi người, cần gửi App để Meta xét duyệt quyền `pages_messaging`.
