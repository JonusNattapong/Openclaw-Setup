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

**ขั้นตอนละเอียด:**
1. ติดตั้ง Signal บนโทรศัพท์และ desktop (ถ้าต้องการ)
2. รันคำสั่ง login ใน OpenClaw
3. สแกน QR code ด้วย Signal บนโทรศัพท์
4. ยืนยันการเชื่อมต่อ

**การตั้งค่าเพิ่มเติม:**
```json
{
  "channels": {
    "signal": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+66812345678"],
      "selfChatMode": false
    }
  }
}
```

### iMessage

สำหรับ macOS เท่านั้น - ไม่ต้อง token เพิ่มเติม แต่ต้องตั้งค่า permissions ใน System Preferences

**ขั้นตอนการตั้งค่า:**
1. เปิด **System Preferences** → **Security & Privacy** → **Privacy** → **Full Disk Access**
2. เพิ่ม OpenClaw หรือ Terminal เข้าไป
3. เปิด **Messages** → **Preferences** → **iMessage** → เปิดใช้งาน
4. รัน `openclaw channels login` เพื่อ sync

### LINE

#### การสร้าง Channel Access Token

1. ไปที่ [LINE Developers Console](https://developers.line.biz/console/)
2. สร้าง **Provider** ใหม่ (ถ้ายังไม่มี)
3. คลิก **Create a new channel** → **Messaging API**
4. ป้อนข้อมูล:
   - **Channel name:** ชื่อ channel
   - **Channel description:** คำอธิบาย
   - **Category:** เลือกที่เหมาะสม
5. ในแท็บ **Messaging API** → **Channel access token**
6. คลิก **Issue** เพื่อสร้าง token
7. คัดลอก **Channel Access Token** (รูปแบบยาว)

#### การตั้งค่า Webhook

1. ใน **Webhook URL** ป้อน URL ของ OpenClaw server
2. เปิดใช้งาน **Use webhook**
3. คัดลอก **Channel Secret** จากแท็บ Basic settings

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "line": {
      "enabled": true,
      "channelAccessToken": "YOUR_CHANNEL_ACCESS_TOKEN",
      "channelSecret": "YOUR_CHANNEL_SECRET",
      "dmPolicy": "pairing"
    }
  }
}
```

### Facebook Messenger

#### การสร้าง Facebook App

1. ไปที่ [Facebook Developers](https://developers.facebook.com/)
2. คลิก **My Apps** → **Create App**
3. เลือก **Business** → **Yourself or your own business**
4. ป้อนชื่อ app และ contact email
5. ใน **Add a Product** ค้นหา **Messenger**
6. คลิก **Set Up** ใน Messenger

#### การตั้งค่า Webhooks

1. ใน Messenger → **Settings** → **Webhooks**
2. คลิก **Add Callback URL**
3. ป้อน URL ของ OpenClaw server และ verify token
4. เลือก events: `messages`, `messaging_postbacks`, `messaging_optins`
5. คลิก **Verify and Save**

#### การสร้าง Page Access Token

1. ไป **Messenger** → **Settings** → **Access Tokens**
2. เลือก Facebook Page ที่ต้องการ
3. คลิก **Generate Token**
4. คัดลอก token ที่ได้

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "facebook": {
      "enabled": true,
      "pageAccessToken": "YOUR_PAGE_ACCESS_TOKEN",
      "verifyToken": "YOUR_VERIFY_TOKEN",
      "appSecret": "YOUR_APP_SECRET"
    }
  }
}
```

### Instagram

#### การตั้งค่า Instagram Basic Display API

1. ไป Facebook Developers Console (app เดียวกับ Facebook Messenger)
2. ใน **Add a Product** ค้นหา **Instagram Basic Display**
3. คลิก **Set Up**
4. ใน **Instagram Basic Display** → **Create New App**
5. เลือก Instagram account ที่ต้องการเชื่อมต่อ
6. คัดลอก **App ID** และ **App Secret**

#### การสร้าง Access Token

