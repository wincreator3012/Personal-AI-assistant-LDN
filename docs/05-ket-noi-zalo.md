# Kết nối Zalo

Zalo không có API chính thức cho việc này. Cách khả thi hiện nay là dùng thư viện không chính thức `zca-js`, mô phỏng trình duyệt để điều khiển một tài khoản Zalo cá nhân.

> Cảnh báo bắt buộc đọc: `zca-js` tự cảnh báo rằng dùng thư viện này có thể khiến tài khoản bị khóa hoặc cấm, đây là rủi ro cố hữu chứ không phải lỗi thao tác. LUÔN dùng một tài khoản Zalo phụ, không dùng tài khoản chính của người dùng.

Vì đây là thư viện cộng đồng duy trì, không phải API chính thức từ Zalo, nó có thể ngừng hoạt động bất cứ lúc nào nếu Zalo thay đổi cơ chế bảo mật. Trước khi triển khai chính thức cho một cộng đồng lớn, kiểm tra trang GitHub của dự án (RFS-ADRENO/zca-js) xem còn được cập nhật gần đây không, và cân nhắc Telegram làm phương án dự phòng nếu Zalo là kênh duy nhất đang dùng.

## Thiết lập

```bash
mkdir -p ~/Desktop/ten-du-an/kenh-zalo
cd ~/Desktop/ten-du-an/kenh-zalo
npm init -y
npm install zca-js axios node-telegram-bot-api
```

Đăng nhập và lưu phiên:

```bash
nano login.js
```

```javascript
const { Zalo } = require("zca-js");
const fs = require("fs");

const zalo = new Zalo();
zalo.loginQR().then((api) => {
    const appContext = api.getContext();
    fs.writeFileSync("appContext.json", JSON.stringify(appContext, null, 2));
    console.log("Đăng nhập thành công, đã lưu phiên.");
    process.exit(0);
}).catch((err) => {
    console.error("Lỗi đăng nhập:", err);
    process.exit(1);
});
```

Chạy `node login.js`, quét mã QR bằng tài khoản Zalo phụ trên điện thoại (bước này bắt buộc người dùng tự làm, không thể tự động hóa vì cần quét vật lý).

## Code điều phối (bot-zalo.js)

