驗證與儲存使用者驗證資料
==========================================

驗證
----------

本節中關鍵的主題是如何儲存驗證資料，因為在網路上，使用者帳戶的資料庫是很容易被洩露的。當然這不能保證一定會發生，但如果使用者的密碼有被妥善保存，則傷害可以減到最低。

首先，讓我們清楚的說明："_所有的驗證控制應該要安全的失敗_"。建議你可以閱讀其他身份驗證和密碼管理的章節，以了解有關錯誤身份驗證的資料以及如何處理日誌記錄的建議。

另外一個建議是：對於循序認證的方式(就像 Google 現在這樣)，驗證的過程只有在受信任的系統輸入完所有資料後才進行。


安全的儲存密碼: 理論部分
-------------------------------------

現在讓我們談談如何安全的儲存密碼。

事實上你不需要真正的儲存密碼，因為他們會由使用者以明文的方式來提供。但你需要驗證每一次使用者提交的密碼是不是相同。

因此，為了安全的考量，你需要的是一個 "單向" 的函式 `H`，當使用者提交 `p1` 和 `p2` 兩個不同的密碼時，`H(p1)` 和 `H(p2)` 必須要不同。

這聽起來像是數學嗎？注意最後一個需求：`H` 是一個函式，這個函式必須要讓 `H⁻¹(H(p1))` 不能夠輸出 `p1`，這表示 `H` 是一個單向的函式，沒辦法知道原始的密碼。

如果 `H` 是一個單向的函式，那關於帳號洩漏的真正問題是什麼？

如果你知道所有可能的密碼組合，你就可以預先計算出他們的 hash 值，並且進行彩虹表攻擊手法。

現在你知道從使用者的角度而言，密碼是很難被管理的，而他們不僅會使用重複的密碼，也會使用那些容易被記憶的簡單密碼。

如何避免這件事情？

重點是：如果兩個使用者提供了相同的密碼 `p1`，我們應該要儲存不同的 hash 值。這聽起來似乎不可能，但其實只要透過 `salt` 的方式：增加一個隨機的值到 `p1` 即可，所以我們產生的 hash 值會是 `H(p1 + salt)`。

So each entry on passwords store should keep the resulting hash and the `salt` itself in plaintext: `salt` does not offer any security concerns.

最後的建議：
* 避免使用 MD5 hash 演算法
* 避免使用 SHA1 演算法，因為他最近被破解了
* 閱讀 [Pseudo-Random Generators section][1] 章節

下面提供一個範例：

```go
package main

import (
    "crypto/rand"
    "crypto/sha256"
    "database/sql"
    "fmt"
    "io"
)

const saltSize = 32

func main() {
    email := []byte("john.doe@somedomain.com")
    password := []byte("47;u5:B(95m72;Xq")

    // create random word
    salt := make([]byte, saltSize)
    _, err := io.ReadFull(rand.Reader, salt)
    if err != nil {
        panic(err)
    }

    // let's create SHA256(password+salt)
    hash := sha256.New()
    hash.Write(password)
    hash.Write(salt)

    // this is here just for demo purposes
    //
    // fmt.Printf("email   : %s\n", string(email))
    // fmt.Printf("password: %s\n", string(password))
    // fmt.Printf("salt    : %x\n", salt)
    // fmt.Printf("hash    : %x\n", hash.Sum(nil))

    // you're supposed to have a database connection
    stmt, err := db.Prepare("INSERT INTO accounts SET hash=?,salt=?,email=?")
    if err != nil {
        panic(err)
    }
    result, err := stmt.Exec(email, h, salt)
    if err != nil {
        panic(err)
    }

}
```

然而，這種方法有幾個缺點，你也不應該使用它。在這裡只是會了給你一個實際的範例，下一章節中我們會探討如何正確的使用 salt。


安全的儲存密碼：實際部分
---------------------------------------

在密碼學中最重要的一句格言是：**永遠不要自己寫加密程式**。不這樣做的話，你可能會把整個應用程式陷於危險之中。這是一個敏感且複雜的主題，可喜的是，加密技術已經有現成的工具可以使用，同時經過專家的檢驗。因此正確的使用它們而不是自己造輪子才是對的做法。

在儲存密碼的前提下，[OWASP][2] 所推薦的 hash 演算法有 [`bcrypt`][2]、[`PDKDF2`][3]、[`Argon2`][4] 和 [`scrypt`][5]。這些套件以強大的方式來處理密碼和 salt。Go 在這方面提供了一個函式庫，它不是標準函式庫的一部份，你可以透過 `go get` 來下載它：

```
go get golang.org/x/crypto
```

下面的範例中會展示如何使用 bcrypt，在大多數的情況下應該都是足夠使用的。bcrypt 的優點是他使用起來很簡單，也因此較不容易出錯：

```go
package main

import (
    "database/sql"
    "fmt"

    "golang.org/x/crypto/bcrypt"
)

func main() {
    email := []byte("john.doe@somedomain.com")
    password := []byte("47;u5:B(95m72;Xq")

    // Hash the password with bcrypt
    hashedPassword, err := bcrypt.GenerateFromPassword(password, bcrypt.DefaultCost)
    if err != nil {
        panic(err)
    }

    // this is here just for demo purposes
    //
    // fmt.Printf("email          : %s\n", string(email))
    // fmt.Printf("password       : %s\n", string(password))
    // fmt.Printf("hashed password: %x\n", hashedPassword)

    // you're supposed to have a database connection
    stmt, err := db.Prepare("INSERT INTO accounts SET hash=?, email=?")
    if err != nil {
        panic(err)
    }
    result, err := stmt.Exec(hashedPassword, email)
    if err != nil {
        panic(err)
    }
}
```

bcrypt 也提供了簡單和安全的方式來比較原始密碼與 hash 過後的密碼：

 ```go
 // credentials to validate
 email := []byte("john.doe@somedomain.com")
 password := []byte("47;u5:B(95m72;Xq")

// fetch the hashed password corresponding to the provided email
record := db.QueryRow("SELECT hash FROM accounts WHERE email = ? LIMIT 1", email)

var expectedPassword string
if err := record.Scan(&expectedPassword); err != nil {
    // user does not exist
}

if bcrypt.CompareHashAndPassword(password, []byte(expectedPassword)) != nil {
    // passwords do not match
}
 ```

[^1]: Hash 函式會有碰撞的問題，但推薦的 hash 函式發生碰撞的機率都非常低。

[1]: /cryptographic-practices/pseudo-random-generators.md
[2]: https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet
[3]: https://godoc.org/golang.org/x/crypto/bcrypt
[4]: https://github.com/p-h-c/phc-winner-argon2
[5]: https://godoc.org/golang.org/x/crypto/pbkdf2
