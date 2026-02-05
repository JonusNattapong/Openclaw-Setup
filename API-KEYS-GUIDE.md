# คู่มือการหา API Keys และ Tokens สำหรับ Messaging Platforms

เอกสารนี้แนะนำวิธีการหา API keys หรือ tokens สำหรับการตั้งค่า channels ต่างๆ ใน OpenClaw อย่างปลอดภัย

## คำแนะนำทั่วไปสำหรับความปลอดภัย

- **เก็บ tokens เป็นความลับ:** อย่าบาง keys เหล่านี้กับผู้อื่นหรือ commit เข้า git
- **ใช้ environment variables:** เก็บ tokens ใน env vars แทนการ hardcode ใน config files
- **หมุนเวียน tokens:** สร้างใหม่เป็นประจำและ revoke เก่า
- **เปิดใช้งาน 2FA:** บนบัญชี developer ที่เกี่ยวข้อง
- **ใช้ secret management:** พิจารณาใช้ tools เช่น HashiCorp Vault, AWS Secrets Manager, หรือ 1Password

## Telegram

### การสร้าง Bot Token

1. เปิด Telegram และค้นหา **@BotFather** ([ลิงก์ตรง](https://t.me/BotFather))
2. ตรวจสอบว่า username ถูกต้องคือ `@BotFather` (มีเครื่องหมาย ✅)
3. ส่งข้อความ `/newbot` และทำตามคำแนะนำ:
   - ป้อนชื่อ bot (เช่น "My OpenClaw Assistant")
   - ป้อน username ที่ลงท้ายด้วย `bot` (เช่น `myopenclawbot`) - ต้อง unique
4. BotFather จะส่ง HTTP API token กลับมา (รูปแบบ: `123456789:AAAbcDefGHiJkLmNopQrStUvWxYz`)
5. คัดลอก token นี้และเก็บไว้ปลอดภัย

### การตั้งค่าเพิ่มเติม (แนะนำ)

หลังสร้าง bot แล้ว สามารถตั้งค่าเพิ่มเติมได้:

- `/setdescription` - ตั้งคำอธิบาย bot
- `/setuserpic` - ตั้งรูปโปรไฟล์
- `/setcommands` - ตั้งคำสั่งที่แสดงใน menu
- `/setjoingroups` - อนุญาต/ไม่อนุญาตให้ bot เข้าร่วมกลุ่ม
- `/setprivacy` - ควบคุมการเห็นข้อความในกลุ่ม (Privacy Mode)

### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "YOUR_BOT_TOKEN_HERE",
      "dmPolicy": "pairing"
    }
  }
}
```

หรือใช้ environment variable:
```bash
export TELEGRAM_BOT_TOKEN="your_token_here"
```

## Discord

### การสร้าง Bot และ Token

1. ไปที่ [Discord Developer Portal](https://discord.com/developers/applications)
2. คลิก **New Application** และป้อนชื่อ (เช่น "OpenClaw Bot")
3. ในแท็บ **Bot** คลิก **Add Bot** เพื่อสร้าง bot user
4. ในส่วน **Token** คลิก **Reset Token** ถ้าจำเป็น และ **Copy** เพื่อคัดลอก bot token
5. ใน **Privileged Gateway Intents** เปิดใช้งาน:
   - **Message Content Intent** (จำเป็นสำหรับอ่านเนื้อหาข้อความ)
   - **Server Members Intent** (สำหรับค้นหาสมาชิกและ allowlists)

### การ Invite Bot เข้า Server

1. ในแท็บ **OAuth2** → **URL Generator**
2. เลือก scopes: `bot`, `applications.commands`
3. เลือก bot permissions:
   - **Send Messages**
   - **Use Slash Commands**
   - **Read Message History**
   - **Read Messages/View Channels**
   - **Attach Files** (ถ้าต้องการส่งไฟล์)
4. คัดลอก URL ที่สร้างและเปิดใน browser
5. เลือก server ที่ต้องการ invite และ authorize

### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN_HERE"
    }
  }
}
```

