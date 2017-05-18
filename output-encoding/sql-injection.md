SQL Injection
=============

另外一個因為沒有設定好編碼導致 injection 的問題就是 sql injection，這通常來自於一個不好的習慣：字串串連。

簡言之：任何帶值的變數可能包含任意的字元，當這個變數值待有特殊意義的字元被注入到資料庫系統中時，就有可能發生 SQL injection。

想像一下你要執行以下的 SQL 語句：

```go
customerId := r.URL.Query().Get("id")
query := "SELECT number, expireDate, cvv FROM creditcards WHERE customerId = " + customerId

row, _ := db.Query(query)
```
執行的話，你會毀了你的人生。

當你提供的是正確的 `customerId` 時，你會得到正確的顧客信用卡資料，但假設 `customerId` 變成了 `1 OR 1` 這樣的字串呢？

你的 SQL 查詢語句變成：

```SQL
SELECT number, expireDate, cvv FROM creditcards WHERE customerId = 1 OR 1=1
```

如此一來，就會把所有資料表的記錄給挑選出來了 (是的，`1=1` 代表任何紀錄都是 True)。

只有一種方式來保護你資料庫的安全，那就是使用 [Prepared Statements][1]。

```go
customerId := r.URL.Query().Get("id")
query := "SELECT number, expireDate, cvv FROM creditcards WHERE customerId = ?"

stmt, _ := db.Query(query, customerId)
```
注意 [^1] `?` 這個符號，如此一來你的查詢會變得：

 * 可讀
 * 簡短，且
 * 安全

你可以閱讀資料庫安全的章節來得到更詳細的資訊。

---
[^1] 這個語法是給資料庫使用的語法

[1]: https://golang.org/pkg/database/sql/#DB.Prepare
