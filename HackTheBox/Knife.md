# Knife Writeup
###### tags: `htb`
> URL : https://app.hackthebox.eu/machines/347

IP : 10.10.69.238
## Recon
- 掃 Port
    - `rustscan -a 10.10.69.238 -r 1-65535`
        - ![](https://i.imgur.com/eytEe9s.png)
        - 有開 80 跟 443
- 觀察 Wappalyzer
    - ![](https://i.imgur.com/uJRIGKU.png)
        - Apache 2.4.41
        - PHP8.1.0
    - ![](https://i.imgur.com/E7SBRv4.png)
- 掃目錄
    - `python3 dirsearch.py -u http://10.10.10.242/`
    - 看起來沒有東西 QQ
        - ![](https://i.imgur.com/Kx8h2w7.png)


## Exploit
- 找到 php 8.1.0 有 RCE 漏洞
    - https://github.com/flast101/php-8.1.0-dev-backdoor-rce
    - `git clone https://github.com/flast101/php-8.1.0-dev-backdoor-rce`
- 執行起來
    - ![](https://i.imgur.com/pzJQzlP.png)
    - 就成功拿到 Shell ㄌ
- 也可以執行另外一個 reverse shell 版本
    - ![](https://i.imgur.com/4JZGBvV.png)
    - ![](https://i.imgur.com/qZkm9Zo.png)
- 取得 `user.txt`
    - ![](https://i.imgur.com/FWppblD.png)


## 提權
- 起手式 `sudo -l`
    - ![](https://i.imgur.com/xPJsZM9.png)
    - 發現我們可以用 `sudo knife`
- [GTFOBins](https://gtfobins.github.io/gtfobins/knife/#sudo) 尋找刀子的 sudo 提權
    - `sudo knife exec -E 'exec "/bin/sh"'`
    - 就提權成功ㄌ
    - ![](https://i.imgur.com/VHcpqD3.png)
- 取得 Root Flag
    - ![](https://i.imgur.com/UVMDqRH.png)

## 心得
這題提醒我們，還是需要優先確認各種服務的版本號。