หรือใช้ environment variable:
```bash
export DISCORD_BOT_TOKEN="your_token_here"
```

**คำเตือน:** เก็บ token เป็นความลับ - ถ้าหลุด สามารถถูกใช้เพื่อควบคุม bot และเข้าถึง server ได้

## WhatsApp

WhatsApp ไม่ใช้ API key แบบ traditional แต่ใช้ระบบ QR code login แทน

### ขั้นตอนการตั้งค่า

1. **เตรียมเบอร์โทรศัพท์:**
   - **แนะนำ:** ใช้เบอร์แยกต่างหาก (เช่น eSIM หรือเบอร์สำรอง)
   - **หรือ:** ใช้เบอร์ส่วนตัว (แต่ต้องเปิดใช้งาน self-chat mode)

2. **รันคำสั่ง login:**
   ```bash
   openclaw channels login
   ```

3. **สแกน QR code:**
   - เปิด WhatsApp บนโทรศัพท์
   - ไป Settings → Linked Devices → Link a Device
   - สแกน QR code ที่แสดงใน terminal

4. **ยืนยันการเชื่อมต่อ:**
   - WhatsApp จะแสดง "OpenClaw" เป็น device ที่เชื่อมต่อ
   - Credentials จะถูกเก็บใน `~/.openclaw/credentials/whatsapp/`

### การตั้งค่าเพิ่มเติม

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+66812345678"],
      "selfChatMode": false
    }
  }
}
```

**selfChatMode:** ตั้งเป็น `true` ถ้าใช้เบอร์ส่วนตัว

### ความปลอดภัย

- ใช้เบอร์แยกเพื่อแยกการใช้งานส่วนตัวและ bot
- ตรวจสอบ Linked Devices เป็นประจำ
- ออกจาก device ที่ไม่ใช้แล้ว

## Slack

### การสร้าง App และ Tokens

1. ไปที่ [Slack API](https://api.slack.com/apps) และคลิก **Create New App** → **From scratch**
2. ป้อนชื่อ app (เช่น "OpenClaw") และเลือก workspace

3. **เปิดใช้งาน Socket Mode:**
   - ในแท็บ **Socket Mode** toggle เป็น on

4. **สร้าง App-Level Token:**
   - ไป **Basic Information** → **App-Level Tokens**
   - คลิก **Generate Token and Scopes**
   - ป้อนชื่อ (เช่น "OpenClaw Socket Token")
   - เลือก scope: `connections:write`
   - คัดลอก token (ขึ้นต้นด้วย `xapp-`)

5. **สร้าง Bot Token:**
   - ในแท็บ **OAuth & Permissions**
   - เพิ่ม Bot Token Scopes:
     - `channels:history`, `channels:read`, `chat:write`
     - `groups:history`, `groups:read`
     - `im:history`, `im:read`, `mpim:history`, `mpim:read`
     - `users:read`, `reactions:read`, `files:read`
   - คลิก **Install to Workspace**
   - คัดลอก **Bot User OAuth Token** (ขึ้นต้นด้วย `xoxb-`)

6. **ตั้งค่า Event Subscriptions:**
   - เปิดใช้งาน Events
   - Subscribe to bot events:
     - `app_mention`
     - `message.channels`, `message.groups`, `message.im`, `message.mpim`
     - `reaction_added`, `reaction_removed`

7. **Invite bot เข้า channels:**
   - Invite bot เข้า channels ที่ต้องการใช้งาน
   - หรือตั้งค่า `/openclaw` slash command

### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "appToken": "xapp-your-app-token",
      "botToken": "xoxb-your-bot-token"
    }
  }
}
```

หรือใช้ environment variables:
```bash
export SLACK_APP_TOKEN="xapp-..."
export SLACK_BOT_TOKEN="xoxb-..."
```

## Mattermost

