# 🧠 MCQ Challenge App

An auto-generating **Multiple Choice Question (MCQ)** quiz application for software engineers. The app uses a local [Ollama](https://ollama.com/) LLM to generate fresh CS questions every 10 minutes, serves them through a sleek dark-themed web UI, and optionally pushes each question as a native quiz poll to a Telegram channel.

---

## ✨ Features

- **AI-Powered Question Generation** — Uses a local Ollama model (default: `qwen2.5:7b`) to create unique, technically accurate MCQs.
- **Auto-Refresh** — A new question is generated every 10 minutes via a built-in cron job.
- **15 CS Topics** — Questions span OOP, Data Structures, Algorithms, OS, Networking, DBMS, System Design, Git, Web Dev, Testing, Security, Cloud, and more.
- **Duplicate Prevention** — Maintains a rolling history of the last 50 questions and instructs the model to avoid repeats.
- **Server-Side Answer Validation** — The correct answer is never exposed to the client; validation happens on the server.
- **Score Tracking** — Tracks correct/wrong answers, cumulative score, current streak, and best streak in `localStorage`.
- **Streak Bonus Scoring** — Earn bonus points for consecutive correct answers (up to +10 bonus per question).
- **Telegram Integration** — Every new MCQ is automatically sent as a native Telegram quiz poll to a configured channel or group.
- **Dark-Themed UI** — A modern, responsive interface with smooth hover animations and visual feedback.
- **PM2 Production Config** — Ships with a ready-to-use `ecosystem.config.js` for 24/7 deployment.

---

## 📸 Demo

<p align="center">
  <img src="./assets/MCQ-Quiz.jpeg" alt="MCQ Challenge App Screenshot" width="600" />
</p>

---

## 🛠️ Tech Stack

| Layer        | Technology                                                       |
| ------------ | ---------------------------------------------------------------- |
| **Backend**  | [Node.js](https://nodejs.org/) + [Express](https://expressjs.com/) |
| **Frontend** | Vanilla HTML, CSS & JavaScript (single-page, no framework)       |
| **AI / LLM** | [Ollama](https://ollama.com/) (local inference, default model: `qwen2.5:7b`) |
| **Scheduler**| [node-cron](https://www.npmjs.com/package/node-cron)             |
| **Telegram** | Telegram Bot API (`sendPoll` — native quiz polls)                |
| **Process Manager** | [PM2](https://pm2.keymetrics.io/) (optional, for production) |

---

## 📁 Project Structure

```
mcq-app-final/
├── public/
│   └── index.html          # Frontend UI (HTML + CSS + JS, all-in-one)
├── server.js               # Express server, API routes & cron scheduler
├── generateMCQ.js          # Ollama prompt builder, JSON parser & question storage
├── telegram.js             # Telegram Bot API integration (quiz poll sender)
├── ecosystem.config.js     # PM2 deployment configuration
├── questions.json           # Auto-generated file — stores current & past questions
├── package.json            # Project metadata and dependencies
└── README.md
```

---

## 🚀 Installation

### Prerequisites

- **Node.js** v16 or later
- **Ollama** installed and running locally ([install guide](https://ollama.com/download))
- An Ollama model pulled (default: `qwen2.5:7b`)

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/your-username/mcq-app-final.git
cd mcq-app-final

# 2. Install dependencies
npm install

# 3. Pull the default Ollama model (if not already downloaded)
ollama pull 
```

---

## 🔐 Environment Variables

The app works out of the box with defaults. All environment variables are **optional**:

| Variable              | Default             | Description                                      |
| --------------------- | ------------------- | ------------------------------------------------ |
| `PORT`                | `5000`              | Port the Express server listens on               |
| `OLLAMA_MODEL`        | `qwen2.5:7b`        | Ollama model name used for question generation   |
| `TELEGRAM_BOT_TOKEN`  | —                   | Your Telegram Bot token (from [@BotFather](https://t.me/BotFather)) |
| `TELEGRAM_CHAT_ID`    | —                   | Target channel (`@channel_name`) or group chat ID |

> **Note:** If `TELEGRAM_BOT_TOKEN` or `TELEGRAM_CHAT_ID` are not set, the app still runs normally — Telegram sends are simply skipped with a console warning.

---

## ▶️ How to Run the Project

### Development

```bash
# Make sure Ollama is running
ollama serve

# Start the app
npm start
```

Open **http://localhost:5000** in your browser.

### With Environment Variables

```bash
PORT=3000 OLLAMA_MODEL=llama3 TELEGRAM_BOT_TOKEN=your_token TELEGRAM_CHAT_ID=@your_channel npm start
```

---

## 🚢 PM2 Deployment

The project includes a production-ready [ecosystem.config.js](ecosystem.config.js) for PM2.

### Setup

```bash
# Install PM2 globally (if not already installed)
npm install -g pm2

# Edit ecosystem.config.js and fill in your Telegram credentials
# Then start the app
pm2 start ecosystem.config.js

# Useful PM2 commands
pm2 logs mcq-app         # View live logs
pm2 status               # Check process status
pm2 restart mcq-app      # Restart the app
pm2 stop mcq-app         # Stop the app
pm2 delete mcq-app       # Remove from PM2
```

### PM2 Configuration Highlights

| Setting                    | Value    | Purpose                                      |
| -------------------------- | -------- | --------------------------------------------- |
| `autorestart`              | `true`   | Automatically restarts on crash               |
| `max_memory_restart`       | `300M`   | Restarts if memory usage exceeds 300 MB       |
| `max_restarts`             | `20`     | Maximum crash-loop retries                    |
| `restart_delay`            | `5000`   | 5-second delay between restarts               |
| `exp_backoff_restart_delay`| `100`    | Exponential backoff on repeated failures      |
| `min_uptime`               | `30s`    | Process must survive 30s to count as stable   |

Logs are written to `./logs/out.log` and `./logs/error.log` with timestamps.

---

## ⏱️ Cron Schedule

The app uses [node-cron](https://www.npmjs.com/package/node-cron) to generate a new MCQ automatically:

| Cron Expression  | Schedule         | Description                          |
| ---------------- | ---------------- | ------------------------------------ |
| `*/10 * * * *`   | Every 10 minutes | Generates a fresh MCQ question       |

- On **first startup**, if no question exists, one is generated immediately.
- The frontend polls for the new question and includes a visible **countdown timer** showing time until the next question.

---

## 🤖 How the Telegram Bot Works

1. **Setup** — Create a bot via [@BotFather](https://t.me/BotFather), get the token, add the bot to your channel/group as an admin, and set the environment variables.

2. **Quiz Poll** — Every time a new MCQ is generated, the app sends a **native Telegram quiz poll** using the [`sendPoll`](https://core.telegram.org/bots/api#sendpoll) API with `type: "quiz"`.

3. **How it looks** — Channel members see an interactive poll. They tap an option, and Telegram reveals whether they're correct, along with the explanation and vote percentages.

4. **Safety limits** — Questions, options, and explanations are automatically truncated to respect Telegram's character limits:
   - Question: 300 characters
   - Each option: 100 characters
   - Explanation: 200 characters

5. **Fault tolerant** — Telegram sends are fire-and-forget. If the Telegram API is unreachable or credentials are missing, the app continues running without interruption.

---

## 💡 Future Improvements

- Add support for difficulty levels (easy, medium, hard).
- Let users choose specific topics to focus on.
- Add a leaderboard with persistent backend storage.
- Support multiple concurrent questions / timed quiz sessions.
- Add user authentication and progress tracking across devices.
- Provide detailed analytics on topic-wise performance.

---

## 👤 Author

**Your Name**

- GitHub: [@your-username](https://github.com/your-username)

---

<p align="center">
  Built with ❤️ and local AI
</p>