1. ไป **Instagram Basic Display** → **User Token Generator**
2. คลิก **Generate Token**
3. Authorize app
4. คัดลอก access token

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "instagram": {
      "enabled": true,
      "accessToken": "YOUR_INSTAGRAM_ACCESS_TOKEN",
      "appSecret": "YOUR_APP_SECRET"
    }
  }
}
```

### Twitter/X

#### การสร้าง Twitter App

1. ไป [Twitter Developer Portal](https://developer.twitter.com/)
2. สมัคร developer account (ต้อง verify identity)
3. สร้าง project และ app ใหม่
4. ใน **App permissions** เลือก **Read and Write**
5. ใน **Authentication settings** เปิดใช้งาน OAuth 2.0

#### การสร้าง API Keys

1. ใน **Keys and tokens** คัดลอก:
   - **API Key** (Consumer Key)
   - **API Key Secret** (Consumer Secret)
   - **Bearer Token**
2. สร้าง **Access Token** และ **Access Token Secret**

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "twitter": {
      "enabled": true,
      "consumerKey": "YOUR_CONSUMER_KEY",
      "consumerSecret": "YOUR_CONSUMER_SECRET",
      "accessToken": "YOUR_ACCESS_TOKEN",
      "accessTokenSecret": "YOUR_ACCESS_TOKEN_SECRET"
    }
  }
}
```

### Zalo

#### การสร้าง Zalo App

1. ไป [Zalo Developers](https://developers.zalo.me/)
2. สมัคร developer account
3. สร้าง app ใหม่ → **Official Account App**
4. ป้อนข้อมูล app และ verify

#### การตั้งค่า Webhook

1. ใน **Webhook** ป้อน URL ของ OpenClaw server
2. คัดลอก **App Secret** และ **Access Token**

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "zalo": {
      "enabled": true,
      "accessToken": "YOUR_ZALO_ACCESS_TOKEN",
      "appSecret": "YOUR_APP_SECRET"
    }
  }
}
```

### Microsoft Teams

#### การสร้าง Azure App

1. ไป [Azure Portal](https://portal.azure.com/)
2. สร้าง **App Registration** ใหม่
3. ป้อนชื่อ app และตั้งค่า
4. ใน **Certificates & secrets** สร้าง **Client secret**
5. คัดลอก **Application (client) ID** และ **Client secret**

#### การตั้งค่า Teams App

1. ไป [Teams Developer Portal](https://dev.teams.microsoft.com/)
2. สร้าง app manifest
3. ตั้งค่า bot และ messaging endpoints
4. Upload manifest ไป Teams

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "msteams": {
      "enabled": true,
      "appId": "YOUR_APP_ID",
      "appPassword": "YOUR_CLIENT_SECRET",
      "tenantId": "YOUR_TENANT_ID"
    }
  }
}
```

### Google Chat

#### การสร้าง Google Cloud Project

1. ไป [Google Cloud Console](https://console.cloud.google.com/)
2. สร้าง project ใหม่
3. เปิดใช้งาน **Hangouts Chat API**
4. สร้าง **Service Account**
5. สร้าง **Private Key** (JSON format)

#### การตั้งค่า Webhook

1. ใน Google Chat API → **Configuration**
2. ตั้งค่า webhook URL
3. คัดลอก credentials จาก service account

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "googlechat": {
      "enabled": true,
      "serviceAccountKey": "/path/to/service-account-key.json",
      "projectId": "your-project-id"
    }
  }
}
```

### Twitch

#### การสร้าง Twitch App

1. ไป [Twitch Developer Console](https://dev.twitch.tv/console/apps)
2. คลิก **Create Application**
3. ป้อนชื่อ app และ OAuth redirect URL
4. คัดลอก **Client ID** และ **Client Secret**

#### การสร้าง Access Token

1. ใช้ Client ID และ Secret เพื่อสร้าง access token:
```bash
curl -X POST 'https://id.twitch.tv/oauth2/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
-d 'client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&grant_type=client_credentials'
```

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "twitch": {
      "enabled": true,
      "clientId": "YOUR_CLIENT_ID",
      "clientSecret": "YOUR_CLIENT_SECRET",
      "accessToken": "YOUR_ACCESS_TOKEN"
    }
  }
}
```

### Matrix

#### การตั้งค่า Matrix Server

