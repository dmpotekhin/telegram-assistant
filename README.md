# 🤖 Личный ассистент в Telegram

Персональный бот на базе Hermes Agent. Живёт в Telegram, думает через DeepSeek,
помнит всё в Obsidian. Понимает текст и голос, ставит напоминания, ведёт
задачи, сохраняет идеи и закладки, качает видео с YouTube.

---

## 🧠 Что умеет

| Команда | Пример |
|---------|--------|
| ⏰ **Напоминания** | `Напомни завтра в 9 утра проверить почту` |
| ☐ **Задачи** | `Задача: подготовить презентацию к пятнице` |
| 💡 **Идеи** | `Идея: сделать лендинг для нового продукта` |
| 📝 **Заметки** | `Заметка: сегодня обсуждали бюджет на Q3` |
| 🔗 **Закладки** | Кинуть любую ссылку — сохранится с тегами |
| 🎬 **YouTube** | Кинуть ссылку на видео — скачает на диск |
| 👤 **Контакты** | `Контакт: Иван, +79031234567, др 15 марта` |
| 🎤 **Голосовые** | Надиктовать сообщение — бот поймёт |
| 🔍 **Поиск** | `Найди заметку про договор` |

---

## 🏗 Архитектура

```
Telegram ←→ Бот ←→ Hermes Gateway ←→ DeepSeek (LLM)
  │                                      │
  │                                      ├── cronjob (⏰ напоминания)
  │                                      ├── write_file (📝 Obsidian)
  │                                      ├── yt-dlp (🎬 YouTube)
  │                                      └── web_search (🔍 поиск)
  │
  └── Ты пишешь с любого устройства
```

- **Hermes Agent** — open-source фреймворк для AI-агентов (Nous Research)
- **DeepSeek** — LLM, ~$2–5/мес при активном использовании
- **Obsidian** — все заметки, задачи, идеи хранятся локально
- **yt-dlp** — скачивает видео с YouTube

---

## 🚀 Как настроить у себя

### 1. Установить Hermes

```bash
# macOS / Linux
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# Windows (PowerShell)
irm https://hermes-agent.nousresearch.com/install.ps1 | iex
```

### 2. Получить API-ключ DeepSeek

- Зарегистрироваться на https://platform.deepseek.com/
- Создать API-ключ, пополнить баланс ($5 хватит на месяц)
- `hermes setup` → выбрать DeepSeek → вставить ключ → модель `deepseek-chat`

### 3. Создать Telegram-бота

- В Telegram найти **@BotFather**
- `/newbot` → имя → username (оканчивается на `bot`)
- Скопировать токен

### 4. Настроить Hermes

```bash
# Добавить токен в .env
echo "TELEGRAM_BOT_TOKEN=123456:ABCdef..." >> ~/.hermes/.env

# Включить Telegram в gateway
hermes config set gateway.platforms.telegram.token "123456:ABCdef..."
hermes config set gateway.platforms.telegram.enabled true

# Заменить SOUL.md на этот:
cp SOUL.md ~/.hermes/profiles/developer/SOUL.md
```

### 5. Запустить

```bash
hermes gateway
```

Написать боту `/start` в Telegram. Hermes запросит подтверждение:

```bash
hermes pairing approve telegram <код>
```

Готово. Бот отвечает на сообщения, ставит напоминания, сохраняет заметки.

---

## ☁️ Развернуть на VPS (24/7)

Бот работает пока запущен `hermes gateway`. Чтобы не держать Mac включённым —
запустить на сервере.

### Минимальные требования

- Ubuntu 22.04 или Debian 12
- 2 ГБ RAM, 10 ГБ диск
- ~$5–6/мес (Hetzner, DigitalOcean, Aeza, Timeweb)

### Установка

```bash
# 1. Hermes
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# 2. Python-зависимости
pip3 install yt-dlp

# 3. Скопировать конфиг и SOUL.md с локальной машины
scp ~/.hermes/.env user@vps:~/.hermes/
scp ~/.hermes/profiles/developer/config.yaml user@vps:~/.hermes/profiles/developer/
scp ~/.hermes/profiles/developer/SOUL.md user@vps:~/.hermes/profiles/developer/

# 4. Настроить автозапуск (systemd)
sudo tee /etc/systemd/system/hermes-gateway.service << 'EOF'
[Unit]
Description=Hermes Gateway
After=network.target

[Service]
Type=simple
User=youruser
ExecStart=/home/youruser/.local/bin/hermes gateway
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now hermes-gateway
```

### Что не будет работать на VPS

- **Obsidian** — заметки можно перенаправить в Markdown-файлы или Notion
- **yt-dlp** — видео скачаются на сервер, не на твой Mac. Можно настроить синхронизацию через rclone или Telegram
- **Голосовые** — нужен ffmpeg: `apt install ffmpeg`

### Альтернативы Obsidian на VPS

```bash
# Вариант 1: Markdown-файлы (просто папка)
mkdir -p ~/bot-notes
# Поменять путь в SOUL.md на /home/user/bot-notes/

# Вариант 2: Notion (через API)
# Вариант 3: Google Docs (через API)
```

---

## 📂 Где что хранится

| Данные | Путь |
|--------|------|
| Заметки | `Brain/notes/bot/notes.md` |
| Задачи | `Brain/notes/bot/tasks.md` |
| Идеи | `Brain/notes/bot/ideas.md` |
| Напоминания | `Brain/notes/bot/reminders.md` |
| Закладки | `Brain/notes/bot/bookmarks.md` |
| Контакты | `Brain/notes/bot/contacts.md` |
| Видео (файлы) | `~/Downloads/bot/` |
| Видео (ссылки) | `Brain/notes/bot/videos.md` |

---

## 💰 Сколько стоит

| Компонент | Цена |
|-----------|------|
| DeepSeek API (LLM) | ~$2–5/мес |
| Telegram Bot API | Бесплатно |
| Hermes Agent | Open-source |
| VPS (опционально) | ~$5–6/мес |
| yt-dlp + Obsidian | Бесплатно |

**Итого: ~$3–11/мес** в зависимости от наличия VPS.

---

## ⚠️ Важно

- **Mac должен быть включён** — бот работает пока запущен `hermes gateway`
- **VPS решает эту проблему** — бот работает 24/7 независимо от твоего компьютера
- **VPN** — нужен для Telegram API если ты в России
- **Токен бота** — не публикуй, не коммить в git
- **yt-dlp** — следи за свободным местом на диске

---

## 🔧 Стек

`Hermes Agent` · `DeepSeek` · `Telegram Bot API` · `Obsidian` · `yt-dlp` · `Python` · `systemd`
