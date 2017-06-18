資料庫安全
=================

在本章中，OWASP SCP 會涵蓋所有關於資料庫安全的議題，同時了解開發者或資料庫管理者在使用資料庫時應該要採取的行動。

想要使用資料庫，你可以參考 [database/sql][1] 套件。

## 最佳實踐

Before implementing your database in Go, you should take care of some configurations that we'll cover next:

在你開始使用 Go 語言撰寫資料庫相關應用前，你應該注意以下幾點：

* 安全的安裝資料庫系統[^1]
    * 變更/設定 `root` 帳號的密碼。
    * 不允許讓 `root` 可以從本機以外的機器存取資料庫。
    * 移除任何匿名的帳戶。
    * 移除任何測試用的資料庫。
* 移除任何不需要的 stored procedures、工具套件、無用的服務、供應商的內容(例如：範例的 schema)。
* 安裝資料庫使用 Go 語言所需要的最小功能。
* 停用任何網站不需要連接到資料庫所使用的帳號。

另外，驗證資料庫的輸入、輸出是相當重要的，確保你已經閱讀[輸入編碼][4]和[輸出編碼][5]等章節。

這些基本上可以適用於任何需要和資料庫連線的程式語言最佳實踐。

---

[^1]: MySQL/MariaDB have a program for this: `mysql_secure_installation`<sup>[1][6], [2][7]</sup>

[1]: https://golang.org/pkg/database/sql/
[2]: https://github.com/go-sql-driver/mysql
[3]: https://github.com/mattn/go-sqlite3
[4]: /input-validation/README.md
[5]: /output-encoding/README.md
[6]: https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html
[7]: https://mariadb.com/kb/en/mariadb/mysql_secure_installation/