1. ติดตั้ง Matrix server (เช่น Synapse) หรือใช้ public server
2. สร้าง user account สำหรับ bot
3. เปิดใช้งาน appservice (ถ้าต้องการ bridge)

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserverUrl": "https://matrix.org",
      "userId": "@bot:matrix.org",
      "accessToken": "YOUR_ACCESS_TOKEN"
    }
  }
}
```

### Nostr

#### การสร้าง Nostr Keys

1. ใช้ Nostr client หรือ library เพื่อสร้าง keypair
2. คัดลอก **Private Key** (nsec) และ **Public Key** (npub)

#### การตั้งค่า Relays

1. เลือก relays ที่เชื่อถือได้ (เช่น `wss://relay.damus.io`)
2. ตั้งค่าใน OpenClaw

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "nostr": {
      "enabled": true,
      "privateKey": "YOUR_PRIVATE_KEY",
      "relays": ["wss://relay.damus.io", "wss://relay.snort.social"]
    }
  }
}
```

### Tlon (Urbit)

#### การตั้งค่า Urbit Ship

1. ซื้อหรือ lease Urbit ship
2. ติดตั้งและ boot ship
3. ตั้งค่า networking และ DNS

#### การใช้งานใน OpenClaw

```json
{
  "channels": {
    "tlon": {
      "enabled": true,
      "shipName": "your-ship-name",
      "shipUrl": "https://your-ship-url.com",
      "code": "your-access-code"
    }
  }
}
```

## Best Practices สำหรับแต่ละ Platform

### Telegram
- ใช้ bot แยกต่างหากสำหรับแต่ละ use case
- ตั้งค่า `/setcommands` เพื่อให้ user เข้าใจคำสั่ง
- เปิดใช้งาน group privacy mode เพื่อความปลอดภัย
- ใช้ webhooks แทน polling สำหรับ high-traffic bots

### Discord
- ใช้ bot permissions เฉพาะที่จำเป็น (Principle of Least Privilege)
- ตั้งค่า bot status และ activity
- ใช้ slash commands สำหรับ UX ที่ดี
- Implement proper error handling สำหรับ rate limits

### Slack
- ใช้ Socket Mode สำหรับ security ที่ดีขึ้น
- ตั้งค่า app home และ interactive components
- ใช้ block kit สำหรับ rich messages
- Monitor app usage ใน analytics dashboard

### WhatsApp
- ใช้ business account สำหรับ production
- Implement message templates สำหรับ marketing
- จัดการ webhooks อย่างมีประสิทธิภาพ
- ตรวจสอบ message delivery status

### Facebook Messenger
- ใช้ page access tokens แทน user tokens
- Implement webhook verification
- จัดการ conversation states
- ใช้ structured messages (generic templates, buttons)

### Instagram
- จัดการ access token refresh อัตโนมัติ
- ใช้ Instagram Basic Display API อย่างระมัดระวัง
- Monitor API usage limits
- จัดการ user consent flows

### Twitter/X
- Implement proper rate limiting
- ใช้ webhooks สำหรับ real-time updates
- จัดการ OAuth flows อย่างปลอดภัย
- Monitor API v2 migration requirements

### LINE
- ใช้ channel access tokens แทน long-lived tokens
- Implement rich menus และ LIFF apps
- จัดการ webhook signatures
- ใช้ multicast สำหรับส่งข้อความหลายคน

### Mattermost
- ใช้ bot accounts แทน user accounts
- ตั้งค่า proper roles และ permissions
- Implement slash commands
- จัดการ webhooks อย่างมีประสิทธิภาพ

## Advanced Configuration

### Environment Variables Setup

สร้างไฟล์ `.env` ที่ปลอดภัย:

```bash
# Telegram
TELEGRAM_BOT_TOKEN="YOUR_TELEGRAM_BOT_TOKEN_HERE"

# Discord
DISCORD_BOT_TOKEN="YOUR_DISCORD_BOT_TOKEN_HERE"

# Slack
SLACK_APP_TOKEN="YOUR_SLACK_APP_TOKEN_HERE"
SLACK_BOT_TOKEN="YOUR_SLACK_BOT_TOKEN_HERE"

# Facebook
FACEBOOK_PAGE_ACCESS_TOKEN="YOUR_FACEBOOK_PAGE_ACCESS_TOKEN_HERE"
FACEBOOK_VERIFY_TOKEN="YOUR_FACEBOOK_VERIFY_TOKEN_HERE"
FACEBOOK_APP_SECRET="YOUR_FACEBOOK_APP_SECRET_HERE"

# LINE
LINE_CHANNEL_ACCESS_TOKEN="YOUR_LINE_CHANNEL_ACCESS_TOKEN_HERE"
LINE_CHANNEL_SECRET="1234567890abcdef"

# Mattermost
MATTERMOST_BOT_TOKEN="bot_token_here"
MATTERMOST_URL="https://chat.company.com"

# และอื่นๆ...
```

โหลด environment variables:
```bash
# Linux/macOS
source .env

