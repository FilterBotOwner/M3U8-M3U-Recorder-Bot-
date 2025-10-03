# 📺 M3U Recorder Bot

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
- Export bot data with `/pull`
- Monitor and paginate all tasks (`/tasks`)
- View FFmpeg logs for each recording with `/flog TASK_ID`
- Interactive **Admin Panel** with `/admin_panel`
</details>

<details>
<summary>⚙️ Technical Features</summary>

- MongoDB for storage (Premium, Admins, Verification)
- Configurable via environment variables
- Uses `ffmpeg` and `ffprobe`
- Optimized with async workers
- Separate tool for **M3U → JSON conversion**
</details>

---

## 📋 Configuration (`config.py`)

| Variable | Purpose | Default |
|----------|---------|---------|
| `API_ID` | Telegram API ID (from [my.telegram.org](https://my.telegram.org)) | `12345678` |
| `API_HASH` | Telegram API Hash | `xxxx` |
| `BOT_TOKEN` | Token from [BotFather](https://t.me/BotFather) | `xxxx:xxxx` |
| `OWNER_ID` | Telegram ID of bot owner (full permissions) | `1234567890` |
| `MONGO_URI` | MongoDB connection string for storing users/admins/premium data | `mongodb+srv://...` |
| `M3U8_FILES_DIRECTORY` | Directory where converted JSON channel lists are stored | `./m3u8_channels` |
| `WORKING_GROUP` | Group ID where bot posts verification & notifications | `-100xxxx` |
| `TIMEZONE` | Used to display dates/times correctly | `Asia/Kolkata` |
| `GROUP_LINK` | Invite link for your Telegram group | `https://t.me/...` |
| `NUM_WORKERS` | How many recordings run in parallel (workers) | `4` |
| `GLOBAL_MAX_PARALLEL_TASKS` | Maximum recordings across all users | `10` |
| `FFPROBE_TIMEOUT` | Timeout for ffprobe probing streams | `30` sec |
| `PREMIUM_MAX_DURATION_SEC` | Max per-recording duration for Premium users | `2h` |
| `PREMIUM_PARALLEL_TASKS` | Max simultaneous tasks for Premium users | `2` |
| `VERIFIED_MAX_DURATION_SEC` | Max per-recording duration for Verified users | `45m` |
| `VERIFIED_PARALLEL_TASKS` | Max simultaneous tasks for Verified users | `2` |
| `ENABLE_SHORTLINK` | Toggle verification system (true/false) | `true` |
| `VERIFICATION_EXPIRY_SECONDS` | How long verification remains valid | `4h` |
| `SHORTLINK_URL` | Shortlink provider base URL | `https://vplink.in` |
| `SHORTLINK_API` | API key for shortlink provider | `xxxxxxxx` |
| `STATUS_PAGE_SIZE` | Number of tasks per page in `/tasks` | `10` |
| `PROGRESS_UPDATE_INTERVAL` | Seconds between progress updates | `60` |

---

## 💬 Bot Commands

### 👤 User Commands

📚 Available Commands:

| Command | Description |
|---------|-------------|
| `/start` | Start the bot & verification process 👋 |
| `/help` | Show help menu 📖 |
| `/status` | Check your status, tier, limits 📊 |
| `/rec` | Record a stream: `/rec "[URL/Channel]" [HH:MM:SS] [Filename] [.L#]` 🎬 |
| `/mytasks` | See your active recording tasks 📋 |
| `/cancel <task_id>` | Cancel a specific task (lists yours if ID not given) 🛑 |
| `/channel` | Browse available channel lists 📺 |
| `/search <query>` | Search channels by name/description 🔍 |
| `/verify` | Verify account (if enabled) 🔑 |

### 👑 Admin Commands

🛠 Admin Commands:

| Command | Description |
|---------|-------------|
| `/tasks` | View & paginate all active recording tasks 🌐 |
| `/auth <user_id> <duration>` | Grant premium (e.g. `30d`, `48h`) 💎 |
| `/deauth <user_id>` | Revoke premium access 🗑️ |
| `/add_m3u8` | Upload and add new M3U8 JSON list ➕ |
| `/remove_m3u8 "json_name"` | Remove an M3U8 channel list ➖ |
| `/pull [m3u8\log\premium\admin]` | Export stored bot data 📥 |
| `/add_admin <user_id>` | Promote user to Admin 👑 |
| `/remove_admin <user_id>` | Demote/remove Admin ⚔️ |
| `/flog [file\msg] <task_id>` | Fetch FFmpeg logs for a task 📄 |
| `/admin_panel` | Interactive inline admin control panel ⚙️ |

---

## 🗂️ M3U → JSON Converter

This repo includes a helper script `M3U To Json.py` to convert `.m3u` or `.m3u8` playlists into JSON channel lists usable by the bot.

### 🔹 How to Use

1. Run the script:
   ```bash
   python "M3U To Json.py"
   ```

2. Select your `.m3u`/`.m3u8` file via the file picker
3. Choose where to save the `.json`
4. The script will:
   - Extract channel names, URLs, and groups
   - Prompt you for group/channel descriptions
   - Save a clean JSON file in the format the bot understands

5. Move the JSON file into your `m3u8_channels` directory or add it with `/add_m3u8`

---

## ⚡️ Pre-requirements

- Python **3.9+**
- [ffmpeg](https://ffmpeg.org/) & [ffprobe](https://ffmpeg.org/ffprobe.html)
- MongoDB Atlas or local MongoDB
- Telegram API credentials (`API_ID`, `API_HASH`)
- Bot token from [BotFather](https://t.me/BotFather)

---

## 🖥️ Deployment

### 🔹 Method 1: Clone Repository (Recommended)
```bash
# Clone repository
git clone https://github.com/your/repo.git
cd repo

# Install dependencies
pip install -r requirements.txt

# Run the bot
python main.py
```

### 🔹 Method 2: Manual Setup
1. Download the source code (ZIP or TAR).
2. Extract files.
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
4. Configure values in `config.py` (API_ID, API_HASH, BOT_TOKEN, MONGO_URI, etc).
5. Run:
   ```bash
   python main.py
   ```

### 🔹 Ubuntu Linux Quick Setup
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip ffmpeg -y
git clone https://github.com/your/repo.git
cd repo
pip3 install -r requirements.txt
python3 main.py
```

---

## 📷 Screenshots

> *(Add screenshots here: welcome message, recording progress, task list, admin panel, verification flow)*

---

## 💡 Notes

- Large files auto-split (2GB Telegram limit).
- Verification optional (via shortlink provider).
- Configurations override with environment variables.
- `M3U To Json.py` is separate and must be run manually to prepare channel lists.

---

## 🙌 Credits

- Built with [Pyrogram](https://docs.pyrogram.org/)
- Recording via [FFmpeg](https://ffmpeg.org/)
- Database: [MongoDB](https://www.mongodb.com/)
