# Odoo 19 Enterprise — Quick Install Guide
### Iran Internal Network

[![فارسی](https://img.shields.io/badge/lang-فارسی-green)](./README.fa.md)

> All downloads use domestic Iranian mirrors. No international internet access required.

---

## Requirements

| Item | Minimum |
|------|---------|
| OS | Ubuntu 22.04 (Jammy) or 24.04 (Noble) |
| RAM | 2 GB |
| Disk | 20 GB free |
| Access | Root or sudo |

---

## Step 0 — Configure Domestic APT Mirrors

Replace the default Ubuntu sources with domestic mirrors so `apt` can reach repositories.

### Ubuntu 22.04 (Jammy)

```bash
sudo nano /etc/apt/sources.list
```

Replace all content with:

```
deb [trusted=yes] http://mirror.arvancloud.ir/ubuntu jammy main restricted universe multiverse
deb [trusted=yes] http://mirror.arvancloud.ir/ubuntu jammy-updates main restricted universe multiverse
deb [trusted=yes] http://mirror.arvancloud.ir/ubuntu jammy-backports main restricted universe multiverse
deb [trusted=yes] http://mirror.arvancloud.ir/ubuntu jammy-security main restricted universe multiverse
```

### Ubuntu 24.04 (Noble)

```bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources
```

Replace all content with:

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

Then update:

```bash
sudo apt-get update
```

---

## Step 1 — Install System Dependencies

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

## Step 2 — Install wkhtmltopdf

wkhtmltopdf is required for PDF report generation. On Ubuntu 22.04/24.04 you must first install `libssl1.1`.

```bash
# Download libssl1.1 from domestic mirror
wget https://dl.odoo-community.ir/odoo-tools/libssl1.1_1.1.1f-1ubuntu2_amd64.deb -O /tmp/libssl1.1.deb
sudo dpkg -i /tmp/libssl1.1.deb

# Download and install wkhtmltopdf
wget https://dl.odoo-community.ir/odoo-tools/wkhtmltox_0.12.5-1.bionic_amd64.deb -O /tmp/wkhtmltox.deb
sudo gdebi --n /tmp/wkhtmltox.deb
```

Verify:

```bash
wkhtmltopdf --version
```

---

## Step 3 — Install Node.js & RTL Support

Odoo needs Node.js for asset compilation and `rtlcss` for Persian/Arabic RTL layout.

```bash
sudo apt-get install nodejs npm -y

# Install rtlcss using domestic npm mirror
sudo npm install -g rtlcss --registry=https://mirror2.chabokan.net/npm/
```

---

## Step 4 — Create Odoo System User

```bash
sudo adduser --system --quiet --shell=/bin/bash --home=/opt/odoo19 --group odoo19
```

---

## Step 5 — Create PostgreSQL User

```bash
sudo su - postgres -c "createuser -s odoo19 -P"
```

> You will be prompted to set a password. Keep it for the config file in Step 8.

---

## Step 6 — Download Odoo 19 Source

```bash
# Create directories
sudo mkdir -p /opt/odoo19 /etc/odoo /var/log/odoo19
sudo chown -R odoo19:odoo19 /opt/odoo19 /etc/odoo /var/log/odoo19

# Switch to odoo19 user
sudo su - odoo19

# Download source archive
wget https://dl.odoo-community.ir/odoo-source/odoo19/src_19e.tar.gz

# Extract
tar --strip-components=1 -zxf src_19e.tar.gz

# Create odoo-bin entry point
cat > /opt/odoo19/odoo-bin << 'EOF'
#!/usr/bin/env python3
__import__('os').environ['TZ'] = 'UTC'
import odoo
if __name__ == "__main__":
    odoo.cli.main()
EOF
chmod +x /opt/odoo19/odoo-bin

# Create custom addons directory
mkdir -p /opt/odoo19/custom_addons

# Clean up archive
rm src_19e.tar.gz

exit
```

---

## Step 7 — Set Up Python Virtual Environment

Install Python dependencies inside a virtual environment using domestic PyPI mirrors.

```bash
sudo su - odoo19
cd /opt/odoo19

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install requirements from primary domestic mirror
pip install \
  --trusted-host mirror-pypi.runflare.com \
  -i https://mirror-pypi.runflare.com/simple/ \
  -r requirements.txt

# If primary fails, use alternate mirror:
# pip install \
#   --trusted-host mirror.chabokan.net \
#   -i https://mirror.chabokan.net/repository/pypi-proxy/simple \
#   -r requirements.txt

# Install additional required packages
pip install psycopg2-binary==2.9.9 pyopenssl==23.2.0

deactivate
exit
```

---

## Step 8 — Create Odoo Configuration File

Replace `YOUR_DB_PASSWORD` with the PostgreSQL password you set in Step 5.

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

> ⚠️ Change `admin_passwd` to a strong random string. This is the Odoo master password.

---

## Step 9 — Create Systemd Service

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

## Step 10 — Install Nginx (Optional)

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

## Verification

```bash
# Check service status
sudo systemctl status odoo19

# Watch logs
sudo tail -f /var/log/odoo19/odoo.log
```

Open your browser and navigate to: **`http://YOUR_SERVER_IP:8069`**

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `apt update` fails | Check mirror URLs in sources.list |
| pip install timeout | Try alternate mirror: `mirror.chabokan.net` |
| wkhtmltopdf PDF error | Verify `libssl1.1` is installed |
| Odoo won't start | Check logs: `journalctl -u odoo19 -n 50` |
| DB connection error | Verify PostgreSQL password matches `odoo19.conf` |
| Port 8069 not accessible | Run `sudo ufw allow 8069` |

---

## Security Notes

> ⚠️ Before going to production:
> 1. Change `admin_passwd` in `/etc/odoo/odoo19.conf` to a strong random value
> 2. Change the PostgreSQL `odoo19` user password
> 3. Configure a firewall (`ufw`)
> 4. Do **not** expose port 8069 publicly — use Nginx

---

## Domestic Download Links

| File | URL |
|------|-----|
| Odoo 19 Source | `https://dl.odoo-community.ir/odoo-source/odoo19/src_19e.tar.gz` |
| wkhtmltopdf | `https://dl.odoo-community.ir/odoo-tools/wkhtmltox_0.12.5-1.bionic_amd64.deb` |
| libssl1.1 | `https://dl.odoo-community.ir/odoo-tools/libssl1.1_1.1.1f-1ubuntu2_amd64.deb` |
| PyPI Mirror | `https://mirror-pypi.runflare.com/simple/` |
| PyPI Mirror (alt) | `https://mirror.chabokan.net/repository/pypi-proxy/simple` |
| npm Mirror | `https://mirror2.chabokan.net/npm/` |
