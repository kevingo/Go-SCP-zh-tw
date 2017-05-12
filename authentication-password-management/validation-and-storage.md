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

The point is: if two different users provide the same password `p1` we should store a different hashed value. It may sound impossible but the answer is `salt`: a pseudo-random value which is append to `p1` so that the resulting hash is computed as follow: `H(p1 + salt)`.

重點是：如果兩個使用者提供了相同的密碼 `p1`，我們應該要儲存不同的 hash 值。

So each entry on passwords store should keep the resulting hash and the `salt` itself in plaintext: `salt` does not offer any security concerns.

Last recommendations.
* Avoid using MD5 hashing algorithm whenever possible;
* Avoid using SHA1 as it has been cracked recently.
* Read the [Pseudo-Random Generators section][1].

The following example shows a basic example of how this works:

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

However, this approach has several flaws and should not be used. It is given here only to illustrate the theory with a practical example. The next section explains how to correctly salt passwords in real life.


Storing password securely: the practice
---------------------------------------

One of the most important adage in cryptography is: **never write your own crypto**. By not doing so, one can put at risk the entire application. It is a sensitive and complex topic. Hopefully, cryptography provides tools and standards reviewed and approved by experts. It is therefore important to use them instead of trying to re-invent the wheel.

In the case of password storage, the hashing algorithms recommended by [OWASP][2] are [`bcrypt`][2], [`PDKDF2`][3], [`Argon2`][4] and [`scrypt`][5]. Those take care of hashing and salting passwords in a robust way. Go authors provides an extended package for cryptography, that is not part of the standard library. It provides robust implementations for most of the aforementioned algorithms. It can be downloaded using  `go get`:

```
go get golang.org/x/crypto
```

The following example shows how to use bcrypt, which should be good enough for most of the situations. The advantage of bcrypt is that it is simpler to use and is therefore less error-prone.

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

Bcrypt also provides a simple and secure way to compare a plaintext password with an already hashed password:

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

[^1]: Hashing functions are the subject of Collisions but recommended hashing functions have a really low collisions probability

[1]: /cryptographic-practices/pseudo-random-generators.md
[2]: https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet
[3]: https://godoc.org/golang.org/x/crypto/bcrypt
[4]: https://github.com/p-h-c/phc-winner-argon2
[5]: https://godoc.org/golang.org/x/crypto/pbkdf2
