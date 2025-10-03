# 📺 M3U Recorder Bot

A **Telegram Bot** to record M3U8 (or M3U) streams, manage recordings, and deliver them directly in Telegram. This README is a complete reference for **admins**, **users**, and developers: configuration, commands (with detailed parameter explanations), deployment, M3U → JSON conversion, troubleshooting and examples.

---

## 🔎 Table of Contents

1. Features
2. Configuration (`config.py`) — full breakdown
3. Commands (User + Admin) — detailed explanations & examples
4. M3U → JSON Converter — how it works + headless example
5. Deployment — Windows & Ubuntu + .env example
6. Troubleshooting & FAQ
7. Security & Notes
8. Credits

---

## 🚀 Features (Expanded)

<details>
<summary>🎥 Recording Features (expanded)</summary>

- Record from **M3U8** URLs or from a **predefined channel list (JSON)** stored in `M3U8_FILES_DIRECTORY`.
- On setup, choose **video track** and then one or more **audio tracks** to include in the final file.
- Live **progress updates**: percentage, ETA, and cancel button.
- **Splits** files larger than the configured max (keeps parts under Telegram limit — ~2GB).
- Generates a thumbnail and uses `ffprobe` + fallback (hachoir) to compute duration.
- Robust `ffmpeg` invocation: supports adding HTTP headers (Referer/User-Agent) if needed.
</details>

<details>
<summary>👤 User Features</summary>

- Tiered access:
  - **Default** users — limited duration & parallel tasks.
  - **Verified** users — via shortlink verification (if enabled).
  - **Premium** — granted by admin with expiry.
- Commands for interacting with the bot: `/rec`, `/mytasks`, `/cancel`, `/status`, `/verify`, `/search`, `/channel`.
</details>

<details>
<summary>🛠 Admin Features (expanded)</summary>

- Grant/revoke premium access (short-lived) with `/auth` and `/deauth`.
- Add or remove **admin** accounts which can use special commands.
- Add new channel lists (JSON) and remove them, either by uploading JSON or via inline modes (see below).
- Export data (tasks, logs, premium list, admin list) via `/pull`.
- View FFmpeg logs and last messages with `/flog`.
- Use `/tasks` to get a paginated, interactive view of all active tasks, and cancel if necessary.
</details>

---

## 📋 Configuration (`config.py`) — full breakdown

> **Tip:** Prefer environment variables in production. `config.py` reads environment variables by default — see the example `.env` later.

| Variable | Type | Purpose / Where used | Example |
|---|---:|---|---|
| `API_ID` | int | Telegram API ID — required by Pyrogram to create Client. | `12345678` |
| `API_HASH` | str | Telegram API Hash — used with API_ID. | `abcd1234efgh5678` |
| `BOT_TOKEN` | str | Bot token from BotFather used to run the bot. | `1234:ABCdefGHIjkl` |
| `OWNER_ID` | int | Owner user ID — special permissions, owner-only commands. | `987654321` |
| `MONGO_URI` | str | MongoDB connection string — used for storing premium users, verification tokens, admins. | `mongodb+srv://user:pw@cluster...` |
| `M3U8_FILES_DIRECTORY` | str | Directory where JSON channel lists live. Bot loads all `*.json` from here. | `./m3u8_channels` |
| `WORKING_GROUP` | int | Primary group id where bot posts verification and some notifications. | `-1001234567890` |
| `TIMEZONE` | str | Timezone used to format timestamps in `/status` etc. | `Asia/Kolkata` |
| `GROUP_LINK` | str | Invite link used in messages to point users to the official group. | `https://t.me/yourgroup` |
| `NUM_WORKERS` | int | Number of async worker tasks for recordings on *this instance*. | `4` |
| `GLOBAL_MAX_PARALLEL_TASKS` | int | Absolute cap of parallel recordings across all users. Useful to protect CPU / bandwidth. | `10` |
| `FFPROBE_TIMEOUT` | int | Timeout (seconds) for probing streams with `ffprobe`. | `30` |
| `PREMIUM_MAX_DURATION_SEC` | int | Maximum duration (seconds) allowed per recording for premium users (stored in seconds). Example: `2*3600` for 2 hours. | `7200` |
| `PREMIUM_PARALLEL_TASKS` | int | Max parallel tasks allowed per premium user. | `2` |
| `VERIFIED_MAX_DURATION_SEC` | int | Max duration for verified users (seconds). | `2700` (45m) |
| `VERIFIED_PARALLEL_TASKS` | int | Max parallel tasks for verified users. | `2` |
| `ENABLE_SHORTLINK` | bool | Toggle shortlink verification on/off. If `false` `/verify` will be disabled. | `true` |
| `VERIFICATION_EXPIRY_SECONDS` | int | How long verification remains valid (seconds). | `4*3600` |
| `SHORTLINK_URL` | str | Shortlink service base URL used to create verification links. Optional. | `https://vplink.in` |
| `SHORTLINK_API` | str | API key for shortlink service. Optional. | `xxx-yyy-zzz` |
| `STATUS_PAGE_SIZE` | int | How many tasks to show per page for `/tasks` pagination. | `10` |
| `PROGRESS_UPDATE_INTERVAL` | int | Interval (seconds) between progress message edits to reduce API spam. | `60` |

