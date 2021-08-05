# Windows Unquoted Services Path
## 簡介
顧名思義，這是一個 Windows 服務上面的漏洞。找到符合沒有加上引號的路徑很常見，但是真正能夠運用的機率不一定很多，因為前提是還需要注意相關路徑的寫入以及讀取權限。

- 使用前提：
	- 檔案的路徑中有空格
	- 而且檔案沒有用引號括起來
	- 對於該檔案的前一個目錄具有寫入的權限

達成這兩個前提就有機會可以實作 Unquoted Services Path 相關的漏洞應用，但通常在 XP 以後的作業系統，預設普通使用者沒有 `C:\` 的寫入權限，所以大多數狀況都不適用；而使用者也須要具備對於該 Services 的開啟與關閉權限。

- 先備知識，觀察權限
	- [AccessChk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) 
	- Services 權限
		- `accesschk.exe /accepteula -ucqv {services_name}`
		- ![](https://i.imgur.com/gmO6qI7.png)
	- 資料夾權限
		- `accesschk -uwdq "C:\Program Files (x86)"`
		- ![](https://i.imgur.com/3X20RSe.png)

## 觀察
### wmic
- 輸入 `wmic service get name,pathname`
	- 可以觀察到所有服務的名稱與檔案名稱
	- ![](https://i.imgur.com/Z2gk6iV.png)
- 輸入 `wmic service get pathname,startname`
	- 可以觀察到程式名稱與執行者名稱(startname)
	- ![](https://i.imgur.com/SuQyoda.png)
		- 在此， startname 若為 `LocalSystem` 則代表有 `system` 權限
### Winpeas
- Winpeas 也有這種相關提示
	- `No quotes and Space detectd`
	- ![](https://i.imgur.com/qC8v4Jb.png)

## 實作與解釋
假設我們看到一個 Path 如下所述
`C:\Program Files (x86)\meow\meowmeow 8.7\meoewww.exe`

- 我們可以發現路徑依序為
	- `Program Files (x86)`
		- Windows 會先嘗試解釋為 `Program.exe` 再解釋 `Program Files (x86)`
	- `meow`
	- `meowmeow 8.7`
		- Windows 會先嘗試解釋為 `meowmeow.exe` 再解釋 `meowmeow 8.7`
	- `meoewww.exe`

在這段路徑中，我們觀察路徑的空白會發現，有兩個可以利用的點，分別是路徑中有空白的`Program Files (x86)`以及`meowmeow 8.7`

透過截斷漏洞，我們假如建立一個檔案 `C:\Program.exe`，則會被優先執行。

而假如我們沒有`C:\`的寫入權限，而剛好有`C:\Program Files (x86)\meow\` 的寫入權限的話，我們也可以對`meowmeow 8.7`這個檔案下手，建立一個`C:\Program Files (x86)\meow\meowmeow.exe`對系統進行誤導。


惡意檔案(如Revrese shell)在指定的位置與名稱建立完畢後，把服務重啟即可自動運行。

- 關閉服務指令
	- `sc stop {services_name}`
- 開啟服務指令
	- `sc start {services_name}`

## Reference
- https://gracefulsecurity.com/privesc-unquoted-service-path/