加密實踐
======================

讓我們做出第一個聲明，你的加密方式應該是：**hashing 和加密應該是兩個不一樣的事情**。

有一個普遍的誤解是：加密和 hashing 在大多時間可以交換使用。這其實是不正確的。他們兩個是不同的概念，也有不同的用途。

Hash 指的是一串字串或數字透過 hash 函式產生出來的結果：

```
hash := F(data)
```

hash 出來的值是固定長度，同時當輸入有微小變化的時候，輸出值就會不同(也是有可能發生碰撞)。一個好的 hash 演算法是不可逆的(無法逆向演算回原本的數值)，MD5 是相當熱門的 hash 演算法，而 SHA-256 被認為是最可靠的 hash 演算法。當你有任何資料是你不需要知道他是什麼，但必須要儲存它時(例如使用者密碼)，就可以考慮使用 hash[^2]。

```go
package main

import "fmt"
import "io"
import "crypto/md5"
import "crypto/sha256"

func main () {
        h_md5 := md5.New()
        h_sha := sha256.New()
        io.WriteString(h_md5, "Welcome to Go Language Secure Coding Practices")
        io.WriteString(h_sha, "Welcome to Go Language Secure Coding Practices")
        fmt.Printf("MD5   : %x\n", h_md5.Sum(nil))
        fmt.Printf("SHA256: %x\n", h_sha.Sum(nil))
}
```

輸出：

```
MD5   : ea9321d8fb0ec6623319e49a634aad92
SHA256: ba4939528707d791242d1af175e580c584dc0681af8be2a4604a526e864449f6
```

另一方面，加密會使用密鑰將資料轉換為可變長度的資料。

```
encrypted_data := F(data, key)
```

跟 hash 不同的是，我們可以使用正確的解密函式和密鑰將 `加密資料` 計算出 `原始的資料`。

```
data := F⁻¹(encrypted_data, key)
```

只要需要傳輸資料或儲存敏感性的資料，都應該使用加密方式來處理。一個常見的案例是 HTTPS - Hyper Text Transfer Protocol Secure。AES 是關於對稱式加密的主流標準。另一方面，你也可以使用公鑰和私鑰來進行非對稱加密法或公開金鑰加密。

```go
package main

import "fmt"
import "io"
import "crypto/aes"
import "crypto/cipher"
import "crypto/rand"

func main() {
        key := []byte("Encryption Key should be 32 char")
        data := []byte("Welcome to Go Language Secure Coding Practices")

        block, err := aes.NewCipher(key)
        if err != nil {
                panic(err.Error())
        }

        nonce := make([]byte, 12)
        if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
                panic(err.Error())
        }

        aesgcm, err := cipher.NewGCM(block)
        if err != nil {
                panic(err.Error())
        }

        encrypted_data := aesgcm.Seal(nil, nonce, data, nil)
        fmt.Printf("Encrypted: %x\n", encrypted_data)

        decrypted_data, err := aesgcm.Open(nil, nonce, encrypted_data, nil)
        if err != nil {
                panic(err.Error())
        }

        fmt.Printf("Decrypted: %s\n", decrypted_data)
}
```

```
Encrypted: a66bd44db1fac7281c33f6ca40494a320644584d0595e5a0e9a202f8aeb22dae659dc06932d4e409fe35a95d14b1cffacbe3914460dd27cbd274b0c3a561
Decrypted: Welcome to Go Language Secure Coding Practices
```

請注意，你應該 "_建立如何管理金鑰的流程_"，保護 "_未經授權的存取_"。另一方面，你的加密密鑰不應該寫在原始的程式碼當中。

[Go's crypto package][1] 搜集了常見的加密方式，而實作的部分分散在各個套件中，像是 [crypto/md5][2] 就是一個例子。

---

[^1]: 彩虹表攻擊並不是 hashing 演算法的弱點。
[^2]: 可以閱讀 [驗證與密碼管理章節][3] 中關於 "_單向的 salted hash 函式_"。

[1]: https://golang.org/pkg/crypto/
[2]: https://golang.org/pkg/crypto/md5/
[3]: /authentication-password-management.html