**Note on durations:** The values like `PREMIUM_MAX_DURATION_SEC` are seconds in `config.py`. Human examples given above are for readability. When using `/auth` you may pass `30d` or `48h` — see command docs below.

---

## 💬 Commands — Full Reference, parameters, examples & behavior

> The following expands the short command list and explains parameters like `.L#`, `<task_id>`, `[m3u8|log|premium|admin]`, duration formats, and how commands are meant to be used.

### 👤 User Commands (detailed)

#### `/start`
- Usage: `/start` or `/start verify_<token>` (used by deeplink verification).
- Behavior: Show welcome message and handle deeplink verification payloads.

#### `/help`
- Usage: `/help`
- Behavior: Shows available commands. Contents vary for normal users vs admins.

#### `/status`
- Usage: `/status`
- Behavior: Shows your tier (Owner/Admin/Premium/Verified/Default), limits and premium expiry (if any).

#### `/rec "[URL/Channel]" [HH:MM:SS] [Optional Filename] [.L#]`
- Usage examples:
  - ` /rec "https://example.com/stream.m3u8" 00:10:00 "My Clip"`
  - `/rec "Disney Channel (4K)" 00:30:00 "Kids" .L1`
- Parameter breakdown:
  - `"[URL/Channel]"` — Either a direct M3U8 URL **or** the name/key of a channel contained in one of your `*.json` channel lists.
  - `[HH:MM:SS]` — Duration. Accepts `HH:MM:SS` or `MM:SS` formats. You may also provide seconds (e.g., `600`).
  - `[Optional Filename]` — Quoted filename for the output video (used when uploaded to Telegram).
  - `[.L#]` — Optional list selector. If you have many JSON lists, the bot sorts them alphabetically and assigns indexes `.L1`, `.L2`, etc. Use `.L#` to pick which list to search for the channel name. Example: `.L1` selects the first JSON list in sorted order.
- Behavior: Starts interactive setup if required (select video & audio tracks). After selection, job is queued and will run when a worker is available.

#### `/mytasks`
- Usage: `/mytasks`
- Behavior: Shows your active / queued jobs (with links to progress messages when available).