### การสร้าง Bot Account

1. เข้าสู่ Mattermost server ของคุณในฐานะ administrator
2. ไป **System Console** → **Integrations** → **Bot Accounts**
3. คลิก **Add Bot Account**
4. ป้อนข้อมูล:
   - **Username:** (เช่น `openclaw-bot`) - ต้อง unique
   - **Display Name:** (เช่น "OpenClaw Assistant")
   - **Description:** คำอธิบาย bot
   - **Roles:** เลือก roles ที่เหมาะสม
5. คลิก **Create Bot Account**
6. คัดลอก **Bot Access Token** ที่แสดง (จะแสดงครั้งเดียว - เก็บไว้ให้ปลอดภัย)

### การตั้งค่าเพิ่มเติม

- จด **Base URL** ของ Mattermost server (เช่น `https://chat.company.com`)
- ตั้งค่า permissions เพิ่มเติมถ้าจำเป็น

### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "mattermost": {
      "enabled": true,
      "botToken": "bot-access-token-here",
      "baseUrl": "https://your-mattermost-server.com",
      "dmPolicy": "pairing"
    }
  }
}
```

หรือใช้ environment variables:
```bash
export MATTERMOST_BOT_TOKEN="your_token"
export MATTERMOST_URL="https://your-server.com"
```

## Platforms อื่นๆ

### Signal

คล้าย WhatsApp - ใช้เบอร์โทรศัพท์และ QR code:

```bash
openclaw channels login
```

### iMessage

สำหรับ macOS เท่านั้น - ไม่ต้อง token เพิ่มเติม แต่ต้องตั้งค่า permissions ใน System Preferences

### LINE, Zalo, WeChat, etc.

ดูเอกสารเฉพาะใน [channels documentation](/channels/)

## การจัดการ Tokens แบบปลอดภัย

### เก็บใน Environment Variables

```bash
# สร้างไฟล์ .env (อย่า commit)
echo "TELEGRAM_BOT_TOKEN=your_token" > .env
echo "DISCORD_BOT_TOKEN=your_token" >> .env

# โหลด env vars
source .env
```

### ใช้ Secret Management Tools

- **1Password:** เก็บ tokens เป็น secure notes
- **HashiCorp Vault:** สำหรับ enterprise
- **AWS Secrets Manager:** สำหรับ AWS environments
- **Azure Key Vault:** สำหรับ Azure

### การหมุนเวียน Tokens

1. สร้าง token ใหม่ใน platform
2. อัปเดตใน OpenClaw config
3. ทดสอบการทำงาน
4. Revoke token เก่า

### การตรวจสอบการใช้งาน

- ตรวจสอบ bot activity logs ในแต่ละ platform
- Monitor สำหรับ suspicious activity
- ออกจาก devices/sessions ที่ไม่รู้จัก

## Troubleshooting

### Token ไม่ทำงาน

- ตรวจสอบว่า token ถูกต้องและไม่หมดอายุ
- ตรวจสอบ permissions/scopes ที่ตั้ง
- ตรวจสอบว่า bot ถูก invite เข้า channels

### Connection Issues

- ตรวจสอบ network connectivity
- ตรวจสอบ firewall settings
- ตรวจสอบว่า platform services ทำงานปกติ

### Security Concerns

- ถ้าสงสัยว่า token หลุด: revoke และสร้างใหม่ทันที
- ตรวจสอบ access logs ของ platform
- เปลี่ยน passwords ที่เกี่ยวข้อง

## อ้างอิง

- [Telegram Bot API](https://core.telegram.org/bots)
- [Discord Developer Portal](https://discord.com/developers/docs)
- [Slack API](https://api.slack.com/)
- [Mattermost Integrations](https://docs.mattermost.com/developer/bot-accounts.html)
- [OpenClaw Channels Documentation](/channels/)</content>
<parameter name="filePath">d:\Projects\Github\Openclaw-Setup\API-KEYS-GUIDE.md