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
* 受保護的 URL。
* 受保護的函式
* 直接物件參考
* 服務
* 應用程式資料
* 使用者和策略資訊

在提供的範例中，我們會測試一個簡單的物件參考。相關的程式碼會放在 [sample in the Session Management][2] 中。

當實作訪問控制時，要確保伺服器端的控制邏輯與前端的邏輯一致。

當具有狀態的資料必須要存放在客戶端時，針對資料進行加密，並確保資料的一致性而避免被竄改是很重要的。

應用程式的邏輯必須和企業的規定符合。

處理交易的需求時，單一使用者或設備在單位時間內可以進行的交易數量必須要高於企業的需求，但也不能高到讓使用者可以進行 DoS 攻擊。

要注意的是，僅僅使用 `referer` HTTP header 是不足以驗證授權與否的，你應該只使用它作為附加的檢查項目。

對於長時間被授權的 session 來說，應用程式應該要定期檢查該使用者的權限是否被改變，如果被授權的功能已經被改變，應該要取消授權，讓使用者重新被驗證。

使用者的帳戶也應該要能夠被稽核，為了確保安全性(例如：當使用者的密碼過期超過 30 天後，帳戶應該被取消)。

系統也應該要支援當使用者的授權被取消時，要能夠中止使用者的帳號(例如：角色改變、就業狀態改變等)。

當你的系統支援外部帳號登入，或登入到外部服務時，應該要維持最低的權限。

[1]: /error-handling-logging/error-handling.md
[2]: URL.go
[3]: /session-management/README.md