#### `/cancel <task_id>`
- Usage: `/cancel`, `/cancel <task_id>`
- Behavior: If no `task_id` is provided, the bot lists your active/queued tasks with inline buttons to cancel. If a `task_id` is provided, the bot attempts to cancel that job (if you're the owner or admin). Cancellation handles:
  - **Queued jobs**: removed from queue.
  - **Running jobs**: sends terminate to ffmpeg process, cleans up temp files.

#### `/channel`
- Usage: `/channel`
- Behavior: Browse loaded JSON channel lists and pick a channel to use in `/rec`.

#### `/search <query>`
- 💡 **Usage:**
• `/search "Disney"` (searches all lists)

• `/search "Disney India" .l1` (searches list 1)

• `/search "Disney SD" .l1 .l3` (searches lists 1 and 3)

- Behavior: Searches across channel lists for channels matching the query based on the specified filters.

#### `/verify`
- Usage: `/verify`
- Behavior: Send the user a verification link (optionally shortened by `SHORTLINK_URL`) which, when opened, verifies the user for a limited time set by `VERIFICATION_EXPIRY_SECONDS`.


### 👑 Admin Commands (detailed)

**Important**: Most admin commands are restricted to admin users (stored in DB) or the OWNER_ID. Where noted, commands expect you to **reply** to a target user's message.

#### `/auth <duration>` — Grant Premium
- How to call (recommended): **Reply** to a user's message in chat with `/auth <duration>`.
- Alternative: `<admin> /auth <user_id> <duration>` may work depending on the implementation, but reply is the safe method.
- `<duration>` syntax:
  - `30d` = 30 days
  - `7d` = 7 days
  - `48h` = 48 hours
  - `12h` = 12 hours
- What it does:
  - Adds/updates a document in `premium_users` collection with:
    - `_id`: user id
    - `is_premium`: true
    - `expires_at`: epoch seconds when premium expires
    - `name` & `username` for convenience
  - Notifies the target user that they have premium access (attempts to DM).
  - `/status` will show the expiry date/time.
- Example (reply mode): Reply to @someuser's message with: `/auth 30d`

#### `/deauth` — Revoke Premium
- How to call: Reply to user's message with `/deauth` OR use `/deauth <user_id>`.
- What it does:
  - Deletes the user's document from `premium_users` collection.
  - Attempts to notify the user that premium access has been revoked.
- Example: Reply to user's message with `/deauth`.

#### `/add_m3u8` — Add Channel Lists
There are multiple **modes** supported conceptually — the bot may allow one or more of these depending on how it's configured.

💡 Usage:
***Inline Method***
1. **File Mode**
Upload The JSON FILE Containing m3u8 list to telegram `WORKING_GROUP` or `BOT's DM`, reply to the .json file with /add_m3u8.

2. **Individual Link Mode**: 
`/add_m3u8 "json name" "url" "channel name" "group"`
Automatically Creates JSON FIle With Given Details


***Direct Method***
Directly upload JSON Folder In The `M3U8_FILES_DIRECTORY`

**Notes & behavior**:
- After adding, the bot loads the file and the list appears in `channel` and `search` commands.
- Filenames are used as list identifiers (and implicitly in the alphabetical sorted order used for `.L#` indexing).

#### `/remove_m3u8 "json_name"`
- Usage: `/remove_m3u8 "channels_list.json"` or reply to a list’s file/message and run `/remove_m3u8`.
- Removes the JSON file from `M3U8_FILES_DIRECTORY` and reloads lists.

#### `/pull [m3u8|log|premium|admin]` — Export Data
- Usage examples:
  - `/pull m3u8` → returns a ZIP or files of all JSON channel lists.
  - `/pull log <task_id>` → returns the FFmpeg log file for `task_id` if available.
  - `/pull premium` → returns a list (CSV/JSON) of premium users with expiry timestamps.
  - `/pull admin` → returns the admin users list saved in DB.
- Purpose: to backup or inspect data.

#### `/flog [file|msg] <task_id>` — FFmpeg logs of specific Task ID
- Usage:
  - `/flog file <task_id>` → send full log file (if present in `flogs/`).
  - `/flog msg <task_id>` → show last ~50 lines of the log in a message (avoids large messages).
- Admin-only — helpful for debugging failed recordings.

#### `/tasks` — View All Tasks
- Shows a paginated view of *all active tasks* (not just your own). Buttons allow paging through lists. Each task shows status, filename, user and a link to the progress message when available.
- The pagination buttons expire (typically after a short time) to avoid stale interactions.

#### `/admin_panel`
- Opens an interactive inline control panel — quick access to common admin actions (reload lists, show queued jobs, quick deauth, etc.).
- Behavior & layout may vary by release, it’s an admin convenience UI that uses callback queries.

---

## 🔢 Command Parameter Cheat-Sheet

- `.L#` — list selector: choose which JSON list to search when supplying a **channel name** instead of a direct URL. `.L1` = first JSON file in alphabetical order, `.L2` = second, etc.
- `<task_id>` — unique job identifier returned by the bot when you create a recording. Usually long random hex; UI often shows first 8 characters as shorthand. Use full id to target a job for cancellation or pulling logs.
- `[m3u8|log|premium|admin]` — options for `/pull` telling the bot which dataset to export.
- `/rec "[URL/Channel]" [HH:MM:SS] [Optional Filename] [.L#]` — full format; quoting the first argument if it contains spaces is required.
- `/auth <user_id> <duration>` — duration accepts `Nd` (days) or `Nh` (hours). The bot converts to seconds and stores expiry.

---

## 🗂️ M3U → JSON Converter (in repo)

File: `M3U To Json.py` — interactive converter using a small Tk file picker + command-line prompts.

### What it does
- Reads `.m3u` or `.m3u8` files.
- Extracts lines with `#EXTINF:` to get names and `group-title` if present.
- Matches the next HTTP line as the stream URL.
- Prompts you for group descriptions and channel descriptions (when group not present).
- Saves a JSON mapping of `slugified_key: { name, url, Group }` which the bot can load directly.

### How to run (GUI/interactive mode)
```bash
python "M3U To Json.py"
```
- Choose your `.m3u` file in the GUI picker. Choose output `.json`. Provide group descriptions when prompted.

### Headless (non-GUI) quick example
If you want a non-interactive converter (no prompts — useful for automation), you can use the short script below and adapt as needed.

```python
# m3u_to_json_headless.py (example)
import re, json, sys, os

def slugify(text):
    text = text.lower()
    text = re.sub(r"[^\w\s-]", "", text)
    return re.sub(r"[\s-]+", "_", text).strip('_')

def convert(in_path, out_path, default_group=None):
    with open(in_path, 'r', encoding='utf-8') as f:
        lines = [l.strip() for l in f if l.strip()]
    channels = {}
    i = 0
    while i < len(lines):
        line = lines[i]
        if line.startswith('#EXTINF:'):
            name_match = re.search(r',([^,]+)$', line)
            name = name_match.group(1).strip() if name_match else 'Unknown'
            group_match = re.search(r'group-title="([^"]*)"', line)
            group = group_match.group(1) if group_match else (default_group or '')
            # next line expected to be URL
            url = lines[i+1] if i+1 < len(lines) and lines[i+1].lower().startswith('http') else ''
            key = slugify(name)
            # ensure unique
            base = key; c = 1
            while key in channels:
                key = f"{base}_{c}"; c += 1
            channels[key] = {"name": name, "url": url, "Group": group}
            i += 2
        else:
            i += 1
    with open(out_path, 'w', encoding='utf-8') as f:
        json.dump(channels, f, indent=2, ensure_ascii=False)

if __name__ == '__main__':
    if len(sys.argv) < 3:
        print('Usage: python m3u_to_json_headless.py input.m3u output.json [default_group]')
        sys.exit(1)
    convert(sys.argv[1], sys.argv[2], sys.argv[3] if len(sys.argv) > 3 else None)
```

> Move the produced `output.json` to `M3U8_FILES_DIRECTORY` and restart the bot or use `/add_m3u8`.

---

## 🖥️ Deployment (full details)

> Two common flows: **Clone + run** or **Manual**. For production, run on Ubuntu server with a process manager (systemd, supervisord) and ensure `ffmpeg` is available.

### Requirements & recommended packages
- Python 3.9+
- ffmpeg & ffprobe in PATH (`ffmpeg -version` should run)
- MongoDB (Atlas or self-hosted)
- pip packages (example `requirements.txt`):
  - `pyrogram`, `pymongo`, `hachoir`, `pillow`, `aiofiles`, `httpx`, `python-dotenv`, `pytz`

Example `requirements.txt` (minimal):
```
pyrogram
pymongo
hachoir
pillow
aiofiles
httpx
python-dotenv
pytz
```

### Example `.env` (recommended — do NOT commit to git)
```
API_ID=12345678
API_HASH=abcd1234efgh5678
BOT_TOKEN=1234:ABCdefGHIjkl
OWNER_ID=987654321
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/dbname
M3U8_FILES_DIRECTORY=./m3u8_channels
WORKING_GROUP=-1001234567890
TIMEZONE=Asia/Kolkata
GROUP_LINK=https://t.me/yourgroup
NUM_WORKERS=4
GLOBAL_MAX_PARALLEL_TASKS=10
ENABLE_SHORTLINK=true
SHORTLINK_URL=https://vplink.in
SHORTLINK_API=your_shortlink_api_key
PROGRESS_UPDATE_INTERVAL=10
```

> Use `python-dotenv` or your process manager to load env variables. `config.py` already reads from the environment by default.

### Deployment Steps (clone & run)
1. `git clone https://github.com/your/repo.git`
2. `cd repo`
3. `python -m venv venv && source venv/bin/activate` or on Windows `venv\Scripts\activate`
4. `pip install -r requirements.txt`
5. Create your `.env` or edit `config.py` for a quick local test
6. Place JSON channel lists in `M3U8_FILES_DIRECTORY`
7. Run: `python main.py`

### Production (Ubuntu) — example with systemd
Create `/etc/systemd/system/m3u-recorder.service`:
```
[Unit]
Description=M3U Recorder Bot
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/m3u-recorder
EnvironmentFile=/home/ubuntu/m3u-recorder/.env
ExecStart=/home/ubuntu/m3u-recorder/venv/bin/python main.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
Then:
```
sudo systemctl daemon-reload
sudo systemctl enable m3u-recorder
sudo systemctl start m3u-recorder
sudo journalctl -u m3u-recorder -f
```

### Windows Quick Run
- Install Python 3.9+ from python.org
- Set up virtualenv, `pip install -r requirements.txt`
- Ensure `ffmpeg.exe` is on your PATH
- Run `python main.py` or create a scheduled Task / NSSM service for persistence

---

## ❗ Troubleshooting & FAQ

**FFmpeg not found**
- Error: `FileNotFoundError: [Errno 2] No such file or directory: 'ffmpeg'`
- Fix: Install ffmpeg and ensure binaries are in PATH. On Ubuntu: `sudo apt install ffmpeg`.

**MongoDB Connection Failure**
- Check `MONGO_URI`, network access to Atlas cluster, and that credentials are correct.

**Bot cannot post to group**
- If uploads fail with `PeerIdInvalid`, the bot may be removed from the group or lacks permission. Re-add bot and give it permission to send messages/documents.

**Recording fails**
- Check `flogs/` for FFmpeg logs and use `/flog msg <task_id>` as admin.

**Task stuck, cannot cancel**
- Admin can use `/cancel <task_id>` to force cancel; check `running_jobs` / `pending_states` in-memory state.

---

## 🔐 Security & Notes

- **Do not** hardcode credentials in the repo. Use `.env` or environment variables on servers.
- Limit `NUM_WORKERS` based on CPU/bandwidth.
- Keep `ffmpeg` updated to avoid stream compatibility issues.
- Consider running the bot behind a reverse proxy or in a sandbox for better isolation.

---

## 🙌 Credits

- Built using Pyrogram, FFmpeg & MongoDB.
- Converter script uses simple parsing and slugify logic — adapt it for your M3U flavor.

---
- Add example `systemd` unit already pasted above into the repo

Tell me which of the three to add and I’ll place them in the canvas as files.
