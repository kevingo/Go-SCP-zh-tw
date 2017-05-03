日誌
=======

日誌應該要由應用程式來處理，而不應該倚賴伺服器上的設定。

所有的日誌應該由主要的 routine 來負責處理，同時開發者必須要確保沒有任何機密性的資料被記錄到日誌中(例如：密碼、session 資訊、系統詳細資訊等)，也不應該有任何程式的 debug 紀錄或 stack trace 資訊。此外，日誌應該要包含成功和非成功的安全活動，以及重要的活動資料。

重要的活動資料通常包含：

* 所有導致輸入檢查錯誤的輸入值。
* 所有認證的嘗試，特別是認證錯誤的部分。
* 所有關於存取控制錯誤的部分。
* 所有明顯的竄改事件，包括對於預期之外的狀態更改。
* 所有嘗試透過非法或過期 token 的嘗試連線。
* 所有系統的例外。
* 所有管理者的功能，包還嘗試更改安全設定的紀錄。
* 所有 TLS 錯誤連線，以及加密模組的錯誤。

一個簡單的範例如下：

```go
func main() {
    var buf bytes.Buffer
    var RoleLevel int

    logger := log.New(&buf, "logger: ", log.Lshortfile)

    fmt.Println("Please enter your user level.")
    fmt.Scanf("%d", &RoleLevel) //<--- example

    switch RoleLevel {
    case 1:
        // 成功登入的日誌紀錄
        logger.Printf("Login successfull.")
        fmt.Print(&buf)
    case 2:
        // 登入失敗的日誌紀錄
        logger.Printf("Login unsuccessfull - Insufficient access level.")
        fmt.Print(&buf)
     default:
        // 非正常狀況的日誌紀錄
        logger.Print("Login error.")
        fmt.Print(&buf)
    }
}
```

同樣的，實作通用性的錯誤訊息或是客製化的錯誤頁面都是確保不會有額外錯誤資訊被洩漏的好方法。

Go's native package to handle logs doesn't support log levels, which
means that natively to have level based logging would mean implementing
levels by hand. Another issue with the native logger is that there is no
way to turn logging on or off on a per-package basis.

Go 原生的日誌套件並沒有支援不同等級的日誌記錄功能，代表如果你直接使用原生的日誌，這部分你必須要自己處理。

由於所有的應用程式都需要有適當的日誌機制來維持良好的安全性，有許多的第三方套件可以提供你選擇：

* [Logrus][1] - https://github.com/Sirupsen/logrus
* [glog][2]   - https://github.com/golang/glog
* [loggo][3]  - https://github.com/juju/loggo

在這些套件中，最常被使用的是 [Logrus](https://github.com/Sirupsen/logrus)

從日誌管理的角度來說，只有被授權的人可以有權限存取日誌。開發者應該確保日誌分析的工具可供使用，同時要注意沒有任何不可信任的軟體或可以存取日誌。

關於記憶體清理的問題，Go 有內建的垃圾回收機制來處理這類的問題。而最後，為了確保日誌的有效性和完整性，應該使用一個 hash 函式來確保該日誌沒有被任意的竄改。

```go
{...}
// Get our known Log checksum from checksum file.
logChecksum, err := ioutil.ReadFile("log/checksum")
str := string(logChecksum) // convert content to a 'string'

// Compute our current log's MD5
b, err := ComputeMd5("log/log")
if err != nil {
  fmt.Printf("Err: %v", err)
} else {
  md5Result := hex.EncodeToString(b)
  // Compare our calculated hash with our stored hash
  if str == md5Result {
    // Ok the checksums match.
    fmt.Println("Log integrity OK.")
  } else {
    // The file integrity has been compromised...
    fmt.Println("File Tampering detected.")
  }
}
{...}
```

注意: `ComputeMD5()` 函式會去計算檔案的 MD5，同時，日誌檔案的 hash 結果必須要存放於安全的地方，並且在更新日誌前，要比對 hash 值，確認日誌沒有被任意竄改後再更新內容。

[1]: https://github.com/Sirupsen/logrus
[2]: https://github.com/golang/glog
[3]: https://github.com/juju/loggo