# Windows PowerShell
foreach ($line in Get-Content .env) {
    if ($line -match '^([^=]+)=(.*)$') {
        $key = $1
        $value = $2 -replace '^"|"$'
        [Environment]::SetEnvironmentVariable($key, $value, "Process")
    }
}
```

### Secret Management Integration

#### 1Password CLI
```bash
# เก็บ token ใน 1Password
op create document .env --title="OpenClaw Tokens"

# อ่าน token
export TELEGRAM_BOT_TOKEN=$(op read "op://Private/OpenClaw Tokens/TELEGRAM_BOT_TOKEN")
```

#### HashiCorp Vault
```bash
# เขียน secrets
vault kv put secret/openclaw/telegram bot_token="your_token"

# อ่าน secrets
export TELEGRAM_BOT_TOKEN=$(vault kv get -field=bot_token secret/openclaw/telegram)
```

#### AWS Secrets Manager
```bash
# เก็บ secrets
aws secretsmanager create-secret --name openclaw/tokens --secret-string '{"telegram_bot_token":"your_token"}'

# อ่าน secrets
TOKEN=$(aws secretsmanager get-secret-value --secret-id openclaw/tokens --query SecretString --output text | jq -r .telegram_bot_token)
```

### Token Rotation Automation

สร้าง script สำหรับหมุนเวียน tokens:

```bash
#!/bin/bash
# token-rotation.sh

# Backup current tokens
cp .env .env.backup.$(date +%Y%m%d_%H%M%S)

