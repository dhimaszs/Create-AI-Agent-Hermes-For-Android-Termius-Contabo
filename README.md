# Tutorial Lengkap: Membuat AI Agent Hermes di Android dengan Termius & VPS Contabo

Tutorial step-by-step untuk membuat AI Agent Hermes yang berjalan di Android menggunakan Termius dan VPS Contabo. Dari nol sampai bot Telegram nyala! 🚀

## 📋 Daftar Isi

1. [Persiapan Awal](#1-persiapan-awal)
2. [Setup Termius di Android](#2-setup-termius-di-android)
3. [Setup VPS Contabo](#3-setup-vps-contabo)
4. [Install Hermes Agent](#4-install-hermes-agent)
5. [Cari Credit & API Key Gratis](#5-cari-credit--api-key-gratis)
6. [Konek ke Bot Telegram](#6-konek-ke-bot-telegram)
7. [Troubleshooting](#7-troubleshooting)

---

## 1. Persiapan Awal

### Yang Dibutuhkan

| Tools | Keterangan |
|-------|------------|
| **HP Android** | Minimal Android 7.0+ |
| **Termius** | SSH Client untuk Android (Download di Play Store) |
| **VPS Contabo** | Server untuk install Hermes (€6.99/bulan) |
| **Bot Telegram** | Untuk interfacing dengan agent |
| **Credit/API Key** | Untuk akses AI model (ada yang gratis!) |

### Download Termius

1. Buka **Google Play Store**
2. Search **"Termius"**
3. Download & install app-nya
4. Buka Termius, buat akun (gratis) atau skip

---

## 2. Setup Termius di Android

### Langkah 1: Tambah VPS ke Termius

1. Buka Termius
2. Tap ikon **"+ NEW HOST"** (右下角)
3. Isi field berikut:

```
Label: Contabo VPS (bebas)
IP Address/Hostname: [IP VPS lo]
SSH Port: 22
Username: root
Password: [password VPS]
```

### Langkah 2: Connect ke VPS

1. Save host yang udah di-setup
2. Tap host tersebut
3. Accept fingerprint (first time only)
4. Kalo muncul login prompt, masukkan password
5. Udah connect ke VPS! 🎉

### Tips Termius

- **Add Key** (kunci SSH) biar gak perlu每次login pake password
- **Snippets**: Simpan command yang sering dipake
- **Themes**: Ganti tema biar nyaman di mata

---

## 3. Setup VPS Contabo

### Langkah 1: Beli VPS Contabo

1. Buka https://contabo.com
2. Pilih **VPS S** (€6.99/bulan) atau lebih
3. Pilih lokasi server (rekomendasi: Frankfurt atau Amsterdam)
4. Pilih OS: **Ubuntu 22.04 LTS**
5. Checkout & bayar

### Langkah 2: Edit credentials untuk contabo

After purchase, lo akan dapat email dengan:
- **IP Address**: contoh `162.144.89.198`
- **Username**: `root`
- **Password**: [random string]

### Langkah 3: Login Pertama Kali

```bash
# Login via Termius
ssh root@162.144.89.198

# Atau langsung dari Termius app
```

### Langkah 4: Update & Install Dependencies

```bash
# Update sistem
apt update && apt upgrade -y

# Install dependencies dasar
apt install curl wget git python3 python3-pip python3-venv ufw fail2ban -y

# Install Docker (optional tapi direkomendasikan)
curl -fsSL https://get.docker.com | sh

# Enable Docker
systemctl enable docker
systemctl start docker
```

### Langkah 5: Setup Firewall

```bash
# Allow SSH
ufw allow 22/tcp

# Allow specific port kalo perlu
ufw allow 8080/tcp

# Enable firewall
ufw enable

# Cek status
ufw status
```

### Langkah 6: Buat User Biasa (Best Practice)

```bash
# Buat user baru
adduser dimas

# Add ke sudo group
usermod -aG sudo dimas

# Switch ke user baru
su - dimas

# Setup SSH key untuk user baru
mkdir ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Paste public key lo di sini
chmod 600 ~/.ssh/authorized_keys
```

---

## 4. Install Hermes Agent

### Menggunakan Script Otomatis

```bash
# Login ke VPS sebagai root/user
# Jalankan script install otomatis

curl -fsSL https://raw.githubusercontent.com/NousResearch/Hermes/main/install.sh | bash
```

### Install Manual

```bash
# Clone repository Hermes
git clone https://github.com/NousResearch/Hermes.git
cd Hermes

# Setup virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install --upgrade pip
pip install hermes-agent

# Copy config example
cp .env.example .env
nano .env
```

### Konfigurasi .env

```env
# ===========================================
# MODEL PROVIDER - Pilih salah satu
# ===========================================

# Option 1: OpenAI (butuh API key)
OPENAI_API_KEY=sk-your-openai-key

# Option 2: Anthropic (butuh API key)
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key

# Option 3: FreeLLMAPI (GRATIS - recommended!)
FREE_LLM_API_KEY=freellmapi-free-key

# ===========================================
# TELEGRAM BOT CONFIG
# ===========================================
TELEGRAM_BOT_TOKEN=your-telegram-bot-token
TELEGRAM_CHAT_ID=your-chat-id

# ===========================================
# SECURITY
# ===========================================
SECRET_KEY=generate-random-string-disini
ALLOWED_USERS=your-telegram-user-id

# ===========================================
# DATABASE
# ===========================================
DATABASE_URL=sqlite:///./hermes.db
```

### Generate Secret Key

```bash
# Generate random secret key
python3 -c "from secrets import token_hex; print(token_hex(32))"
```

### Test Running

```bash
# Activate environment
source venv/bin/activate

# Run Hermes
python -m hermes_agent run

# Kalo berhasil, lo akan see:
# ✅ Hermes Agent started successfully!
# 🔗 Connected to Telegram
```

### Setup Systemd Service (Auto-start)

```bash
# Buat service file
sudo nano /etc/systemd/system/hermes.service
```

Paste ini:

```ini
[Unit]
Description=Hermes AI Agent
After=network.target

[Service]
Type=simple
User=dimas
WorkingDirectory=/home/dimas/hermes
ExecStart=/home/dimas/hermes/venv/bin/python -m hermes_agent run
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start service
sudo systemctl enable hermes
sudo systemctl start hermes
sudo systemctl status hermes
```

---

## 5. Cari Credit & API Key Gratis

### Option 1: FreeLLMAPI (RECOMMENDED - 100% FREE)

FreeLLMAPI menyediakan akses gratis ke berbagai model AI.

**Cara dapat key:**
1. Buka https://freellmapi.com
2. Daftar akun (gratis)
3. Dapat API key otomatis
4. Tidak perlu credit card!

**Model yang tersedia:**
- Qwen (recommended)
- Gemini
- Llama
- Dan banyak lagi

**Contoh penggunaan:**
```env
FREE_LLM_API_KEY=freellmapi-your-key
MODEL=deepseek-ai/DeepSeek-V4-Flash
```

### Option 2: Groq (FREE tier available)

Groq menyediakan API gratis dengan throughput tinggi.

**Cara dapat key:**
1. Buka https://console.groq.com
2. Sign up dengan email
3. Dapat $30 free credits
4. API key ada di dashboard

**Model:** Llama3, Mixtral, Gemma

### Option 3: Together AI (FREE tier)

**Cara dapat key:**
1. Buka https://together.ai
2. Sign up, dapat $5 free credits
3. Akses berbagai open source models

### Option 4: OpenRouter (FREE credits new user)

**Cara dapat key:**
1. Buka https://openrouter.ai
2. Sign up, dapat $1 free credits
3. Rate limiting lebih longgar

### Option 5: Hugging Face Inference API (FREE)

**Cara dapat key:**
1. Buka https://huggingface.co/settings/inference-endpoints
2. Generate API token (gratis)
3. Akses berbagai models

### Option 6: Google AI Studio (FREE tier)

**Cara dapat key:**
1. Buka https://aistudio.google.com
2. Login dengan Google account
3. Dapat API key di section "Get API key"
4. Free tier: 60 requests/minute

### Ringkasan API Key Gratis

| Provider | Free Credits | Model Available | Link |
|----------|--------------|-----------------|------|
| **FreeLLMAPI** | Unlimited (free tier) | Qwen, Gemini, Llama | freellmapi.com |
| **Groq** | $30 | Llama3, Mixtral | console.groq.com |
| **Together AI** | $5 | Various open source | together.ai |
| **OpenRouter** | $1 | 100+ models | openrouter.ai |
| **HuggingFace** | Free tier | Various | huggingface.co |
| **Google AI** | Free tier | Gemini Pro | aistudio.google.com |

---

## 6. Konek ke Bot Telegram

### Langkah 1: Buat Bot Telegram

1. Buka Telegram, search **@BotFather**
2. Send command `/newbot`
3. Beri nama bot: `My Hermes Bot`
4. Beri username bot (harus ending `bot`): `myhermesbot123_bot`
5. BotFather akan kasih **Bot Token**: `6123456789:AAFxxxxxxxxxxxxxxxxxxxxxx`
6. **SIMPAN TOKEN INI!**

### Langkah 2: Get Chat ID

1. Buka bot yang baru dibuat (search nama bot lo)
2. Send message: `/start`
3. Buka browser,访问:
   ```
   https://api.telegram.org/bot6123456789:AAFxxxxxxxxxxxxxxxxxxxxxx/getUpdates
   ```
4. Cari `"chat":{"id":YourTelegramID` — itu chat ID lo

### Langkah 3: Setup di Hermes

Edit file `.env`:

```env
TELEGRAM_BOT_TOKEN=6123456789:AAFxxxxxxxxxxxxxxxxxxxxxx
TELEGRAM_CHAT_ID=YourTelegramID
ALLOWED_USERS=YourTelegramID
```

### Langkah 4: Restart Hermes

```bash
# Restart service
sudo systemctl restart hermes

# Check logs
sudo journalctl -u hermes -f
```

### Langkah 5: Test Bot

1. Buka Telegram
2. Chat dengan bot lo
3. Ketik `/start`
4. Kalo bales, berarti **BERHASIL!** 🎉

### Command Bot Telegram

| Command | Fungsi |
|---------|--------|
| `/start` | Start bot |
| `/help` | Tampilkan bantuan |
| `/status` | Cek status agent |
| `/stats` | Statistik penggunaan |
| `/model` | Lihat model yang digunakan |
| `/reset` | Reset conversation |

---

## 7. Troubleshooting

### Error: "Connection refused"

```bash
# Cek apakah Hermes running
sudo systemctl status hermes

# Cek port listening
sudo netstat -tlnp | grep python

# Restart service
sudo systemctl restart hermes
```

### Error: "Invalid Bot Token"

1. Cek token di `.env` — pastikan benar
2. Cek format token: `123456:ABC-DEF...`
3. Regenerate token via @BotFather: `/revoke`

### Error: "Chat ID not allowed"

```bash
# Pastikan CHAT_ID sama persis
# Include tanda minus kalo bot dalam group
ALLOWED_USERS=YourTelegramID,-1001234567890
```

### Error: "API Key invalid"

1. Cek apakah API key sudah benar di `.env`
2. Pastikan key masih aktif
3. Cek credit/limit belum habis

### Error: "Module not found"

```bash
# Reinstall dependencies
source venv/bin/activate
pip install --upgrade hermes-agent
```

### Error: "Permission denied"

```bash
# Fix permissions
chmod +x venv/bin/hermes
chown -R $USER:$USER /home/$USER/hermes
```

### Cek Logs

```bash
# Real-time logs
sudo journalctl -u hermes -f

# Old logs
sudo journalctl -u hermes --since "1 hour ago"

# App logs
tail -f /home/dimas/hermes/logs/hermes.log
```

### Reset Everything

```bash
# Stop service
sudo systemctl stop hermes

# Backup config
cp .env .env.backup

# Remove and reinstall
cd ~
rm -rf hermes
git clone https://github.com/NousResearch/Hermes.git hermes
cd hermes
python3 -m venv venv
source venv/bin/activate
pip install hermes-agent
cp .env.backup .env

# Start again
sudo systemctl start hermes
```

---

## 🎉 Selamat!

Lo udah berhasil setup Hermes AI Agent!Sekarang lo punya:
- ✅ AI agent yang running 24/7 di VPS
- ✅ Akses via Telegram dari mana aja
- ✅ AI capabilities dengan credit gratis
- ✅ Bot Telegram yang bisa di-customize

## 📚 Referensi

- [Hermes Agent Official Docs](https://hermes-agent.nousresearch.com)
- [FreeLLMAPI](https://freellmapi.com)
- [Groq Console](https://console.groq.com)
- [Telegram BotFather](https://t.me/botfather)
- [Contabo](https://contabo.com)

## 🤝 Kontribusi

Tutorial ini open source. Kalo ada yang salah atau mau添 加内容, feel free untuk buat PR!

## 📄 Lisensi

MIT License

---

*Made with ❤️ menggunakan Hermes Agent*