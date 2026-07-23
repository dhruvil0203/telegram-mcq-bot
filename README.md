<h1 align="center">🧠 MCQ Challenge App</h1>

<p align="center">
  <strong>AI-Powered Multiple Choice Quiz — Auto-Generating, Self-Refreshing</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-339933?logo=node.js&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/Express-000000?logo=express&logoColor=white" alt="Express" />
  <img src="https://img.shields.io/badge/Ollama-000000?logo=ollama&logoColor=white" alt="Ollama" />
  <img src="https://img.shields.io/badge/PM2-2B037A?logo=pm2&logoColor=white" alt="PM2" />
  <img src="https://img.shields.io/badge/License-MIT-green" alt="MIT License" />
</p>

---

## ✨ Features

- **AI-Powered Question Generation** — Uses a local [Ollama](https://ollama.com/) LLM (default: `qwen2.5:7b`) to create unique, technically accurate MCQs.
- **Auto-Refresh** — A new question is generated every **10 minutes** via a built-in cron job (`node-cron`).
- **15 CS Topics** — Questions span OOP, Data Structures, Algorithms, Operating Systems, Networking, DBMS, System Design, Git, Web Dev, Testing, Security, Cloud, and more.
- **Duplicate Prevention** — Maintains a rolling history of the last 50 questions and instructs the model to avoid repeats.
- **Server-Side Answer Validation** — The correct answer is never exposed to the client; validation happens securely on the server.
- **Score Tracking** — Tracks correct/wrong answers, cumulative score, current streak, and best streak in `localStorage`.
- **Streak Bonus Scoring** — Earn bonus points for consecutive correct answers (up to +10 bonus per question).
- **Telegram Integration** — Every new MCQ is automatically sent as a native **Telegram quiz poll** to a configured channel or group.
- **Dark-Themed UI** — A modern, responsive interface with smooth hover animations and visual feedback.
- **PM2 Production Config** — Ships with a ready-to-use `ecosystem.config.js` for 24/7 deployment with auto-restart, log rotation, and memory limits.

<p align="left">
  <img src="./assets/MCQ-Quiz.jpeg" alt="MCQ Challenge App Screenshot" width="350" height="712" />
</p>

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **Backend** | [Node.js](https://nodejs.org/) + [Express](https://expressjs.com/) |
| **Frontend** | Vanilla HTML, CSS & JavaScript (single-page, no framework) |
| **AI / LLM** | [Ollama](https://ollama.com/) (local inference, default: `qwen2.5:7b`) |
| **Scheduler** | [node-cron](https://www.npmjs.com/package/node-cron) |
| **Telegram** | Telegram Bot API (`sendPoll` — native quiz polls) |
| **Process Manager** | [PM2](https://pm2.keymetrics.io/) (optional, for production) |

---

## 📁 Project Structure

```
mcq-app-final/
├── public/
│   └── index.html              # Frontend UI (HTML + CSS + JS, all-in-one)
├── assets/
│   ├── MCQ-Quiz.jpeg           # Screenshot for README
│   └── readme.md               # Extended project documentation
├── server.js                   # Express server, API routes & cron scheduler
├── generateMCQ.js              # Ollama prompt builder, JSON parser & storage
├── telegram.js                 # Telegram Bot API integration (quiz poll sender)
├── ecosystem.config.js         # PM2 deployment configuration
├── .env                        # Environment variables (gitignored)
├── .env.example                # Environment variable template
├── .gitignore                  # Git ignore rules
├── LICENSE                     # MIT License
├── package.json                # Project metadata and dependencies
└── README.md                   # This file
```

---

## 🚀 Quick Start

### Prerequisites

- **Node.js** v16 or later
- **Ollama** installed and running locally ([install guide](https://ollama.com/download))
- An Ollama model pulled (default: `qwen2.5:7b`)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/mcq-app-final.git
cd mcq-app-final

# 2. Install dependencies
npm install

# 3. Pull the default Ollama model (if not already downloaded)
ollama pull qwen2.5:7b

# 4. (Optional) Configure environment variables
cp .env.example .env
# Edit .env with your preferred settings
```

### Run

```bash
# Make sure Ollama is running
ollama serve

# Start the app
npm start
```

Open **http://localhost:5000** in your browser.

---

## 🔐 Environment Variables

All variables are configured via `.env` (copy from `.env.example`):

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `5000` | Port the Express server listens on |
| `OLLAMA_MODEL` | `qwen2.5:7b` | Ollama model used for question generation |
| `TELEGRAM_BOT_TOKEN` | — | Telegram Bot token (from [@BotFather](https://t.me/BotFather)) |
| `TELEGRAM_CHAT_ID` | — | Target channel (`@channel`) or group chat ID |

> **Note:** If Telegram variables are not set, the app runs normally — Telegram sends are skipped with a console warning.

---

## 🚢 PM2 Production Deployment

The project includes a pre-configured `ecosystem.config.js` for [PM2](https://pm2.keymetrics.io/).

```bash
# Install PM2 globally
npm install -g pm2

# Set environment variables (or rely on .env)
export TELEGRAM_BOT_TOKEN=your_token
export TELEGRAM_CHAT_ID=@your_channel

# Start the app
pm2 start ecosystem.config.js
pm2 save
pm2 startup   # enables auto-start on server reboot
```

### PM2 Highlights

| Setting | Value | Purpose |
|---------|-------|---------|
| `autorestart` | `true` | Auto-restart on crash |
| `max_memory_restart` | `300M` | Restart if memory exceeds 300 MB |
| `max_restarts` | `20` | Maximum retries before backing off |
| `exp_backoff_restart_delay` | `100` | Exponential backoff on repeated failures |

Logs are written to `./logs/out.log` and `./logs/error.log` with timestamps.

### Log Rotation (Recommended)

```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
```

---

## ⏱️ Cron Schedule

The app generates a new MCQ automatically every **10 minutes** using `node-cron` (`*/10 * * * *`):

- On **first startup**, if no question exists, one is generated immediately.
- The frontend polls for the new question and shows a **countdown timer**.

To change the interval, edit the cron pattern in `server.js` and the `POLL_MS` constant in `public/index.html`.

---

## 🤖 Telegram Integration

1. Create a bot via [@BotFather](https://t.me/BotFather) and get the token.
2. Add the bot as an **admin** to your channel or group.
3. Set `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` in `.env`.
4. Every new MCQ is automatically posted as a native Telegram quiz poll.

Character limits (auto-truncated):
- Question: **300 characters**
- Each option: **100 characters**
- Explanation: **200 characters**

---

## 🧪 API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/current-question` | `GET` | Returns the current question (without the correct answer) |
| `/api/check-answer` | `POST` | Validates an answer server-side; returns correct/incorrect + explanation |

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
