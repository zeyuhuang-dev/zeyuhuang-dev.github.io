---
title: GLPI Server架設 Apache

---

**GLPI Server架設 Apache version**
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
dnf install -y epel-release vim wget curl unzip bash-completion git net-tools
```
**4. 安裝 Apache**
```
dnf install -y httpd
systemctl enable --now httpd
```
**5. 安裝 PHP**
```
dnf module reset php -y
dnf module enable php:8.1 -y
dnf install php php-fpm php-mysqlnd php-xml php-gd php-curl php-mbstring php-zip php-cli php-soap php-intl php-json php-bcmath
```
**6. 重啟 Apache 以套用 PHP：**
```
systemctl restart httpd
```
**7. 防火牆開放 port 80, 443**
```
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
**8. 安裝 MariaDB**
```
dnf install -y mariadb-server
systemctl enable --now mariadb
```
**9. 安全化 MariaDB 安裝**
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
**10. 再次確認PHP 及常用模組是否皆已安裝**
```
dnf install php php-cli php-fpm php-mysqlnd php-xml php-mbstring php-gd php-curl php-json php-intl php-opcache php-zip -y
```
**11. 建立 MariaDB 資料庫和帳號**
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
**12. 下載 GLPI 原始檔案**
```
cd /var/www/html
```
```
wget https://github.com/glpi-project/glpi/releases/tag/10.0.18/glpi-10.0.18.tgz
```
```
tar -xvzf glpi-10.0.18.tgz
```
```
rm -f glpi-10.0.18.tgz
```
```
sudo chown -R apache:apache /var/www/html/glpi
sudo chmod -R 755 /var/www/html/glpi
```
**13. 設定 Apache config**
```
vim /etc/httpd/conf.d/glpi.conf
```
```
<VirtualHost *:80>
    ServerName glpi2.ace-acc.com.tw
    DocumentRoot /var/www/html/glpi

    <Directory /var/www/html/glpi>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/glpi_error.log
    CustomLog /var/log/httpd/glpi_access.log combined
</VirtualHost>
```
```
systemctl restart httpd
```
**14. 開始網頁安裝**
```
http://glpi2.ace-acc.com.tw
```
