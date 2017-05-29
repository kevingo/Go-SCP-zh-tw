通訊安全
======================

當談到通訊安全時，開發者應該確保其所使用的通道是安全的。通訊的種類包含：伺服器-客戶端、資料庫和所有其他後端的通訊。這些通訊應該要被加密，並且確保資料的一致性。當你沒有確保通訊的安全時，很可能會遭受像 MITM 攻擊(譯注：Man-in-the-middle attack 中間人攻擊)，讓攻擊者可以攔截雙方的通訊。

本章節會談論到以下兩種通訊模式：

* HTTP/TLS
* Websockets

[1]: https://www.owasp.org/index.php/Man-in-the-middle_attack
