# Grandpa 
- URL : https://app.hackthebox.eu/machines/13
- IP : 10.129.2.85

## Recon
- Rustscan
    - ![](https://i.imgur.com/fIUBhTc.png)
    - 開 80 port
- nmap
    - 發現是 IIS6 可以用 CVE-2017-7269 RCE
    - https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269
## Exploit
- 執行 Exploit
    - ![](https://i.imgur.com/i3boH3K.png)
- 開 nc 收 shell
    - ![](https://i.imgur.com/yPKDwiV.png)
- 跑 systeminfo
    - ![](https://i.imgur.com/tbw8skf.png)
- 發現 Windows 2003 可以跑 Churrasco
    - https://github.com/Re4son/Churrasco
- 用 `impacket-smbserver meow .` 開
    - ![](https://i.imgur.com/MU2TXX1.png)
    - 執行 Churrasco
- 提權完畢
    - ![](https://i.imgur.com/UrZVB2K.png)
- 取得 User Flag
    - `bdff5ec67c3cff017f2bedc146a5d869`
- 取得 Root Flag
    - ![](https://i.imgur.com/gQePLVE.png)
    - `9359e905a2c35f861f6a57cecf28bb7b`