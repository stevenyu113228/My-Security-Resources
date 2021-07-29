# Anthem
> URL : https://tryhackme.com/room/anthem

IP : 10.10.18.8
- 這題感覺有點廢，但還是寫一下WP好ㄌ
- 機器開機要等將近5分鐘
	- 為什麼?
	- 不知道，反正我等了5分鐘才有畫面
	- 估計是因為 Windows 有點肥ㄅ
## Recon
- 老梗 nmap -A 
	```
	Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-28 21:42 EDT
	Nmap scan report for 10.10.18.8
	Host is up (0.28s latency).
	Not shown: 998 filtered ports
	PORT     STATE SERVICE       VERSION
	80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
	| http-robots.txt: 4 disallowed entries 
	|_/bin/ /config/ /umbraco/ /umbraco_client/
	|_http-title: Anthem.com - Welcome to our blog
	3389/tcp open  ms-wbt-server Microsoft Terminal Services
	| rdp-ntlm-info: 
	|   Target_Name: WIN-LU09299160F
	|   NetBIOS_Domain_Name: WIN-LU09299160F
	|   NetBIOS_Computer_Name: WIN-LU09299160F
	|   DNS_Domain_Name: WIN-LU09299160F
	|   DNS_Computer_Name: WIN-LU09299160F
	|   Product_Version: 10.0.17763
	|_  System_Time: 2021-07-29T01:42:55+00:00
	| ssl-cert: Subject: commonName=WIN-LU09299160F
	| Not valid before: 2021-07-28T01:38:13
	|_Not valid after:  2022-01-27T01:38:13
	|_ssl-date: 2021-07-29T01:43:01+00:00; +1s from scanner time.
	```
	- 有開 80、3389
	- 有抓到 robots.txt

- robots.txt
	- ```
		UmbracoIsTheBest!

		# Use for all search robots
		User-agent: *

		# Define the directories not to crawl
		Disallow: /bin/
		Disallow: /config/
		Disallow: /umbraco/
		Disallow: /umbraco_client/
	  ```
	- 看到一組奇怪密碼 `UmbracoIsTheBest!`
	- 還有叫做 umbraco 的東西
		- Google 後發現他是一種 CMS
	  
## 回答問題
### 普通問題
- What port is for the web server?
	- `80`
- What port is for remote desktop service?
	- `3389`
- What is a possible password in one of the pages web crawlers check for?
	- 他都說 crawlers 了，所以應該就是 `robots.txt` 的密碼
	- `UmbracoIsTheBest!`
- What CMS is the website using?
	- `umbraco`
	- robots.txt 上有寫
-  What is the domain of the website? 
	-  `anthem.com`
	-  首頁上就有
### 通靈問題
- What's the name of the Administrator
	- CMS 中有一篇文章這樣寫
	- http://10.10.18.8/archive/a-cheers-to-our-it-department/
	- ![](https://i.imgur.com/Q0YQ938.png)
	-
		```
		Born on a Monday,
		Christened on Tuesday,
		Married on Wednesday,
		Took ill on Thursday,
		Grew worse on Friday,
		Died on Saturday,
		Buried on Sunday.
		That was the end…                    
		```
	- 把字串丟去 Google 可以找到這篇文
		- https://en.wikipedia.org/wiki/Solomon_Grundy_(nursery_rhyme)
		- 所以 admin 叫做 `Solomon_Grundy`

- Can we find find the email address of the administrator?
	- 在某篇貼文中
		- http://10.10.18.8/archive/we-are-hiring/
	- ![](https://i.imgur.com/bijPypa.png)
	- 貼文者叫做 `Jane Doe`
	- Email 是 : `JD@anthem.com`
		- 看起來規則是姓名各取一個字，都大寫 `@anthem.com`
	- 那 admin 這個
		- Solomon_Grundy 就 SG ㄅ 
	- `SG@anthem.com`

## Flag 們
- 我覺得這邊的 Flag 也都偏通靈
	- 沒有任何 web 技巧可言
	- 就把整個網站繞一圈就能逛完
- Flag1
	- http://10.10.18.8/archive/we-are-hiring/
	- ![](https://i.imgur.com/CzYJ4C4.png)
	- `THM{L0L_WH0_US3S_M3T4}`
- Flag2
	- http://10.10.18.8/
	- ![](https://i.imgur.com/IOPlHbO.png)
	- `THM{G!T_G00D}`
- Flag4
	- http://10.10.18.8/archive/a-cheers-to-our-it-department/
	- ![](https://i.imgur.com/ONgiSFD.png)
	- `THM{AN0TH3R_M3TA}`
- Flag3
	- http://10.10.18.8/authors/jane-doe/
	- THM{L0L_WH0_D15}
	- ![](https://i.imgur.com/eaHzgde.png)
	


## 通靈登入
- 通靈登入
	- 使用帳號 : SG@anthem.com
	- 密碼 : UmbracoIsTheBest!
		- 前面 robots.txt 找到ㄉ
	- 登入 http://10.10.18.8/umbraco
	- ![](https://i.imgur.com/MOvk1MR.png)
	
- 通靈RDP
	- `sudo apt install freerdp2-x11`
	- 帳號 : `SG`
	- 密碼 : `UmbracoIsTheBest!`
	- `xfreerdp +drives /u:SG /v:10.10.18.8:3389`
- 桌面上就有 user.txt
	- ![](https://i.imgur.com/bslatpD.png)
	- `THM{N00T_NO0T}`

## 提權
- 開啟顯示隱藏檔案
- 逛到 `c:\backup\restore`
	- ![](https://i.imgur.com/depc0nK.png)
- 他沒有讀取權限，但我們可以修改他的權限
	- ![](https://i.imgur.com/2gQDd6c.png)
	- ![](https://i.imgur.com/36JBMgC.png)
- 修改後點開可以看到以下字串
	- `ChangeMeBaby1MoreTime`
	- 猜測他可能是 admin 密碼
- 使用RDP連 Admin
	- `xfreerdp +drives /u:Administrator /v:10.10.18.8:3389`
	- 使用密碼 `ChangeMeBaby1MoreTime`
- 取得 admin 權限
	- ![](https://i.imgur.com/zlObe8V.png)
	- `THM{Y0U_4R3_1337}`