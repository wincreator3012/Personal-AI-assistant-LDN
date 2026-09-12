# Kết nối Viber

## Thực tế cần biết trước khi làm

Từ ngày 05/02/2024, Viber không còn cho tự do tạo bot miễn phí như trước. Việc tạo bot hiện đi theo diện thương mại: phải nộp đơn trực tiếp cho Rakuten Viber hoặc qua một đối tác chính thức được xác nhận, và trả phí duy trì hàng tháng 115 euro cho mỗi bot (đã kiểm chứng tại thời điểm biên soạn tài liệu này, tháng 9/2026, xem trang chính sách chính thức của Viber Support để cập nhật nếu con số này đã thay đổi). Tin nhắn trong phiên hội thoại không tính thêm phí và không giới hạn số lượng, nhưng tin nhắn do bot chủ động gửi ngoài phiên hội thoại tính phí riêng theo bảng giá của Viber. Vì vậy Viber chỉ hợp lý cho tổ chức có mục đích thương mại rõ ràng và ngân sách vận hành, không hợp cho dự án cá nhân hay cộng đồng phi lợi nhuận quy mô nhỏ.

Nếu không chắc có nên đầu tư vào kênh này, cân nhắc Zalo, Telegram, hoặc Messenger trước.

## Nếu đã có bot Viber (do đã qua xét duyệt)

Kiến trúc kỹ thuật tương tự Messenger: hoạt động qua webhook, Viber chủ động gọi đến máy chủ của bạn, cần một URL HTTPS công khai với chứng chỉ SSL hợp lệ từ một tổ chức chứng thực đáng tin cậy, không chấp nhận chứng chỉ tự ký. Cloudflare Tunnel (xem `docs/07-ket-noi-messenger.md`) đáp ứng được yêu cầu này vì đi kèm chứng chỉ hợp lệ.

### Thiết lập webhook

```bash
curl -X POST https://chatapi.viber.com/pa/set_webhook \
  -H "Content-Type: application/json" \
  -H "X-Viber-Auth-Token: ĐIỀN_AUTH_TOKEN_VIBER" \
  -d '{"url": "https://ten-mien-cong-khai-cua-ban/webhook"}'
```

### Thiết lập dự án

```bash
mkdir -p ~/Desktop/ten-du-an/kenh-viber
cd ~/Desktop/ten-du-an/kenh-viber
npm init -y
npm install express axios node-telegram-bot-api
```

### Code điều phối (bot-viber.js)

```javascript
const express = require("express");
const axios = require("axios");
const TelegramBot = require("node-telegram-bot-api");

// ==================== CẤU HÌNH ====================
const PORT = 3001;
const VIBER_AUTH_TOKEN = "ĐIỀN_AUTH_TOKEN_VIBER";
const DIFY_API_URL = "http://localhost/v1/chat-messages";
const DIFY_API_KEY = "ĐIỀN_MÃ_APP_API_DIFY";
const TELEGRAM_TOKEN = "ĐIỀN_TOKEN_BOT_TELEGRAM";
const TELEGRAM_ADMIN_ID = "ĐIỀN_TELEGRAM_ID_CỦA_BẠN";
const TEN_TRO_LY = "ĐIỀN_TÊN_TRỢ_LÝ";
const MA_BAO_DONG = "ĐIỀN_MÃ_BÁO_ĐỘNG";
const TIEN_TO_YEU_CAU = "VB";
// =====================================================

const app = express();
app.use(express.json());
const teleBot = new TelegramBot(TELEGRAM_TOKEN, { polling: false });
const pendingTasks = new Map();
let requestCounter = 100;

app.post("/webhook", async (req, res) => {
    res.status(200).send("{}"); // Viber yêu cầu phản hồi 200 nhanh
    const event = req.body;
    if (event.event !== "message" || !event.message || !event.message.text) return;

    const userId = event.sender.id;
    const query = event.message.text.trim();

    try {
        const dify = await axios.post(DIFY_API_URL, {
            inputs: {}, query, response_mode: "blocking", user: userId
        }, {
            headers: { Authorization: `Bearer ${DIFY_API_KEY}`, "Content-Type": "application/json" },
            timeout: 60000
        });
        let answer = dify.data.answer.trim();

        if (answer.includes(`[${MA_BAO_DONG}]`)) {
            requestCounter++;
            const reqId = `${TIEN_TO_YEU_CAU}-${requestCounter}`;
            const suggestedAnswer = answer.replace(`[${MA_BAO_DONG}]`, "").trim();
            pendingTasks.set(reqId, { userId, query, suggestedAnswer });

            await sendViberText(userId, "Câu hỏi này cần thêm thời gian để trả lời cho thấu đáo, mình đã chuyển đến quản trị viên nhé!");
            await teleBot.sendMessage(TELEGRAM_ADMIN_ID,
                `⚠️ *CẦN DUYỆT [#${reqId}]* (kênh Viber)\n❓ "${query}"\n\n🌱 _${suggestedAnswer}_`,
                { parse_mode: "Markdown", reply_markup: { inline_keyboard: [
                    [{ text: "✅ Duyệt gửi", callback_data: `APPROVE_${reqId}` }]
                ]}});
        } else {
            await sendViberText(userId, answer);
        }
    } catch (err) {
        console.error("Lỗi kết nối Dify:", err.message);
        await sendViberText(userId, "Hiện tại nhịp kết nối tri thức đang chậm, bạn chờ mình chút nhé.");
    }
});

async function sendViberText(receiverId, text) {
    await axios.post("https://chatapi.viber.com/pa/send_message", {
        receiver: receiverId,
        type: "text",
        text
    }, { headers: { "X-Viber-Auth-Token": VIBER_AUTH_TOKEN } });
}

teleBot.on("callback_query", async (cb) => {
    const action = cb.data;
    if (action.startsWith("APPROVE_")) {
        const reqId = action.replace("APPROVE_", "");
        const task = pendingTasks.get(reqId);
        if (task) {
            await sendViberText(task.userId, task.suggestedAnswer);
            await teleBot.editMessageText(`✅ Đã gửi [#${reqId}]`, { chat_id: TELEGRAM_ADMIN_ID, message_id: cb.message.message_id });
            pendingTasks.delete(reqId);
        }
    }
    teleBot.answerCallbackQuery(cb.id);
});

app.listen(PORT, () => console.log(`[${TEN_TRO_LY}] (Viber) đang lắng nghe cổng ${PORT}`));
```

Ghi chú: Viber chỉ gửi được tin nhắn chủ động cho người dùng đã "subscribe" (đã tự nhắn cho bot trước ít nhất một lần), đúng nguyên lý tương tự Messenger.
