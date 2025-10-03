# 📺 QZBCast Recorder Bot

A **Telegram Bot** to record M3U8 streams, manage recordings, and deliver them directly in Telegram with features like premium access, verification system, admin controls, and task management.

---

## 🚀 Features

<details>
<summary>🎥 Recording Features</summary>

- Record from **M3U8 links** or **predefined channels**
- Choose **video & audio tracks** before recording
- **Progress tracking** with cancel option
- Auto **splitting large files** (2GB limit)
- Generates **thumbnails** & metadata
- Uploads recorded videos directly to Telegram
</details>

<details>
<summary>👤 User Features</summary>

- **Verification system** with shortlink provider
- Tier-based limits (Default, Verified, Premium)
- `/status` command to view limits & expiry
- Easy-to-use `/rec` command
- View your recordings with `/mytasks`
- Cancel tasks with `/cancel`
</details>

<details>
<summary>🛠 Admin Features</summary>

- Manage premium users (`/auth`, `/deauth`)
- Add/remove admins (`/add_admin`, `/remove_admin`)
- Manage channel lists (`/add_m3u8`, `/remove_m3u8`)
- Monitor all tasks (`/tasks`)
- View FFmpeg logs (`/flog`)
</details>

<details>
<summary>⚙️ Technical Features</summary>

- MongoDB for storage (Premium, Admins, Verification)
- Configurable via environment variables
- Uses `ffmpeg` and `ffprobe`
- Optimized with async workers
</details>

---

## 📋 Configuration (`config.py`)

| Variable | Description | Default |
|----------|-------------|---------|
| `API_ID` | Telegram API ID from [my.telegram.org](https://my.telegram.org) | `26378339` |
| `API_HASH` | Telegram API Hash | `d5b14fef7f38588836b1a8ba762f8f08` |
| `BOT_TOKEN` | Bot token from [BotFather](https://t.me/BotFather) | `8022381023:XXXX` |
| `OWNER_ID` | Bot owner’s Telegram user ID | `7958067256` |
| `MONGO_URI` | MongoDB connection string | `mongodb+srv://...` |
| `M3U8_FILES_DIRECTORY` | Directory for channel JSON files | `./m3u8_channels` |
| `WORKING_GROUP` | Group ID where bot operates | `-1002852374763` |
| `TIMEZONE` | Bot timezone | `Asia/Kolkata` |
| `GROUP_LINK` | Invite link for group | `https://t.me/...` |
| `NUM_WORKERS` | Concurrent recording workers | `4` |
| `GLOBAL_MAX_PARALLEL_TASKS` | Max global recording tasks | `10` |
| `FFPROBE_TIMEOUT` | Timeout for `ffprobe` | `30` sec |
| `PREMIUM_MAX_DURATION_SEC` | Max duration per premium task | `2h` |
| `PREMIUM_PARALLEL_TASKS` | Parallel premium tasks | `2` |
| `VERIFIED_MAX_DURATION_SEC` | Max duration for verified users | `45m` |
| `VERIFIED_PARALLEL_TASKS` | Parallel verified tasks | `2` |
| `ENABLE_SHORTLINK` | Enable/disable verification shortlinks | `true` |
| `VERIFICATION_EXPIRY_SECONDS` | Verification validity | `4h` |
| `SHORTLINK_URL` | Shortlink provider URL | `https://vplink.in` |
| `SHORTLINK_API` | Shortlink provider API key | `xxxxxxxx` |
| `STATUS_PAGE_SIZE` | Tasks per page in `/tasks` | `5` |
| `PROGRESS_UPDATE_INTERVAL` | Interval for progress updates | `10s` |

---

## 💬 Bot Commands

### 👤 User Commands

| Command | Description |
|---------|-------------|
| `/start` | Start bot & verification handling |
| `/help` | Show help message |
| `/status` | Show your tier, limits, expiry |
| `/rec` | Record stream or channel |
| `/mytasks` | Show your active recordings |
| `/cancel` | Cancel your recording |
| `/channel` | Browse channel lists |
| `/search <query>` | Search channels |
| `/verify` | Verify account (if enabled) |

### 👑 Admin Commands

| Command | Description |
|---------|-------------|
| `/tasks` | View all active tasks |
| `/auth <user_id> <duration>` | Grant premium (e.g. `30d`, `48h`) |
| `/deauth <user_id>` | Revoke premium |
| `/add_m3u8` | Add M3U8 channel list |
| `/remove_m3u8 "file.json"` | Remove M3U8 list |
| `/pull [m3u8|log|premium|admin]` | Export bot data |
| `/add_admin <user_id>` | Add new admin |
| `/remove_admin <user_id>` | Remove admin |
| `/flog [file|msg] <task_id>` | Fetch FFmpeg logs |

---

## ⚡️ Pre-requirements

- Python **3.9+**
- [ffmpeg](https://ffmpeg.org/) & [ffprobe](https://ffmpeg.org/ffprobe.html)
- MongoDB Atlas or local MongoDB instance
- Telegram API credentials (`API_ID`, `API_HASH`)
- Bot token from [BotFather](https://t.me/BotFather)

---

## 🖥️ Deployment

### 🔹 Windows

```bash
# Clone repo
git clone https://github.com/your/repo.git
cd repo

# Install requirements
pip install -r requirements.txt

# Run bot
python main.py
```

### 🔹 Ubuntu Linux

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install python3 python3-pip ffmpeg -y

# Clone repo
git clone https://github.com/your/repo.git
cd repo

# Install Python dependencies
pip3 install -r requirements.txt

# Run bot
python3 main.py
```

---

## 📷 Screenshots

> *(Add images here: start message, recording progress, task list, verification page)*

---

## 📝 Notes

- Large videos will be **split automatically** (2GB Telegram limit).
- Verification uses **shortlinks** (optional).
- All configurations can be overridden with **environment variables**.

---

## 💡 Credits

- Built with [Pyrogram](https://docs.pyrogram.org/)
- Recording with [FFmpeg](https://ffmpeg.org/)
- Database powered by [MongoDB](https://www.mongodb.com/)

---
