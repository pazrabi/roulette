# Боевой сервер на Aeza (с 2026-11-07)

Railway закончился (триал), прод перенесён на общий сервер Aeza (Амстердам, 1 vCPU, 3,9 ГБ RAM; на нём же gemini-pool, droidhub, avia-bot).

| Что | Значение |
|---|---|
| Адрес сервера | `https://casino.193-41-86-92.sslip.io` (Caddy, сертификат автоматический; при желании заменить на свой домен: строка в `/etc/caddy/Caddyfile`, затем `API_URL` в клиенте, `PUBLIC_URL`, `/setdomain` не нужен) |
| Служба | `casino.service` (systemd), пользователь `casino`, порт 127.0.0.1:8100, `Nice=-5`, `MemoryMax=800M` |
| Код | `/home/casino/app` (копия папки `bot/`), окружение Python 3.12 в `/home/casino/venv` |
| Секреты | `/home/casino/casino.env` (права 600, не в git): те же переменные, что были на Railway, `DB_PATH=/home/casino/data/players.db` |
| База и копии | `/home/casino/data/players.db`, копии `/home/casino/data/backups` (ежедневно, шифрованная копия владельцу раз в неделю) |

## Выкатка
```
rsync -a --delete --exclude .venv --exclude .env --exclude __pycache__ --exclude backups --exclude '*.db' -e "ssh -i ~/.ssh/aeza_pool" bot/ root@193.41.86.92:/home/casino/app/
ssh -i ~/.ssh/aeza_pool root@193.41.86.92 'chown -R casino:casino /home/casino/app && /home/casino/venv/bin/pip install -q -r /home/casino/app/requirements.txt && systemctl restart casino && sleep 4 && curl -s http://127.0.0.1:8100/health'
```
Клиент по-прежнему выкатывается пушем в `main` (GitHub Pages). Перед выкаткой со сменой схемы: `/backupnow` у бота.

## Логи и проверка
`journalctl -u casino -n 50 --no-pager`, `systemctl status casino`, снаружи `curl https://casino.193-41-86-92.sslip.io/health`. Webhook бот ставит сам при старте (`getWebhookInfo` должен показывать этот адрес).

## Переезд 2026-11-07
- База восстановлена из зашифрованной копии от 2026-10-30 (8 игроков, 245 раундов), всё, что было после неё на Railway, потеряно. Старый том Railway не удалялся.
- Webhook перевыставлен автоматически, `API_URL` и CSP клиента переключены на новый адрес.
- Тестовый контур staging остаётся на Railway и сейчас не работает; при необходимости разворачивать по образцу этого файла (отдельный пользователь, порт, бот).
