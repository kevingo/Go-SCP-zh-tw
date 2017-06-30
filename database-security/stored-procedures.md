預存程序
=================

開發者可以透過預存程序來建立特定的視圖，以防止特定敏感的資訊被封存，而不是使用正常的查詢。

透過建立和限制預存程序的使用，開發者可以針對資料庫的操作建立一個介面，透過這個介面來限制與區分誰可以存取特定的資料。使用這種方式，開發者在安全控制上，是比較方便的。

讓我們看一個範例：

想像一下你有一個資料表是儲存關於使用者密碼和 ID，你使用以下的 query：

```SQL
SELECT * FROM tblUsers WHERE userId = $user_input
```

除了[輸入驗證][1] 的問題外，資料庫的使用者(假設叫做 John)可以存取所有使用者的資料。

如果 John 只擁有操作預存程序的權限呢？

```SQL
CREATE PROCEDURE db.getName @userId int = NULL
AS
    SELECT name, lastname FROM tblUsers WHERE userId = @userId
GO
```

你可以只執行以下指定：

```
EXEC db.getName @userId = 14
```

這樣的話你可以確保 John 只會看到 `name` 和 `lastname`。

預存程序並不是一層防彈玻璃，但他可以為你的網站應用建立一層保護殼。相比權限控制(例如：讓使用者僅能存取特定的資料或資料表)，它讓 DBA 可以有更好的方式來管理安全性，甚至也可以有更好的效能。

[1]: /input-validation/README.md
