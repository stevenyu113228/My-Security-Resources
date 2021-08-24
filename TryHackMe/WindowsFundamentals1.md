# Windows Fundamentals 1
> URL : https://tryhackme.com/room/windowsfundamentals1xbx
## Task 1 Introduction to Windows 
- 可以用 RDP 之類的方法進入 Windows
	- 預設帳號 : `administrator`
	- 預設密碼 : `letmein123!`
- 但我直接用 THM 提供的 web RDP 介面
	- ![](https://i.imgur.com/l3kfrD2.png)

## Task 2 Windows Editions 
- Home 版本 跟 Pro 的差別
	- 可以使用 BitLocker 
	- 可以使用 WIP 
	- ![](https://i.imgur.com/j4oLsSC.png)
	- https://www.microsoft.com/en-us/windows/compare-windows-10-home-vs-pro
- What encryption can you enable on Pro that you can't enable in Home? 
	- A: BitLocker
## Task 3 The Desktop (GUI) 
-  Which selection will hide/disable the Search box? 
	-  ![](https://i.imgur.com/dCK3KO7.png)
	- Hidden
- Which selection will hide/disable the Task View button?
	- ![](https://i.imgur.com/f05fvL6.png)
	- Show Task View button
- Besides Clock, Volume, and Network, what other icon is visible in the Notification Area?
	- ![](https://i.imgur.com/HYsPpK2.png)
	- action center

## Task 4 The File System 
- 檔案系統
	- FAT16 / FAT32
		- File Allocation Table
		- USB 裝置之類ㄉ還是看ㄉ到
	- HPFS 
		- High Performance File System
	- NTFS
		- New Technology File System 
- NTFS 優勢
	- 文件紀錄系統，可以自動修復日誌
	- 單檔案 >= 4G
	- 可以針對資料夾跟黨按設定權限
	- 資料夾跟檔案的壓縮
	- 加密 (Encryption File System, EFS)
- ![](https://i.imgur.com/K48VenM.png)
- NTFS 權限
	- Read
		- 對資料夾 ： 可以觀看，跟顯示資料夾中的檔案與子資料夾
		- 對檔案：可以觀看或存取檔案的內容
	- Write
		- 對資料夾 ： 可以在資料夾中增加檔案或子資料夾
		- 對檔案：可以對檔案進行寫入
	- Read & Execute
		- 對資料夾 ： 可以觀看，顯示資料夾中的檔案與子資料夾；可以直行檔案，會繼承給檔案以及資料夾
		- 對檔案：可以觀看、存取檔案的內容，也可以直行黨按
	- List folder contents
		- 對資料夾：可以觀看，並列出檔案以及子資料夾，以及執行檔案，只能給 資料夾繼承
	- Modify
		- 對資料夾：可以讀取、寫入檔案以及子資料夾；允許刪除資料夾
		- 對檔案：可以讀取，寫入檔案；允許刪除檔案
	- Full Control
		- 對資料夾，允許讀取、寫入、變更以及刪除檔案與子資料夾
		- 對檔案：允許讀取、寫入、變更予刪除檔案
### Alternate Data Streams (ADS)
- 一種 NTFS 的 attribute 
- 每一個檔案至少都會有一個 Data Stream ($DATA)
- ADS 允許一個檔案包含多個 Data Stream
- Windows Explorer 不會對使用者顯示 ADS，需要透過特定的第三方軟體或是 Powershell
- 所以有時候 Malware 會使用 ADS 來隱藏資料
- 延伸閱讀 https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/
- 範例
	- 建立一個 ADS 的東東
		- ![](https://i.imgur.com/5kOtSRd.png)
	- 用 Dir 看不出來
		- ![](https://i.imgur.com/jCrO2X8.png)
	- 用 Type 也看不出來
		- ![](https://i.imgur.com/ekfM1mN.png)
	- 可以用 powershell ` GET-Item -path C:\Users\Administrator\Desktop\example -stream *`
		- ![](https://i.imgur.com/zgKdKdH.png)
	- CMD 需要裝 Sysinternals 的 Streams 
		- https://docs.microsoft.com/zh-tw/sysinternals/downloads/streams
-  What is the meaning of NTFS? 
	-  New Technology File System

## Task 5 The Windows\System32 Folders 
- `%windir%` 
	- 最常見的位子在 `C:\Windows`
	- 裡面存放著作業系統，但他不一定需要在 C，也有一些喪心病狂的人會把他設定在其他地方
	- 所以我們可以使用環境變數，或是說系統環境變數(System environment variables) 來正確的訪問
- `System32`
	- 資料夾中有許多對作業系統來說很重要的檔案
	- 在變動、刪除時要特別小心，很可能讓系統爛掉
-  What is the system variable for the Windows folder? 
	-  `%windir%`
## Task 6 User Accounts, Profiles, and Permissions 
- 兩種類型的帳戶
	- Administrator
		- 增、刪使用者
		- 修改 Group
		- 修改作業系統設定 
	- Standard User
		- 修改與該使用者相關的資料夾、檔案的屬性
		- 不能做系統層級的修改，例如安裝軟體
- 使用者家目錄
	- `C:\Users\{Username}`
	- 預設會有
		- Desktop
		- Documents
		- Downloads
		- Music
		- Pictures
- Local User and Group Management
	- `lusrmgr.msc`
	- ![](https://i.imgur.com/fyKUJjj.png)
	- 可以看到各種使用者、群組的資訊
- What is the name of the other user account?
	- tryhackmebilly
- What groups is this user a member of?
	- ![](https://i.imgur.com/UG7fYkT.png)
	- `Remote Desktop Users,Users`
-  What built-in account is for guest access to the computer? 
	-  Guest
-  What is the account status?
	- ![](https://i.imgur.com/HaK4mu6.png)
	- Account is disabled
## Task 7 User Account Control 
- UAC
	- User Account Control 
- Vista 後開始使用的
- 可以觀察到一個安裝檔案可能並沒有使用者相關的權限
	- ![](https://i.imgur.com/Upd8XM6.png)
- 登入一個普通使用者
	- `tryhackmebilly` / `window$Fun1!`
	- 10.10.229.121
	- ![](https://i.imgur.com/LW8bHbS.png)
	- 會發現多出盾牌 icon
- 如果執行安裝
	- 會需要輸入 Administrator 的密碼
	- ![](https://i.imgur.com/w5JAACI.png)
- What does UAC mean? 
	-  User Account Control 
## Task 8 Settings and the Control Panel 
- Win8 後出現 Settings 頁面
	- ![](https://i.imgur.com/AnP3CPq.png)
- 舊版控制台 (Control Panel)
	- ![](https://i.imgur.com/xyxW5OJ.png)
-  In the Control Panel, change the view to Small icons. What is the last setting in the Control Panel view? 
	-  ![](https://i.imgur.com/VyOGwm5.png)
	- 右上角選 small icons
	- 最後一個選項是 Windows Defender Firewall
## Task 9 Task Manager 
- What is the keyboard shortcut to open Task Manager? 	
	- `ctrl+shift+esc`
	- 哇酷，我還真不知道有這種東西呢
## Task 10 Conclusion 
- 喵喵