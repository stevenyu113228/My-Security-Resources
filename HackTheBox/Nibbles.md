# Nibbles
- URL : https://app.hackthebox.eu/machines/121
- IP : 10.129.214.27


## Recon
- Rustscan
    - ![](https://i.imgur.com/m4G08Jy.png)
- 觀察首頁
    - ![](https://i.imgur.com/0jgF561.png)
    - ![](https://i.imgur.com/RYElMsf.png)
    - Apache
    - ![](https://i.imgur.com/IQdi1Cj.png)
        - 找到隱藏目錄 `/nibbleblog/`
- 看起來是一個 CMS
    - ![](https://i.imgur.com/KutA1td.png)
- 搜尋 Exploit
    - ![](https://i.imgur.com/2Mdxscn.png)
    - 看起來 4.0.3 版本有洞
    - 有帳密就可以傳東西
    - https://github.com/TheRealHetfield/exploits/blob/master/nibbleBlog_fileUpload.py
- 繼續掃目錄
    - http://10.129.214.27/nibbleblog/admin/
    - ![](https://i.imgur.com/cNJwr3I.png)
    - ![](https://i.imgur.com/azNBSX5.png)
        - 登入頁面
    - ![](https://i.imgur.com/PiHxr0E.png)
        - install
    - ![](https://i.imgur.com/YejpqvW.png)
        - update
- Google 到預設密碼
    - `admin / nibbles`
    - ![](https://i.imgur.com/lrj8MCV.png)
    - 這邊有點通靈 QQ
## Exploit
- 用 MSF
    - `msf multi/http/nibbleblog_file_upload`
- `spawn` shell
    - `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- 取得 User flag
    - ![](https://i.imgur.com/qHqBpzs.png)
## 提權
- 起手式
    - ![](https://i.imgur.com/Ryh3MeA.png)
        - 可以執行 `monitor.sh`
- 直接用`/bin/bash`蓋掉
    - ![](https://i.imgur.com/I6ka7fH.png)
- sudo 執行
    - ![](https://i.imgur.com/gTwWLBu.png)
    - 取得 Root
- 取得 Root Flag
    - ![](https://i.imgur.com/xYnSpZj.png)
    - `4c0ebef2dfb32a8d33212e1a902f50f2`
