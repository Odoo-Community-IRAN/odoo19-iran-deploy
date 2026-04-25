<div align="right" dir="rtl">

# راهنمای نصب سریع Odoo 19 Enterprise
### شبکه داخلی ایران

[![English](https://img.shields.io/badge/lang-English-blue)](./README.en.md)

> تمام دانلودها از میرورهای داخلی انجام می‌شود و نیازی به اینترنت بین‌الملل نیست.

---

## پیش‌نیازها

| مورد | حداقل |
|------|--------|
| سیستم‌عامل | Ubuntu 22.04 (Jammy) یا 24.04 (Noble) |
| RAM | 2 GB |
| فضای دیسک | 20 GB آزاد |
| دسترسی | Root یا sudo |

---

## مرحله ۰ — تنظیم میرور داخلی APT

منابع پیش‌فرض Ubuntu را با میرورهای داخلی جایگزین کنید.

### Ubuntu 22.04 (Jammy)

</div>

```bash
sudo nano /etc/apt/sources.list
```

<div align="right" dir="rtl">محتوا را با این جایگزین کنید:</div>

```
deb [trusted=yes] http://mirror.arvancloud.ir/ubuntu jammy main restricted universe multiverse
deb [trusted=yes] http://mirror.arvancloud.ir/ubuntu jammy-updates main restricted universe multiverse
deb [trusted=yes] http://mirror.arvancloud.ir/ubuntu jammy-backports main restricted universe multiverse
deb [trusted=yes] http://mirror.arvancloud.ir/ubuntu jammy-security main restricted universe multiverse
```

<div align="right" dir="rtl">

### Ubuntu 24.04 (Noble)

</div>

```bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources
```

<div align="right" dir="rtl">محتوا را با این جایگزین کنید:</div>

```
deb https://mirror.iranserver.com/ubuntu/ noble main restricted
deb https://mirror.iranserver.com/ubuntu/ noble-updates main restricted
deb https://mirror.iranserver.com/ubuntu/ noble universe
deb https://mirror.iranserver.com/ubuntu/ noble-updates universe
deb https://mirror.iranserver.com/ubuntu/ noble multiverse
deb https://mirror.iranserver.com/ubuntu/ noble-updates multiverse
deb https://mirror.iranserver.com/ubuntu/ noble-backports main restricted universe multiverse
deb https://mirror.iranserver.com/ubuntu/ noble-security main restricted
deb https://mirror.iranserver.com/ubuntu/ noble-security universe
deb https://mirror.iranserver.com/ubuntu/ noble-security multiverse
deb http://mirror.arvancloud.ir/ubuntu noble universe
```

<div align="right" dir="rtl">سپس آپدیت کنید:</div>

```bash
sudo apt-get update
```

---

<div align="right" dir="rtl">

## مرحله ۱ — نصب پیش‌نیازهای سیستم

</div>

```bash
sudo apt install -y \
  postgresql postgresql-contrib \
  python3 python3-pip python3-venv python3-dev python3-wheel python3-setuptools \
  git curl wget build-essential unzip \
  python3-cffi libxslt-dev libzip-dev libldap2-dev libsasl2-dev \
  libpng-dev libjpeg-dev libpq-dev libssl-dev libffi-dev \
  libxml2-dev libxslt1-dev libfreetype6-dev liblcms2-dev \
  libopenjp2-7-dev libtiff5-dev libwebp-dev libyaml-dev \
  fontconfig xfonts-75dpi gdebi-core node-less
```

---

<div align="right" dir="rtl">

## مرحله ۲ — نصب wkhtmltopdf

wkhtmltopdf برای تولید گزارش‌های PDF ضروری است. روی Ubuntu 22.04/24.04 ابتدا باید `libssl1.1` نصب شود.

</div>

```bash
wget https://dl.odoo-community.ir/odoo-tools/libssl1.1_1.1.1f-1ubuntu2_amd64.deb -O /tmp/libssl1.1.deb
sudo dpkg -i /tmp/libssl1.1.deb

wget https://dl.odoo-community.ir/odoo-tools/wkhtmltox_0.12.5-1.bionic_amd64.deb -O /tmp/wkhtmltox.deb
sudo gdebi --n /tmp/wkhtmltox.deb
```

<div align="right" dir="rtl">تأیید نصب:</div>

```bash
wkhtmltopdf --version
```

---

<div align="right" dir="rtl">

## مرحله ۳ — نصب Node.js و پشتیبانی RTL

Odoo برای کامپایل asset ها به Node.js و برای چیدمان راست‌به‌چپ فارسی/عربی به `rtlcss` نیاز دارد.

</div>

```bash
sudo apt-get install nodejs npm -y
sudo npm install -g rtlcss --registry=https://mirror2.chabokan.net/npm/
```

---

<div align="right" dir="rtl">

## مرحله ۴ — ایجاد کاربر سیستمی Odoo

</div>

```bash
sudo adduser --system --quiet --shell=/bin/bash --home=/opt/odoo19 --group odoo19
```

---

<div align="right" dir="rtl">

## مرحله ۵ — ایجاد کاربر PostgreSQL

</div>

```bash
sudo su - postgres -c "createuser -s odoo19 -P"
```

<div align="right" dir="rtl">

> از شما رمز خواسته می‌شود. آن را برای فایل تنظیمات در مرحله ۸ نگه دارید.

---

## مرحله ۶ — دانلود سورس Odoo 19

</div>

```bash
sudo mkdir -p /opt/odoo19 /etc/odoo /var/log/odoo19
sudo chown -R odoo19:odoo19 /opt/odoo19 /etc/odoo /var/log/odoo19

sudo su - odoo19

wget https://dl.odoo-community.ir/odoo-source/odoo19/src_19e.tar.gz
tar --strip-components=1 -zxf src_19e.tar.gz

cat > /opt/odoo19/odoo-bin << 'EOF'
#!/usr/bin/env python3
__import__('os').environ['TZ'] = 'UTC'
import odoo
if __name__ == "__main__":
    odoo.cli.main()
EOF
chmod +x /opt/odoo19/odoo-bin

mkdir -p /opt/odoo19/custom_addons
rm src_19e.tar.gz
exit
```

---

<div align="right" dir="rtl">

## مرحله ۷ — راه‌اندازی محیط مجازی Python

وابستگی‌های Python را درون محیط مجازی با میرور داخلی PyPI نصب کنید.

</div>

```bash
sudo su - odoo19
cd /opt/odoo19

python3 -m venv venv
source venv/bin/activate

pip install \
  --trusted-host mirror-pypi.runflare.com \
  -i https://mirror-pypi.runflare.com/simple/ \
  -r requirements.txt

# در صورت خطا از میرور جایگزین:
# pip install \
#   --trusted-host mirror.chabokan.net \
#   -i https://mirror.chabokan.net/repository/pypi-proxy/simple \
#   -r requirements.txt

pip install psycopg2-binary==2.9.9 pyopenssl==23.2.0

deactivate
exit
```

---

<div align="right" dir="rtl">

## مرحله ۸ — ایجاد فایل تنظیمات Odoo

`YOUR_DB_PASSWORD` را با رمز PostgreSQL که در مرحله ۵ تعیین کردید جایگزین کنید.

</div>

```bash
sudo nano /etc/odoo/odoo19.conf
```

```ini
[options]
admin_passwd = CHANGE_THIS_MASTER_PASSWORD
db_host = localhost
db_port = False
db_user = odoo19
db_password = YOUR_DB_PASSWORD
addons_path = /opt/odoo19/odoo/addons, /opt/odoo19/custom_addons
xmlrpc_port = 8069
logfile = /var/log/odoo19/odoo.log
```

```bash
sudo chown odoo19:odoo19 /etc/odoo/odoo19.conf
sudo chmod 640 /etc/odoo/odoo19.conf
```

<div align="right" dir="rtl">

> ⚠️ مقدار `admin_passwd` را به یک رشته تصادفی قوی تغییر دهید.

---

## مرحله ۹ — ایجاد سرویس Systemd

</div>

```bash
sudo nano /etc/systemd/system/odoo19.service
```

```ini
[Unit]
Description=Odoo 19.0
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo
PermissionsStartOnly=true
User=odoo19
Group=odoo19
ExecStart=/opt/odoo19/venv/bin/python3 /opt/odoo19/odoo-bin -c /etc/odoo/odoo19.conf
StandardOutput=journal+console
Restart=on-failure
KillMode=process
TimeoutSec=5min

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable odoo19
sudo systemctl start odoo19
```

---

<div align="right" dir="rtl">

## مرحله ۱۰ — نصب Nginx (اختیاری)

</div>

```bash
sudo apt-get install nginx -y
sudo nano /etc/nginx/sites-available/odoo19
```

```nginx
upstream odoo {
    server 127.0.0.1:8069;
}
upstream odoochat {
    server 127.0.0.1:8072;
}

server {
    listen 80;
    server_name YOUR_DOMAIN_OR_IP;

    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    access_log /var/log/nginx/odoo19.access.log;
    error_log  /var/log/nginx/odoo19.error.log;

    location /longpolling {
        proxy_pass http://odoochat;
    }

    location / {
        proxy_redirect off;
        proxy_pass http://odoo;
    }

    gzip on;
    gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
}
```

```bash
sudo ln -s /etc/nginx/sites-available/odoo19 /etc/nginx/sites-enabled/odoo19
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl restart nginx
```

---

<div align="right" dir="rtl">

## تأیید نهایی

</div>

```bash
sudo systemctl status odoo19
sudo tail -f /var/log/odoo19/odoo.log
```

<div align="right" dir="rtl">

مرورگر را باز کنید و به آدرس زیر بروید: **`http://YOUR_SERVER_IP:8069`**

---

## رفع اشکال

| مشکل | راه‌حل |
|------|--------|
| خطا در `apt update` | آدرس میرورها را در sources.list بررسی کنید |
| timeout در pip install | از میرور جایگزین `mirror.chabokan.net` استفاده کنید |
| خطای wkhtmltopdf در PDF | نصب `libssl1.1` را تأیید کنید |
| Odoo اجرا نمی‌شود | لاگ را بررسی کنید: `journalctl -u odoo19 -n 50` |
| خطای اتصال به دیتابیس | رمز PostgreSQL را با `odoo19.conf` تطبیق دهید |
| پورت 8069 در دسترس نیست | `sudo ufw allow 8069` را اجرا کنید |

---

## نکات امنیتی

> ⚠️ پیش از استفاده در محیط واقعی:
> 1. مقدار `admin_passwd` را به یک مقدار تصادفی قوی تغییر دهید
> 2. رمز کاربر `odoo19` در PostgreSQL را تغییر دهید
> 3. فایروال (`ufw`) را تنظیم کنید
> 4. پورت 8069 را مستقیماً در معرض اینترنت قرار ندهید — از Nginx استفاده کنید

---

## لینک‌های دانلود داخلی

| فایل | لینک |
|------|------|
| سورس Odoo 19 | `https://dl.odoo-community.ir/odoo-source/odoo19/src_19e.tar.gz` |
| wkhtmltopdf | `https://dl.odoo-community.ir/odoo-tools/wkhtmltox_0.12.5-1.bionic_amd64.deb` |
| libssl1.1 | `https://dl.odoo-community.ir/odoo-tools/libssl1.1_1.1.1f-1ubuntu2_amd64.deb` |
| میرور PyPI | `https://mirror-pypi.runflare.com/simple/` |
| میرور PyPI (جایگزین) | `https://mirror.chabokan.net/repository/pypi-proxy/simple` |
| میرور npm | `https://mirror2.chabokan.net/npm/` |

</div>
