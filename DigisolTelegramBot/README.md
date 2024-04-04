# Digisol Telegram Bot

Create folder on server that will contain application and python environment:

```bash
mkdir /var/www/digisol_tg_bot
```

Create python environment in that folder

```bash
cd /var/www/digisol_tg_bot
python3 -m venv env
```

Move `requirements.txt` on that folder and install all dependenices:
```bash
source env/bin/activate
pip install -r requirements.txt
```

Upload `bot.py` and `db.py` files in `/var/www/digisol_tg_bot`.

Create environment file and fill that with required variables:

```bash
touch /var/www/digisol_tg_bot/.env
echo "
TELEGRAM_BOT_KEY=6564725503:AAEREPcP-NgRoWAhzc8E4S0EvVma7Ayj5WQ
TICK_TIME=120
GROUP_ID=-1002135122845
DB_NAME=digisol_telegram
DB_USER=digisol_telegram_user
DB_PASSWORD=pWmwb2y4nzxeBP8DLGjXqv
DB_HOST=localhost
DB_PORT=5432
" > .env
```

Group id is the id of telegram group where this bot should post messages. To get this group id see the instructions: https://stackoverflow.com/a/72649378

Next setup database:

```bash
# login into postgresql installed on server. You can use remote psql server if needed.
sudo -u postgres psql

# create user and database:
CREATE DATABASE digisol_telegram;
CREATE USER digisol_telegram_user WITH PASSWORD 'pWmwb2y4nzxeBP8DLGjXqv';
ALTER ROLE digisol_telegram_user SET client_encoding TO 'utf8';
ALTER ROLE digisol_telegram_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE digisol_telegram_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE digisol_telegram TO digisol_telegram_user;

# Next create tables:
CREATE TABLE public.jobs (
	id serial4 NOT NULL,
	name varchar NOT NULL,
	link varchar NOT NULL,
	CONSTRAINT jobs_link_key UNIQUE (link),
	CONSTRAINT jobs_name_key UNIQUE (name),
	CONSTRAINT jobs_pkey PRIMARY KEY (id)
);
CREATE TABLE public.settings (
	id serial4 NOT NULL,
	"name" varchar NOT NULL,
	value varchar NOT NULL,
	CONSTRAINT settings_name_key UNIQUE (name),
	CONSTRAINT settings_pkey PRIMARY KEY (id),
	CONSTRAINT settings_value_key UNIQUE (value)
);
```

Create systemd configuration file with following content:

```bash
# /etc/systemd/system/digisol_tg_bot.service
[Unit]
Description=Digisol telegram
After=network.target

[Service]
ExecStart=/var/www/digisol_tg_bot/env/bin/python /var/www/digisol_tg_bot/bot.py
Restart=on-failure
RestartSec=1s
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=digisol_tg_bot
User=root
Group=root
Environment=PYTHONUNBUFFERED=1
EnvironmentFile=/var/www/digisol_tg_bot/.env

[Install]
WantedBy=multi-user.target
```

Enable and start service:

```bash
systemctl enable digisol_tg_bot.service
systemctl start digisol_tg_bot.service
```

To watch logs use journalctl:

```bash
journalctl -u digisol_tg_bot.service -f
```

### NOTE

This service has crontab task that restart service each 30 minutes. Telegram will disconnect bot if there are no activity at all for some period of time. 

```bash
crontab -e

# config to periodic restart:
*/30 * * * *    systemctl restart digisol_tg_bot.service
```

### SETUP

At this point bot is running and ready to parse rss feeds. To set rss link send next command:

```bash
/setLink <upwork rss feed link>
```

This command will save rss link into database. You can add more then one link to bot. Bot will parse them one by one with 15 seconds timeout.

After that you can restart bot if needed:

```bash
systemctl restart digisol_tg_bot.service
```