```javascript
const { Zalo } = require("zca-js");
const TelegramBot = require("node-telegram-bot-api");
const axios = require("axios");
const fs = require("fs");

// ==================== CẤU HÌNH ====================
const DIFY_API_URL = "http://localhost/v1/chat-messages";
const DIFY_API_KEY = "ĐIỀN_MÃ_APP_API_DIFY";
const TELEGRAM_TOKEN = "ĐIỀN_TOKEN_BOT_TELEGRAM";
const TELEGRAM_ADMIN_ID = "ĐIỀN_TELEGRAM_ID_CỦA_BẠN";
const TEN_TRO_LY = "ĐIỀN_TÊN_TRỢ_LÝ";
const TU_KHOA_KICH_HOAT = /^(ĐIỀN_TỪ_KÍCH_HOẠT_1|ĐIỀN_TỪ_KÍCH_HOẠT_2|!bot)/i;
const MA_BAO_DONG = "ĐIỀN_MÃ_BÁO_ĐỘNG"; // phải khớp System Prompt
const TIEN_TO_YEU_CAU = "ZL"; // dùng khi chia sẻ chung 1 bot Telegram với kênh khác
// =====================================================

const teleBot = new TelegramBot(TELEGRAM_TOKEN, { polling: true });
const appContext = JSON.parse(fs.readFileSync("appContext.json", "utf8"));
const zalo = new Zalo({ appContext });
const pendingTasks = new Map();
let requestCounter = 100;
let waitingCustomReplyForId = null;

zalo.login().then((api) => {
    console.log(`[${TEN_TRO_LY}] (Zalo) đã sẵn sàng.`);
    teleBot.sendMessage(TELEGRAM_ADMIN_ID, `🟢 *${TEN_TRO_LY}* (Zalo) đã kích hoạt.`, { parse_mode: "Markdown" });

    api.listener.on("message", async (msg) => {
        if (!msg.isGroup || !msg.data || !msg.data.content) return;
        const text = msg.data.content.trim();
        const mentions = msg.data.mentions || [];
        const isMentioned = mentions.some(m => m.uid === api.getContext().uid);
        const hasTriggerWord = TU_KHOA_KICH_HOAT.test(text);
        if (!isMentioned && !hasTriggerWord) return;

        const query = text.replace(TU_KHOA_KICH_HOAT, "").trim();
        const senderName = msg.data.dName || "Bạn";
        if (!query) {
            await api.sendMessage({ msg: `Chào bạn, mình là ${TEN_TRO_LY}. Bạn muốn hỏi mình điều gì?`, quote: msg }, msg.threadId);
            return;
        }

        try {
            const res = await axios.post(DIFY_API_URL, {
                inputs: {}, query, response_mode: "blocking", user: String(msg.data.uidFrom)
            }, {
                headers: { Authorization: `Bearer ${DIFY_API_KEY}`, "Content-Type": "application/json" },
                timeout: 60000
            });

            let answer = res.data.answer.trim();
            if (answer.includes(`[${MA_BAO_DONG}]`)) {
                requestCounter++;
                const reqId = `${TIEN_TO_YEU_CAU}-${requestCounter}`;
                const suggestedAnswer = answer.replace(`[${MA_BAO_DONG}]`, "").trim();
                pendingTasks.set(reqId, { originalMsg: msg, threadId: msg.threadId, senderName, question: query, suggestedAnswer });

                await api.sendMessage({
                    msg: `Chào ${senderName}, câu hỏi này cần thêm thời gian để trả lời cho thấu đáo. Mình đã chuyển đến quản trị viên, bạn sẽ sớm nhận phản hồi nhé!`,
                    quote: msg
                }, msg.threadId);

                await teleBot.sendMessage(TELEGRAM_ADMIN_ID,
                    `⚠️ *CẦN DUYỆT [#${reqId}]* (kênh Zalo)\n👤 ${senderName}\n❓ "${query}"\n\n🌱 _${suggestedAnswer || "Chưa có gợi ý"}_`,
                    { parse_mode: "Markdown", reply_markup: { inline_keyboard: [
                        [{ text: "✅ Duyệt gửi", callback_data: `APPROVE_${reqId}` }],
                        [{ text: "✍️ Tự viết", callback_data: `CUSTOM_${reqId}` }]
                    ]}});
            } else {
                await api.sendMessage({ msg: `[${TEN_TRO_LY}]\n\n${answer}`, quote: msg }, msg.threadId);
            }
        } catch (error) {
            console.error("Lỗi kết nối Dify:", error.message);
            await api.sendMessage({ msg: "Hiện tại nhịp kết nối tri thức đang chậm, bạn chờ mình chút nhé.", quote: msg }, msg.threadId);
        }
    });

    api.listener.start();

    teleBot.on("callback_query", async (cb) => {
        const action = cb.data;
        const msgId = cb.message.message_id;
        if (action.startsWith("APPROVE_")) {
            const reqId = action.replace("APPROVE_", "");
            const task = pendingTasks.get(reqId);
            if (task) {
                await api.sendMessage({ msg: `[${TEN_TRO_LY}]\n\n${task.suggestedAnswer}`, quote: task.originalMsg }, task.threadId);
                await teleBot.editMessageText(`✅ Đã gửi [#${reqId}]`, { chat_id: TELEGRAM_ADMIN_ID, message_id: msgId });
                pendingTasks.delete(reqId);
            }
        } else if (action.startsWith("CUSTOM_")) {
            waitingCustomReplyForId = action.replace("CUSTOM_", "");
            await teleBot.sendMessage(TELEGRAM_ADMIN_ID, `✍️ Gõ nội dung cho [#${waitingCustomReplyForId}]:`);
        }
        teleBot.answerCallbackQuery(cb.id);
    });

    teleBot.on("message", async (teleMsg) => {
        if (!waitingCustomReplyForId || teleMsg.text.startsWith("/")) return;
        const task = pendingTasks.get(waitingCustomReplyForId);
        if (task) {
            await api.sendMessage({ msg: `Từ chia sẻ của quản trị viên:\n\n${teleMsg.text.trim()}`, quote: task.originalMsg }, task.threadId);
            pendingTasks.delete(waitingCustomReplyForId);
            waitingCustomReplyForId = null;
        }
    });
}).catch((err) => console.error("Lỗi đăng nhập Zalo:", err));
```

Chạy liên tục bằng PM2, xem `docs/09-van-hanh-va-xu-ly-su-co.md`.
