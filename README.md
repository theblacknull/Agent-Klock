# Agent Klock

Bot Telegram cho [Klock](https://github.com/theblacknull/Klock) — mã hóa khóa thời gian
(time-lock encryption): nội dung chỉ giải mã được sau 1 thời điểm do người gửi chọn.

Dùng lại **đúng** công thức mã hóa với web Klock — AES-256-GCM + Argon2id (key derivation)
+ [tlock-js](https://www.npmjs.com/package/tlock-js) (time-lock qua mạng drand), cùng định
dạng envelope text/file — nên 1 link/file tạo từ bot có thể giải mã trên web, và ngược lại.

## Tính năng

- Mã hóa văn bản hoặc file/ảnh (tối đa ~19MB, giới hạn của Telegram Bot API) với mật khẩu
  bí mật + thời điểm mở khóa.
- Chọn thời điểm mở khóa nhanh qua nút bấm (+10 phút … +1 tháng) hoặc nhập tay ngày giờ cụ thể.
- Giải mã từ link hoặc từ file `.klock.json` gửi trực tiếp (không cần server nếu dùng file).
- Mật khẩu bí mật không bao giờ được lưu đĩa; tin nhắn chứa mật khẩu tự xóa khỏi lịch sử
  chat ngay sau khi bot đọc xong, để giảm rủi ro lộ mật khẩu nếu xem lại lịch sử chat.
- Đa ngôn ngữ: EN / RU / VI / ZH / ES (`/lang` để đổi, giống bộ ngôn ngữ của web).

## Chạy

Cần có server [klock-web](https://github.com/theblacknull/Klock) đang chạy (local hoặc
production) để lưu slug/link — bot chỉ gọi API REST của server đó, không tự lưu trữ.

```bash
npm install
cp .env.example .env   # điền API_KEY_BOT (token từ @BotFather), API_BASE, PUBLIC_BASE
npm start
```

### Lưu ý về `hash-wasm`

`package.json` của `hash-wasm` không khai báo `exports`, nên `require("hash-wasm")` (và cả
`import` ESM thông thường) sẽ dùng bản UMD, cho ra kết quả Argon2id **khác** với bản ESM
(`dist/index.esm.js`) mà web Klock dùng — dù cùng version/tham số. `bot.js` đã nạp đúng file
ESM này để khóa suy ra khớp 100% với phía trình duyệt; nếu sửa code, giữ nguyên cách nạp đó.

## Triển khai production

Khuyến nghị chạy cùng máy với `klock-web` (để `API_BASE=http://127.0.0.1:8787`), quản lý
bằng systemd, ví dụ:

```ini
[Unit]
Description=Agent Klock Telegram bot
After=network.target

[Service]
Type=simple
WorkingDirectory=/path/to/Agent-Klock
ExecStart=/usr/bin/node bot.js
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
