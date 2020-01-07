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

## HTS模擬題目

## Contribution
[clickme](https://hackmd.io/EkD3WmKyQgexSx-1F85Mkg)
