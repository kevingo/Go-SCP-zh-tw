驗證資料通訊方式
=================================

在本章節中，"通訊" 廣泛地用指使用者體驗(UX) 和客戶端-伺服器端的通訊。 

不僅僅是_密碼輸入的欄位應該在使用者螢幕上被遮蔽_，同時_記住我的功能也應該被停用_。

你可以透過把輸入的欄位設定為 `type="password"`，以及將 `autocomplete` 的屬性設為 `off`[^1] 來達成這樣的目標。

```html
<input type="password" name="passwd" autocomplete="off" />
```

認證用的帳號密碼或其他憑證應該只透過 HTTP POST 方法與 HTTPS 協定來傳送。唯一的例外可能是與電子郵件重置相關的臨時密碼。

儘管你使用 HTTP GET 在 TLS/SSL (HTTPS) 的通訊協定上看起來跟 HTTP POST 一樣安全，但你要記住一般的 HTTP 伺服器(例如：Apache[^2], Nginx[^3]) 會將請求的 URL 路徑寫到日誌當中，如下所示：

```text
xxx.xxx.xxx.xxx - - [27/Feb/2017:01:55:09 +0000] "GET /?username=user&password=70pS3cure/oassw0rd HTTP/1.1" 200 235 "-" "Mozilla/5.0 (X11; Fedora; Linux x86_64; rv:51.0) Gecko/20100101 Firefox/51.0"
```

一個設計良好用來進行驗證的 HTML 表單如下：

```html
<form method="post" action="https://somedomain.com/user/signin" autocomplete="off">
    <input type="hidden" name="csrf" value="CSRF-TOKEN" />

    <label>Username <input type="text" name="username" /></label>
    <label>Password <input type="password" name="password" /></label>

    <input type="submit" value="Submit" />
</form>
```

在處理驗證的錯誤訊息時，應用程式不應該明確的指出哪一部份的資料是不正確的，比如說，你不應該指出使用者輸入的使用者名稱錯誤，或是密碼錯誤，而應該只跟使用者說 "使用者名稱或密碼錯誤"。

```html
<form method="post" action="https://somedomain.com/user/signin" autocomplete="off">
    <input type="hidden" name="csrf" value="CSRF-TOKEN" />

    <div class="error">
        <p>Invalid username and/or password</p>
    </div>

    <label>Username <input type="text" name="username" /></label>
    <label>Password <input type="password" name="password" /></label>

    <input type="submit" value="Submit" />
</form>
```

透過這樣較為一般性的錯誤訊息，你可以避免揭露類似以下的敏感資訊給使用者：

* 誰已經註冊了： "錯誤密碼" 代表該使用者已經存在。
* 你的系統運作方式： "錯誤的密碼" 揭露了你的系統運作的模式。

```go
  record, err = db.Query("SELECT password FROM accounts WHERE username = ?", username)
  if err != nil {
    // user does not exist
  }

  if subtle.ConstantTimeCompare([]byte(record[0]), []byte(password)) != 1 {
    // passwords do not match
  }
```

After a successful login, the user should be informed about the last successful or unsuccessful access date/time so that he can detect and report suspicious activity. Further information regarding logging can be found in the [`Error Handling and Logging`][4] section of the document. Moreover, it is also recommended to use a constant time comparison function while checking passwords in order to prevent timing attack. The latter consists of analyzing the difference of time between multiple requests with different inputs. In this case, a standard comparison of the form `record == password` would return false at the first character that does not match. The closer the submitted password, the longer the response time. By exploiting that, an attacker could guess the password.

---

[^1]: [How to Turn Off Form Autocompletion][1], Mozilla Developer Network
[^2]: [Log Files][2], Apache Documentation
[^3]: [log_format][3], Nginx log_module "log_format" directive

[1]: https://developer.mozilla.org/en-US/docs/Web/Security/Securing_your_site/Turning_off_form_autocompletion
[2]: https://httpd.apache.org/docs/1.3/logs.html#accesslog
[3]: http://nginx.org/en/docs/http/ngx_http_log_module.html#log_format
[4]: ../error-handling-logging/logging.md
