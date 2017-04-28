驗證
==========
在驗證檢查中，使用者的輸入會被一連串的條件檢查，來確保它輸入的資料是如我們所預期

**重要:** 當驗證錯誤時，請求應該被拒絕。

這個重要性不僅僅在於資訊安全考量上，同時也確保了資料的一致性，畢竟這些資料經常會被用在多個系統中。在本文章中會列舉出開發者在開發網站時所需要注意的資訊安全考量要點。

## 使用者互動
在網站中任何可以允許使用者進行互動的部分都可能會造成安全風險。問題不僅僅來自於差的代理因為妥協而造成的風險，也是來自於人為疏忽所導致的輸入錯誤(統計上來看，大多數無效資料通常是因為人為的錯誤所引起)。在 Go 中，有一些方法可以避免這些狀況。

Go 有一些內建的函式庫可以讓你確保以上的錯誤不會發生。當你要針對字串進行處理時，我們可以使用底下的一些範例：

* `strconv` 將字串轉換為其他型別的函式庫
    * [`Atoi`](https://golang.org/pkg/strconv/#Atoi)
    * [`ParseBool`](https://golang.org/pkg/strconv/#ParseBool)
    * [`ParseFloat`](https://golang.org/pkg/strconv/#ParseFloat)
    * [`ParseInt`](https://golang.org/pkg/strconv/#ParseInt)
* `strings` 處理字串的套件
    * [`Trim`](https://golang.org/pkg/strings/#Trim)
    * [`ToLower`](https://golang.org/pkg/strings/#ToLower)
    * [`ToTitle`](https://golang.org/pkg/strings/#ToTitle)
* [`regexp`][4] 支援正規表示式的套件[^1].
* [`utf8`][9] 支援 UTF-8 處理的套件，當中的函式可以轉換 runes 和 UTF-8。

  驗證用 UTF-8 編碼的 rune：
    * [`Valid`](https://golang.org/pkg/unicode/utf8/#Valid)
    * [`ValidRune`](https://golang.org/pkg/unicode/utf8/#ValidRune)
    * [`ValidString`](https://golang.org/pkg/unicode/utf8/#ValidString)

  針對 UTF-8 runes 進行編碼：
    * [`EncodeRune`](https://golang.org/pkg/unicode/utf8/#EncodeRune)

  解碼 UTF-8：
    * [`DecodeLastRune`](https://golang.org/pkg/unicode/utf8/#DecodeLastRune)
    * [`DecodeLastRuneInString`](https://golang.org/pkg/unicode/utf8/#DecodeLastRuneInString)
    * [`DecodeRune`](https://golang.org/pkg/unicode/utf8/#DecodeLastRune)
    * [`DecodeRuneInString`](https://golang.org/pkg/unicode/utf8/#DecodeRuneInString)


**注意**: `Forms` 在 Go 中是以 `Maps` 型別的 `String` 值存在。

其他用來驗證資料的方法有：

* 白名單驗證 - 透過白名單來檢查輸入的值是否在名單中。查看 [Validation - Strip tags][1].
* 長度檢查 - 驗證資料和數字的輸入長度。
* 跳脫字元 - 記得要跳脫特殊的字元，例如單引號。
* 數值驗證 - 確認輸入是數值型態。
* 檢查 NULL 型別 - `(%00)`
* 檢查換行字元 - `%0d`, `%0a`, `\r`, `\n`
* 檢查任何改變路徑的字元 - `../` or `\\..`
* 檢查擴展的 UTF-8 - 檢查某些特殊的字元

**注意**: 確認 HTTP 的 Header 請求和回應只包含 ASCII 字元。

有一些第三方套件也可以幫助你驗證：

* [Gorilla][6] - 用於網站安全性最常被使用的套件之一。它支援 `websockets`、`cookie sessions`、`RPC`等等。

* [Form][7] - 將 `url.Values` 解析成 Go 的值，同時也可以將 Go 的值編碼為 `url.Values`。可以支援陣列和 `map`。

* [Validator][8] - Go 的 `Struct` 和 `Field` 驗證。包含 `Cross Field`、`Cross Struct`、`Map`、`Slice` 和 `Array`。

## 轉址

Go 會在內部直接處理轉址，除非你在伺服器端有特別處理，不然 Go 會直接轉址到目標的頁面。這可能會讓一些惡意的攻擊者有機會跳過資料驗證的流程，直接發送惡意的資料到目標的網站。這種情況下，開發者必須要特別處理需要轉址的情形，確保使用者提供的資料是符合驗證的流程。

## 檔案操作

任何時候需要進行文件的讀寫操作時，也需要通過驗證檢查，因為大多數的文件操作都會和使用者的資料有關。另外一個檔案檢查的流程也會包含「檔案存在檢查」，用來驗證檔案的檔名是否存在。其他關於檔案的資料會在 [檔案管理][2] 的章節，而「錯誤處理」的章節中也會包含如何處理讀寫檔案的錯誤。

## 資料來源

任何時候當檔案從一個可信任的來源傳到一個較不可信任的來源時，都必須要進行驗證檢查。這會確保資料沒有被竄改，我們正在接收的是如我們所預期的資料。其他資料來源檢查的方法包含：

* 跨系統一致性檢查
* 透過 Hash 值檢查
* 參照完整性檢查

**注意:** 在現代資料庫系統中，如果主鍵中的值不受到資料庫內部機制的約束時，就應該被驗證。

* 唯一性檢查
* 資料表查詢

## Post-validation Actions

According to Data Validation's best practices, the input validation is only
the first part of the data validation guidelines. As such,
_Post-validation Actions_ should also be performed.
The _Post-validation Actions_ used vary with the context and are divided in
three separate categories:

根據資料驗證的最佳實踐，輸入資料的驗證只是資料驗證守則中的第一項。因此我們還需要進行 `Post-validation Actions`。

## Enforcement Actions

有數個 Enforcement Actions 可以用來確保我們的應用程式和資料的安全性。

其中一個 Enforcement Actions 在於通知使用者所送出的資料並未符合預期要求，並且應該將資料調整為符合要求的內容。

另外一種強制執行的方法是在伺服器端強制修改使用者提交上來的資料，而不通知使用者他送出的資料已經被修改。這是大多數用於互動式系統上的情境。

**注意:** 後者的情況大多用在較不敏感的資料(修改敏感性較高的資料容易導致資料缺失或其他問題)。

* **建議操作** - 建議操作通常允許沒有變更的資料被輸入，但是來源通常知道輸入的資料存在問題。這適用於非交互的系統中。
* **驗證操作** - 驗證操作是建議操作的一種特殊案例。在這個例子中，使用者輸入的資料會被建議要如何修改，使用者可以接受這些建議的變更，或是保留原本他自己的輸入。

A simple way to illustrate this is a Billing address form, where the user
enters his address and the system suggests addresses associated with the
account. The user then accepts one of these suggestions or ships to the address
that was initially entered.

---

[^1]: Before writing your own regular expression have a look at [OWASP Validation Regex Repository][5]

[1]: sanitization.md
[2]: ../file-management/README.md
[3]: ../error-handling-logging/README.md
[4]: https://golang.org/pkg/regexp/
[5]: https://www.owasp.org/index.php/OWASP_Validation_Regex_Repository
[6]: https://github.com/gorilla/
[7]: https://github.com/go-playground/form
[8]: https://github.com/go-playground/validator
[9]: https://golang.org/pkg/unicode/utf8/
