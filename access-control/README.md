存取控制
==============

當談到存取控制時，第一步就是只使用被信任的系統物件來存取授權決定。在 [Session Management][3] 中的例子中，我們使用 JWT。JSON Web token 來產生伺服器端的 token。

```go
// 建立 JWT token 並放在客戶端的 cookie 中
func setToken(res http.ResponseWriter, req *http.Request) {
    // 30m Expiration for non-sensitive applications - OWASP
    expireToken := time.Now().Add(time.Minute * 30).Unix()
    expireCookie := time.Now().Add(time.Minute * 30)

    // token Claims
    claims := Claims{
        {...}
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    signedToken, _ := token.SignedString([]byte("secret"))
```

我們可以使用這個 token 來驗證使用者，並且執行控制訪問的模式。

用來進行權限控制的元件應該只有單一一個，並且是全站共用。這包含呼叫外部授權服務的函式庫等。

萬一發生故障時，訪問控制應該安全的失敗，在 Go 中，我們可以使用 `defer` 來實現。
更多細節可以參考 [Error Logging][1] 中的章節。

如果一個應用程式無存取他的設定資訊，所有對於該應用程式的請求應該被拒絕。

在每個請求上都應該執行存取控制，包含伺服器端的程式和客戶端的 AJAX 請求。

同樣的，將控制權限的程式碼和其他的程式碼區別開來也是很重要的。

其他關於防止未經授權的使用者存取你的應用程式有：

* 檔案和其他資源。
* 被保護的 URL。
* 
* Direct object references
* Services
* Application data
* User and data attributes and policy information.

In the provided sample, a simple direct object reference is tested. This code
is built upon the [sample in the Session Management][2].

When implementing these access controls, it's important to verify that the
server-side implementation and the presentation layer representations of access
control rules match.

If _state data_ needs to be stored on the client-side, it's necessary to use
encryption and integrity checking in order to prevent tampering.

Application logic flow must comply with the business rules.

When dealing with transactions, the number of transactions a single user or
device can perform in a given period of time must be above the business
requirements but low enough to prevent a user from performing a DoS type
attack.

It is important to note that using only the `referer` HTTP header is
insufficient to validate authorization, and should only be used as a
supplemental check.

Regarding long authenticated sessions, the application should periodically
re-evaluate the user's authorization to verify that the user's permissions
have not changed. If the permissions have changed, log the user out and force
them to re-authenticate.

User accounts should also have a way to audit them, in order to comply with
safety procedures. (e.g. Disabling a user's account 30 days after the
password's expiration date).

The application must also support the disabling of accounts and the termination
of sessions when a user's authorization is revoked. (e.g. Role change,
employment status, etc.).

When supporting external service accounts and accounts that support connections
_from_ or _to_ external systems, these accounts must run on the lowest level of
privilege possible.

[1]: /error-handling-logging/error-handling.md
[2]: URL.go
[3]: /session-management/README.md
