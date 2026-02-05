# การติดตั้ง OpenClaw แบบปลอดภัยที่สุด

## ภาพรวม

เอกสารนี้แนะนำวิธีการติดตั้ง OpenClaw ที่ให้ความปลอดภัยสูงสุด โดยเน้นการลดความเสี่ยงจาก malware, compromised binaries, หรือการเข้าถึงระบบ host ที่ไม่จำเป็น

วิธีการที่ปลอดภัยที่สุดคือการใช้ **Docker** ซึ่งให้การแยกตัว (isolation) อย่างสมบูรณ์ระหว่าง OpenClaw และระบบปฏิบัติการหลักของคุณ หรือ **ติดตั้งจาก source code** และ build เองเพื่อ verify ความถูกต้องของโค้ด

## ข้อกำหนดระบบ

- Docker Desktop (หรือ Docker Engine) + Docker Compose v2
- Git (สำหรับการ clone repository)
- Node.js >=22 และ pnpm (สำหรับการ build จาก source เท่านั้น)

## วิธีที่ 1: การติดตั้งด้วย Docker (แนะนำ - ปลอดภัยที่สุด)

Docker ให้การแยกตัวอย่างสมบูรณ์ ทำให้ OpenClaw ทำงานใน container ที่แยกจากระบบ host ไม่สามารถเข้าถึงไฟล์หรือทรัพยากรของ host ได้โดยตรง (ยกเว้นที่ mount อย่างชัดเจน)

### ขั้นตอนการติดตั้งด่วน

1. Clone repository:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

2. รัน script ตั้งค่า Docker:

```bash
./docker-setup.sh
```

Script นี้จะ:
- Build Docker image สำหรับ OpenClaw
- รัน onboarding wizard
- สร้าง gateway token และเขียนลง `.env`
- เริ่ม gateway ด้วย Docker Compose

3. เปิด Dashboard:

เปิด browser และไปที่ `http://127.0.0.1:18789/`

4. ใส่ token ที่ได้จากขั้นตอนก่อนหน้าเข้าไปใน Control UI

### การตั้งค่าเพิ่มเติม (Optional)

#### เพิ่ม extra mounts (สำหรับเข้าถึงไฟล์ host)

ถ้าต้องการ mount โฟลเดอร์เพิ่มเติมจาก host เข้า container:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/Documents:/home/node/documents:ro,$HOME/Downloads:/home/node/downloads:rw"
./docker-setup.sh
```

**คำเตือน:** เฉพาะ mount ที่จำเป็นเท่านั้น เพื่อรักษาความปลอดภัย

#### Persist container home directory

เพื่อให้ข้อมูลใน container คงอยู่ระหว่าง restart:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

#### ติดตั้ง system packages เพิ่มเติม

ถ้าต้องการติดตั้ง packages ระบบใน image:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential git"
./docker-setup.sh
```

### การจัดการ Container

- ดูสถานะ: `docker compose ps`
- หยุด: `docker compose down`
- เริ่มใหม่: `docker compose up -d`
- ดู logs: `docker compose logs -f openclaw-gateway`

### การอัปเดต

```bash
git pull
docker compose down
docker build -t openclaw:local -f Dockerfile .
docker compose up -d
```

### ความปลอดภัยของ Docker Setup

- Container ทำงานเป็น user `node` (ไม่ใช่ root)
- ไม่มีการเข้าถึง filesystem ของ host ยกเว้นที่ mount อย่างชัด
- Network แยกตัว (bridge network)
- Images สร้างจาก source ที่ verify ได้

## วิธีที่ 2: ติดตั้งจาก Source Code (Build เอง)

การ build จาก source ให้คุณ verify โค้ดก่อนใช้งาน และหลีกเลี่ยงการใช้ binaries ที่อาจถูก compromise

### ขั้นตอนการติดตั้ง

1. Clone และ verify repository:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

2. ตรวจสอบโค้ด (optional แต่แนะนำ):

```bash
# ดู commit history และ verify signatures ถ้ามี
git log --oneline -10
git verify-commit HEAD  # ถ้ามี GPG signatures
```

3. ติดตั้ง dependencies:

```bash
pnpm install
```

4. Build UI และ application:

```bash
pnpm ui:build
pnpm build
```

