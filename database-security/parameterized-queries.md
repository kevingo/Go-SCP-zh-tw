參數化查詢
=====================

Prepared Statements (加上參數化查詢) 是防止 SQL injection 攻擊最好也最安全的方法。

在某些報告中，你可能會閱讀到 prepared statements 可能會降低應用程式的性能，因此如果你想要停用 prepared statements 時，我們強烈建議你閱讀 [輸入驗證][1] 和 [輸出編碼][2] 等章節。

Go 和其他語言在使用 prepared statmenets 的方法不同：你不是使用在資料庫連線上，而是使用在資料庫端。

## 流程

1. 開發者在 statement(`stmt`) 從連接池中準備連線。
2. `Stmt` 會記住現在使用哪個連線。
3. 當應用程式使用 `stmt` 物件時，它會嘗試使用該連線。如果該連線無法使用，則嘗試從連接池中尋找另外一個連線。

這可能會造成資料庫有過多的連線，並且創造太多的 prepared statements，所以要記得這個重要的流程。

底下是一個 prepared statement 加上參數化查詢的範例：

```go
customerName := r.URL.Query().Get("name")
db.Exec("UPDATE creditcards SET name=? WHERE customerId=?", customerName, 233, 90)
```

某些時候，prepared statements 可能不是你想要的，原因有以下兩點：

* 資料庫不支援 prepared statement。比如說，當你使用 MySQL 的驅動程式時，你可以連線到 MemSQL 和 Sphinx，因為他們兩個資料庫也支援 MySQL 的通訊協定。但他們並不支援 "binary" 的通訊協定，包含了 prepared statement。

* 這些 statements 不會重用，導致使用他們並不划算。同時，你可以會把安全考量在其他的面向處理掉(你可以閱讀：[輸入驗證][1] 和 [輸出編碼][2] 等章節)。

[1]: /input-validation/README.md
[2]: /output-encodeing/README.md
