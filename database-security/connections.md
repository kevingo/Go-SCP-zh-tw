資料庫連線
====================

## 保持連線關閉！

大多數開發者忘記做的一件事情就是關閉資料庫連線，讓我們來看一個範例：

```go
package main

import "fmt"
import "database/sql"
import "github.com/go-sql-driver/mysql"

func main() {
    db, _ := sql.Open("mysql", "user:@/cxdb")
    defer db.Close()

    var version string db.QueryRow("SELECT VERSION()").Scan(&version)
    fmt.Println("Connected to:", version)
}
```

當你開啟了資料庫連線後，Go 會使用 [db.Close()][1] 來關閉連線。在這個範例中，我們使用了 MariaDB 的資料庫連線驅動程式。特別注意在 `defer` 敘述中的 `db.Close()`。在 Go 語言中，這樣會確保此連線最後會被關閉。更多資訊可以參考[錯誤管理與日誌][2] 章節。

你可能會問 - *為什麼我需要關掉它？*

在關閉 SQL 連線後，它會將連線的資源歸還到連線池中。更進一步地說，因為連線的數目是有限的，同時需要消耗運算資源，如果你使用相同的連線字串，就有可能重複從連線池中重用。

## 保護連線字串

為了保護你的資料庫連線字串，將這些驗證使用的詳細內容設定檔案放在公開存取以外的地方是一個很好的做法。

不要將這些設定檔案放在像是 `/home/public_html/` 公開的目錄，可以考慮放置於  `/home/private/configDB.xml` (應該被放在受保護的目錄下)。

```xml
<connectionDB>
  <serverDB>localhost</serverDB>
  <userDB>f00</userDB>
  <passDB>f00?bar#ItsP0ssible</passDB>
</connectionDB>
```

你可以在程式碼中呼叫設定檔：

```go
configFile, _ := os.Open("../private/configDB.xml")
```

讀取設定檔後，建立資料庫連線：

```go
db, _ := sql.Open(serverDB, userDB, passDB)
```

當然，當攻擊者擁有 root 權限時，他當然可以看到這些檔案，更警慎一點的話，我們可以加密這些檔案。

## 資料庫憑證

你應該針對不同等級的使用者給予不同的資料庫憑證：

* 使用者
* 唯讀使用者
* 訪客
* 管理者

當唯讀的使用者連線到你的資料庫時，不用擔心他會破壞你的資料庫內容，因為他只能讀取資料而已。

[1]: https://golang.org/pkg/database/sql/#DB.Close
[2]: ../error-handling-logging/README.md