# Rotate Telegram token
NEW_TELEGRAM_TOKEN=$(curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getMe" | jq -r '.result.id')
# ... logic to get new token from platform APIs

# Update .env file
sed -i "s/TELEGRAM_BOT_TOKEN=.*/TELEGRAM_BOT_TOKEN=${NEW_TELEGRAM_TOKEN}/" .env

# Reload environment
source .env

# Restart OpenClaw
openclaw restart

echo "Tokens rotated successfully"
```

### Monitoring และ Alerting

ตั้งค่า monitoring สำหรับ token health:

```bash
# Health check script
#!/bin/bash
# health-check.sh

# Check Telegram bot
if curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getMe" | grep -q '"ok":true'; then
    echo "✅ Telegram bot OK"
else
    echo "❌ Telegram bot FAILED"
    # Send alert
fi

# Check Discord bot
if curl -s -H "Authorization: Bot ${DISCORD_BOT_TOKEN}" "https://discord.com/api/users/@me" | grep -q '"id":'; then
    echo "✅ Discord bot OK"
else
    echo "❌ Discord bot FAILED"
    # Send alert
fi

# และอื่นๆ...
```

### Backup และ Recovery

**Backup strategy:**
1. เก็บ tokens ใน encrypted storage
2. Backup configuration files
3. Document recovery procedures
4. Test recovery process เป็นประจำ

**Recovery checklist:**
- [ ] Identify compromised tokens
- [ ] Revoke old tokens
- [ ] Generate new tokens
- [ ] Update configurations
- [ ] Test functionality
- [ ] Update backup files

## Troubleshooting

### Token ไม่ทำงาน

**ปัญหา:** Token ถูก reject หรือ authentication ล้มเหลว

**สาเหตุที่เป็นไปได้:**
- Token หมดอายุหรือถูก revoke
- Permissions/scopes ไม่เพียงพอ
- Token format ไม่ถูกต้อง
- Rate limiting จาก platform

**วิธีแก้ไข:**
1. ตรวจสอบ token ใน developer console ของ platform
2. ตรวจสอบว่า scopes/permissions ครบถ้วน
3. สร้าง token ใหม่และทดสอบ
4. ตรวจสอบ logs ของ OpenClaw สำหรับ error messages

### Connection Issues

**ปัญหา:** ไม่สามารถเชื่อมต่อกับ platform

**สาเหตุที่เป็นไปได้:**
- Network connectivity issues
- Firewall blocking connections
- Platform services down
- Webhook URLs ไม่ถูกต้อง

**วิธีแก้ไข:**
1. ทดสอบ network connectivity: `ping api.telegram.org`
2. ตรวจสอบ firewall rules
3. ตรวจสอบ platform status pages
4. Verify webhook URLs และ SSL certificates

### Webhook ไม่ทำงาน

**ปัญหา:** ไม่ได้รับ messages จาก platform

**สาเหตุที่เป็นไปได้:**
- Webhook URL ไม่ถูกต้องหรือไม่สามารถเข้าถึงได้
- SSL certificate issues
- Platform ไม่สามารถส่งถึง server
- Webhook secret/token ไม่ตรง

**วิธีแก้ไข:**
1. ตรวจสอบว่า server สามารถเข้าถึงได้จาก internet
2. Verify SSL certificate ถูกต้อง
3. ตรวจสอบ webhook logs ใน platform console
4. Test webhook ด้วย tools เช่น ngrok หรือ webhook.site

### Permission Errors

**ปัญหา:** Bot ไม่สามารถส่งข้อความหรือเข้าถึงข้อมูล

**สาเหตุที่เป็นไปได้:**
- Bot permissions ไม่เพียงพอ
- Bot ไม่ถูก invite เข้า channels
- Scopes ไม่ครบถ้วน

**วิธีแก้ไข:**
1. ตรวจสอบ bot permissions ใน platform settings
2. Invite bot เข้า channels ที่ต้องการ
3. เพิ่ม scopes ที่ขาดหายไป
4. Restart OpenClaw และ test

### Rate Limiting

**ปัญหา:** API calls ถูก limit หรือ block

**สาเหตุที่เป็นไปได้:**
- ส่ง requests เร็วเกินไป
- เกิน quota ของ platform
- IP address ถูก block

**วิธีแก้ไข:**
1. Implement exponential backoff
2. ตรวจสอบ rate limits ของแต่ละ platform
3. ใช้ webhooks แทน polling เมื่อเป็นไปได้
4. Contact platform support ถ้าจำเป็น

### QR Code Login Issues (WhatsApp/Signal)

**ปัญหา:** QR code ไม่สามารถสแกนได้

**สาเหตุที่เป็นไปได้:**
- QR code หมดอายุ
- Network issues ขณะ login
- Device storage เต็ม
- Multiple sessions

**วิธีแก้ไข:**
1. รีเฟรช QR code
2. ตรวจสอบ network connectivity
3. ลบ sessions เก่าใน WhatsApp/Signal
4. ลอง login อีกครั้ง

### Security Concerns

**ปัญหา:** สงสัยว่า token ถูก compromise

**Immediate Actions:**
1. Revoke token ใน platform console ทันที
2. สร้าง token ใหม่
3. อัปเดต OpenClaw configuration
4. ตรวจสอบ access logs สำหรับ suspicious activity
5. เปลี่ยน passwords ที่เกี่ยวข้อง
6. แจ้งเตือนทีม security ถ้าจำเป็น

### Platform-Specific Issues

#### Telegram
- **403 Forbidden:** ตรวจสอบ bot token และ permissions
- **429 Too Many Requests:** Implement rate limiting
- **400 Bad Request:** ตรวจสอบ message format

#### Discord
- **401 Unauthorized:** Token หมดอายุหรือไม่ถูกต้อง
- **403 Forbidden:** Bot permissions ไม่เพียงพอ
- **429 Rate Limited:** รอและ retry ด้วย backoff

#### Slack
- **invalid_auth:** Tokens ไม่ถูกต้อง
- **missing_scope:** เพิ่ม scopes ที่ขาด
- **account_inactive:** Workspace inactive

#### Facebook Messenger
- **100 Invalid parameter:** ตรวจสอบ parameter format
- **200 Permissions error:** เพิ่ม permissions
- **613 Rate limited:** Implement backoff

### Debug Tools และ Logs

**เปิดใช้งาน debug logging:**
```bash
export DEBUG=openclaw:*
openclaw channels status
```

**ตรวจสอบ logs:**
```bash
openclaw logs --tail 100
```

**Test channel connectivity:**
```bash
openclaw channels test telegram
openclaw channels test discord
```

### Getting Help

ถ้าปัญหายังคงอยู่:
1. ตรวจสอบ [OpenClaw documentation](/docs/)
2. ค้นหาใน GitHub issues
3. ติดต่อ community support
4. File bug report กับ platform ที่เกี่ยวข้อง

## อ้างอิง

- [Telegram Bot API](https://core.telegram.org/bots)
- [Discord Developer Portal](https://discord.com/developers/docs)
- [Slack API](https://api.slack.com/)
- [Mattermost Integrations](https://docs.mattermost.com/developer/bot-accounts.html)
- [OpenClaw Channels Documentation](/channels/)</content>
<parameter name="filePath">d:\Projects\Github\Openclaw-Setup\API-KEYS-GUIDE.md