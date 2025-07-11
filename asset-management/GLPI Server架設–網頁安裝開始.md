---
title: GLPI Server架設–網頁安裝開始

---

**GLPI Server架設--網頁安裝開始**
**1. 按下OK-->繼續-->安裝**
![2025-07-11_121348](/pics/glpi/001.png)
![2025-07-11_121537](/pics/glpi/002.png)
![2025-07-11_121616](/pics/glpi/003.png)
**跳出錯誤**
![2025-07-11_121645](/pics/glpi/004.png)
**解決方式：先對 config/ 資料夾給「暫時寫入權限」**
```
chmod -R 777 /var/www/html/glpi/config
```
**並且手動建立空檔案後再給兩個檔案寫入權限**
```
touch /var/www/html/glpi/config/config_db.php
touch /var/www/html/glpi/config/glpicrypt.key

chown apache:apache /var/www/html/glpi/config/config_db.php
chown apache:apache /var/www/html/glpi/config/glpicrypt.key

chmod 666 /var/www/html/glpi/config/config_db.php
chmod 666 /var/www/html/glpi/config/glpicrypt.key
```
**⏭️ 等最後安裝完成後記得恢復權限：**
```
chmod 644 /var/www/html/glpi/config/config_db.php
chmod 640 /var/www/html/glpi/config/glpicrypt.key
chmod 750 /var/www/html/glpi/config
```
**套用正確 SELinux 寫入 context：**
```
chcon -t httpd_sys_rw_content_t /var/www/html/glpi/config/config_db.php
chcon -t httpd_sys_rw_content_t /var/www/html/glpi/config/glpicrypt.key
chcon -R -t httpd_sys_rw_content_t /var/www/html/glpi/config
chcon -R -t httpd_sys_rw_content_t /var/www/html/glpi/files
```
**🔄 然後，重新啟動 Apache：**
```
systemctl restart httpd
```
**2. 成功進入步驟 0：環境相容性檢查**
![2025-07-11_135242](/pics/glpi/005.png)
**第一個待解決問題**
![2025-07-11_140906](/pics/glpi/006.png)
SELinux 的某些布林開關（boolean）未開啟，可能會導致特定功能無法使用。
| 項目                             | 意義                         | 影響功能                           | 建議              |
| ------------------------------ | ------------------------      | -------------------------        | --------------- |
| `httpd_can_network_connect`    | Apache 可否連外網               | 例如 GLPI 發送 webhook、API 整合    | ✅ 建議開啟          |
| `httpd_can_network_connect_db` | Apache 可否連資料庫（非本機）     | 如果你 DB 在另一台機器               | ❌ 若 DB 在同機器可不用開 |
| `httpd_can_sendmail`           | Apache 可否寄信（透過 sendmail） | SMTP、告警通知                      | ✅ 建議開啟          |

```
setsebool -P httpd_can_network_connect on
setsebool -P httpd_can_network_connect_db on
setsebool -P httpd_can_sendmail on
```
**第二個待解決問題**
![2025-07-11_142850](/pics/glpi/007.png)
⚠️ PHP 沒有安裝或啟用 LDAP 擴充套件（extension），因此無法使用 LDAP 認證功能。
```
php -v
dnf install php-ldap
systemctl restart httpd
```
**第三個待解決問題**
![2025-07-11_143340](/pics/glpi/008.png)
⚠️GLPI 嘗試寫入 /var/www/html/glpi/marketplace 資料夾時失敗，無法在該目錄建立檔案或資料夾，因此 無法從 Marketplace 安裝 Plugin
```
chmod 755 /var/www/html/glpi/marketplace
chcon -R -t httpd_sys_rw_content_t /var/www/html/glpi/marketplace
systemctl restart httpd
```
按下繼續
![2025-07-11_143755](/pics/glpi/009.png)
**3. 進入步驟 1：資料庫連接設定**
輸入帳密
MySQL/MariaDB 伺服器: localhost
使用者名稱: glpiuser
密碼: 之前設定的「強密碼」
![2025-07-11_144602](/pics/glpi/010.png)
**4. 進入步驟 2：測試接資料庫**
選擇已建立的glpidb
![2025-07-11_144905](/pics/glpi/011.png)
**5. 進入步驟 3：初始化資料庫**
![2025-07-11_144959](/pics/glpi/012.png)
**6. 進入步驟 4：選擇是否傳送分析資料**
![2025-07-11_145116](/pics/glpi/013.png)
**7. 進入步驟 5：按下繼續**
![2025-07-11_145213](/pics/glpi/014.png)
**6. 進入步驟 6：按下使用GLPI**
![2025-07-11_145213](/pics/glpi/015.png)
**9. 進入GLPI管理介面**
預設使用者名稱及密碼皆為glpi
![2025-07-11_145518](/pics/glpi/016.png)
