---
title: GLPI Server架設

---

**GLPI Server架設**
**1. Install OS: Rocky Linux 9 Minimal**
```
Rocky-9.6-x86_64-minimal.iso
```
**2. Update system**
```
dnf update -y
```
**3. Install tools**
```
dnf install -y epel-release vim wget curl unzip git net-tools
```
**4. 安裝 Nginx + MariaDB + PHP（OCS/GLPI 需要）**
```
dnf install -y nginx mariadb-server mariadb
```
**5. 安裝 PHP**
```
dnf install php php-fpm php-mysqlnd php-xml php-gd php-curl php-mbstring php-zip php-cli php-soap php-intl php-json php-bcmath
```
**6. 啟用並開啟服務自動啟動**
```
sudo systemctl enable --now nginx
sudo systemctl enable --now mariadb
sudo systemctl enable --now php-fpm
```
**7. 防火牆開放 port 80, 443**
```
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
**8. 安全化 MariaDB 安裝**
```
sudo mysql_secure_installation
```
**Q1. MariaDB 安裝時已經使用 unix_socket 機制來保護 root 帳號，因此可以不要改變目前的認證機制。**
```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
```
**Q2. 「刪除匿名使用者」。以提升資料庫安全性，避免任何人在沒有帳號的情況下連進資料庫。**
```
By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
```
**Q3. 「不允許 root 從遠端登入」。以提升資料庫安全性，避免任何人在沒有帳號的情況下連進資料庫。**
```
Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
```
**Q4. 「刪除 test 資料庫」。 MariaDB 預設建立的測試用資料庫。**
```
By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
```
**Q5. 「讓剛剛所有的設定立即生效」。**
```
Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
```
**9. 再次確認PHP 及常用模組是否皆已安裝**
```
dnf install php php-cli php-fpm php-mysqlnd php-xml php-mbstring php-gd php-curl php-json php-intl php-opcache php-zip -y
```
**10. 設定 Nginx config**
```
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup
```
```
vim /etc/nginx/nginx.conf
```
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile        on;
    keepalive_timeout 65;

    server {
        listen       80 default_server;
        server_name  _;

        root /var/www/html/glpi;
        index index.php index.html index.htm;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
            try_files $uri =404;
            include fastcgi_params;
            fastcgi_index index.php;
            fastcgi_pass unix:/run/php-fpm/www.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param DOCUMENT_ROOT $document_root;
        }


        location ~ /\.ht {
            deny all;
        }
    }
}
```
**11. 建立 GLPI 網站目錄**
```
sudo mkdir -p /var/www/html/glpi
sudo chown -R nginx:nginx /var/www/html/glpi
```
**12. 測試 Nginx 配置**
```
sudo nginx -t
```
**13. 重啟 Nginx**
```
sudo systemctl restart nginx
```
**14. 下載 GLPI 原始檔案**
```
cd /tmp
```
```
wget https://github.com/glpi-project/glpi/releases/tag/10.0.18/glpi-10.0.18.tgz
```
```
tar -xvzf glpi-10.0.18.tgz
```
```
mv glpi /var/www/html/
```
```
sudo chown -R apache:apache /var/www/html/glpi
sudo chmod -R 755 /var/www/html/glpi
```
**15. 建立 MariaDB 資料庫和帳號**
```
sudo mysql -u root -p
```
```
CREATE DATABASE glpidb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY '你的強密碼';
GRANT ALL PRIVILEGES ON glpidb.* TO 'glpiuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
```
sudo systemctl restart nginx
sudo systemctl restart php-fpm
```
**16. 開始網頁安裝**
```
http://192.168.5.11/glpi
```