# ERPNext v16 + Frappe HR — Rocky Linux 9 Deployment Runbook

Adapted from the existing Ubuntu 24.04 deployment (`erp.jakir.online`). Every step below is a Rocky Linux equivalent of what that deployment needed — package manager, MariaDB auth, supervisor, nginx, and SELinux all differ from Ubuntu, so don't just `apt`→`dnf` swap the old runbook.

All commands below run **on the Rocky Linux 9 server** (EC2 or otherwise). Where a command must run as a specific user, it's called out.

---

## 1. Base system update and tools

**Rocky Linux server, as your sudo user:**

```bash
sudo dnf update -y
sudo dnf install -y epel-release
sudo dnf config-manager --set-enabled crb
sudo dnf install -y git curl wget nano policycoreutils-python-utils
```

`crb` (CodeReady Builder) is Rocky's equivalent of Ubuntu's `universe`/`multiverse` — several devel packages later need it enabled.

---

## 2. Python 3.11

Rocky 9's default `python3` is 3.9, which is too old for Frappe v16 (needs 3.11+). Rocky ships `python3.11` directly in AppStream — no extra repo needed.

```bash
sudo dnf install -y python3.11 python3.11-devel python3.11-pip
python3.11 --version
python3.11 -m venv --help
```

---

## 3. MariaDB 10.11

Rocky's AppStream default MariaDB stream is 10.5, below what Frappe v16 wants. Add the official MariaDB repo and install from there instead of the module stream.

```bash
sudo tee /etc/yum.repos.d/mariadb.repo > /dev/null << 'EOF'
[mariadb]
name = MariaDB
baseurl = https://rpm.mariadb.org/10.11/rhel/$releasever/$basearch
gpgkey = https://rpm.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck = 1
EOF

sudo dnf install -y MariaDB-server MariaDB-client MariaDB-devel MariaDB-shared
sudo systemctl enable --now mariadb
sudo mysql_secure_installation
```

Set the required charset/collation for Frappe:

```bash
sudo tee /etc/my.cnf.d/frappe.cnf > /dev/null << 'EOF'
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
EOF

sudo systemctl restart mariadb
```

**Auth plugin — this is the Rocky equivalent of the MariaDB auth issue from the Ubuntu deployment.** The MariaDB upstream repo build defaults `root` to `unix_socket` auth, but `bench new-site` needs a password-based root login:

```bash
sudo mysql -u root << 'EOF'
ALTER USER 'root'@'localhost' IDENTIFIED VIA mysql_native_password USING PASSWORD(PASSWORD('REPLACE_WITH_ROOT_PASSWORD'));
FLUSH PRIVILEGES;
EOF
```

---

## 4. Redis

```bash
sudo dnf install -y redis
sudo systemctl enable --now redis
```

---

## 5. Node.js and Yarn (via NVM)

Frappe v16 wants Node 20+.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20
npm install -g yarn
```

---

## 6. wkhtmltopdf (patched Qt build)

The generic distro/EPEL wkhtmltopdf is not the patched-Qt build and breaks PDF print formats — this was one of the Ubuntu deployment issues. Rocky has no native wkhtmltopdf package at all, so use the upstream RPM. There's no Rocky-labeled release; the AlmaLinux 9 build is RHEL-family compatible and works on Rocky 9.

```bash
sudo dnf install -y libXrender libXext fontconfig freetype libpng libjpeg-turbo xorg-x11-fonts-Type1 xorg-x11-fonts-75dpi
cd /tmp
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox-0.12.6.1-3.almalinux9.x86_64.rpm
sudo dnf localinstall -y wkhtmltox-0.12.6.1-3.almalinux9.x86_64.rpm
wkhtmltopdf --version
wkhtmltoimage --version
```

---

## 7. supervisor and nginx

```bash
sudo dnf install -y nginx supervisor
sudo systemctl enable nginx
sudo systemctl enable supervisord
```

**Note the service name is `supervisord`** on Rocky (EPEL package), not `supervisor` as on Ubuntu. `bench setup production` generates a config file for supervisor to load — after that step, restart with `sudo systemctl restart supervisord`, not `supervisor`.

---

## 8. Create the frappe user and install bench

**Rocky Linux server, as your sudo user:**

```bash
sudo useradd -m -s /bin/bash frappe
sudo usermod -aG wheel frappe
sudo su - frappe
```

**Rocky Linux server, as the `frappe` user (after `su`):**

```bash
python3.11 -m pip install --user frappe-bench
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
source ~/.bashrc
bench --version
```

---

## 9. Init the bench, pinned to Python 3.11

**Rocky Linux server, as the `frappe` user:**

```bash
bench init frappe-bench --frappe-branch version-16 --python python3.11
cd frappe-bench
```

---

## 10. Get ERPNext and Frappe HR

```bash
bench get-app erpnext --branch version-16
bench get-app hrms --branch version-16
```

---

## 11. Create the new site

Replace `erp.jakir.online` if this is a different host/domain than the Ubuntu deployment.

```bash
bench new-site erp.jakir.online --mariadb-root-password REPLACE_WITH_ROOT_PASSWORD --admin-password REPLACE_WITH_ADMIN_PASSWORD
bench --site erp.jakir.online install-app erpnext
bench --site erp.jakir.online install-app hrms
```

---

## 12. Set up production

**Rocky Linux server, back as your sudo user** (exit the `frappe` shell first, `bench setup production` needs sudo):

```bash
exit
cd /home/frappe/frappe-bench
sudo bench setup production frappe --yes
```

**If this fails with an `nginx_vhosts` traceback** — this is the Rocky-side recurrence of the ansible-core bug from the Ubuntu deployment, since `bench setup production` still shells out to ansible regardless of distro:

```bash
python3.11 -m pip install --user "ansible-core<2.16" --force-reinstall
sudo bench setup production frappe --yes
```

---

## 13. SELinux (no Ubuntu equivalent — new step)

Rocky enforces SELinux by default, unlike the Ubuntu deployment where AppArmor didn't need touching for this stack. Without policy adjustments, nginx will be blocked from proxying to the bench sockets/gunicorn.

```bash
sudo setsebool -P httpd_can_network_connect 1
sudo semanage fcontext -a -t httpd_sys_content_t "/home/frappe/frappe-bench/sites(/.*)?"
sudo restorecon -Rv /home/frappe/frappe-bench/sites
```

If nginx still can't reach the backend after this, check `sudo ausearch -m avc -ts recent` for denials and add the specific rule it reports.

---

## 14. Firewall (firewalld instead of ufw)

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

---

## 15. Start and verify

```bash
sudo systemctl restart supervisord
sudo systemctl restart nginx
sudo supervisorctl status
```

All processes should show `RUNNING`. Then hit the site in a browser to confirm the login page loads.

---

## Open items to confirm during the actual run

- **MariaDB root auth** — if `ALTER USER ... IDENTIFIED VIA mysql_native_password` errors on your MariaDB 10.11 build, paste the exact error here and this gets updated.
- **ansible-core version pin** — the `<2.16` pin above is carried over from the Ubuntu issue; confirm it's still the correct ceiling once you hit the actual traceback (if any) on this bench version.
- **SELinux denials** — the two rules in step 13 cover the common nginx-proxy case; specific AVC denials from your run may need additional `semanage`/`setsebool` rules not listed here.