5. รัน onboarding:

```bash
openclaw onboard --install-daemon
```

หรือรันจาก repo:

```bash
pnpm openclaw onboard --install-daemon
```

### การรัน Gateway

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

### ความปลอดภัยของ Source Build

- คุณเห็นและ verify โค้ดทั้งหมดก่อน build
- ไม่ต้อง trust third-party binaries
- สามารถ audit dependencies ได้
- หลีกเลี่ยง supply chain attacks

## การตั้งค่า Authentication แบบปลอดภัย

### ใช้ API Keys แทน OAuth (สำหรับ production)

ใน config `~/.openclaw/openclaw.json`:

```json
{
  "routing": {
    "agents": {
      "main": {
        "auth": {
          "anthropic": {
            "apiKey": "your-secure-api-key-here"
          }
        }
      }
    }
  }
}
```

เก็บ API keys ใน environment variables หรือ secret management system

### เปิดใช้งาน Security Features

```json
{
  "gateway": {
    "auth": {
      "token": "generate-strong-random-token"
    },
    "security": {
      "auditLog": true,
      "pairingRequired": true
    }
  }
}
```

## การจัดการ Credentials แบบปลอดภัย

ดูคู่มือการหา API keys และ tokens ใน [API-KEYS-GUIDE.md](API-KEYS-GUIDE.md)

Credentials ถูกเก็บที่:
- `~/.openclaw/credentials/` (สำหรับ host installation)
- ใน container volume (สำหรับ Docker)

### สำรองข้อมูล Credentials

```bash
# สำหรับ host
cp -r ~/.openclaw/credentials /secure/backup/location

# สำหรับ Docker
docker run --rm -v openclaw_home:/source -v $(pwd)/backup:/dest alpine tar czf /dest/credentials.tar.gz -C /source .openclaw/credentials
```

### ลบ Credentials เมื่อไม่ใช้

```bash
rm -rf ~/.openclaw/credentials
# หรือสำหรับ Docker: docker volume rm openclaw_home
```

## การตรวจสอบความปลอดภัย

### รัน Security Audit

```bash
openclaw security audit --deep
```

### ตรวจสอบ Permissions

```bash
# ตรวจสอบว่าไฟล์ config ไม่มี permissions ที่กว้างเกินไป
ls -la ~/.openclaw/
```

### Monitor Logs

```bash
openclaw logs --follow
```

## คำแนะนำเพิ่มเติมเพื่อความปลอดภัย

1. **ใช้ VPN หรือ Tailscale** สำหรับ remote access แทนการเปิด port โดยตรง
2. **เปิดใช้งาน 2FA** บนบัญชีที่ใช้ API keys
3. **อัปเดตเป็นประจำ** และตรวจสอบ security advisories
4. **จำกัด network access** ของ container ให้มากที่สุด
5. **ใช้ read-only mounts** เมื่อเป็นไปได้
6. **หมุนเวียน tokens และ keys** เป็นประจำ

## การแก้ปัญหา

### Docker Issues

- ตรวจสอบ Docker Desktop กำลังทำงาน
- ตรวจสอบ disk space: `docker system df`
- Rebuild image: `docker build --no-cache -t openclaw:local -f Dockerfile .`

### Build Issues

- ล้าง cache: `pnpm store prune`
- Reinstall: `rm -rf node_modules && pnpm install`

### Permission Issues

- ตรวจสอบ ownership: `chown -R $(whoami) ~/.openclaw/`
- สำหรับ Docker: ตรวจสอบ file permissions ใน mounted volumes

## อ้างอิง

- [MULTI-AGENT-SETUP.md](MULTI-AGENT-SETUP.md) - คู่มือตั้งค่า Multi-Agent
- [API-KEYS-GUIDE.md](API-KEYS-GUIDE.md) - คู่มือหา API Keys
- [Docker Installation](/install/docker) - รายละเอียด Docker
- [Security Guide](/gateway/security) - คู่มือความปลอดภัย
- [Gateway Configuration](/gateway/configuration) - ตั้งค่า Gateway
- [Sandboxing](/gateway/sandboxing) - การแยก sandbox</content>
<parameter name="filePath">d:\Projects\Github\Openclaw-Setup\INSTALL-SECURE.md