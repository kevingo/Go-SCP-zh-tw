XSS - 跨站腳本攻擊
==========================

大多數的開發者應該都聽過 XSS 攻擊，但很少人會實際嘗試他。

跨站腳本攻擊從 2003 年開始是名列 [OWASP Top 10][0] 的資安問題，直到現在還是相當常見的風險之一。在 [2013 version][1] 中詳細的描述了這個資安問題，包括了：攻擊的對象、安全弱點、技術影響和企業業務影響等。

簡單來說：

> 如果你沒辦法確保使用者的輸入有正確的被處理，或是在伺服器端驗證之前就把使用者的輸入顯示在輸出的頁面，你將會處於風險中。([source][1])

Go 語言，就像其他許多語言一樣，儘管它提供了清楚的 [html/template 套件][2] 說明文件，你還是很容易因為不注意而暴露在 XSS 攻擊的風險中。

想像一下下面的程式碼：

```go
package main

import "net/http"
import "io"

func handler (w http.ResponseWriter, r *http.Request) {
    io.WriteString(w, r.URL.Query().Get("param1"))
}

func main () {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

這一段程式碼建立並啟動一個 HTTP 伺服器，監聽 `8080` port，並且接收 `/` 的請求。

`handler()` 函式是用來處理使用者的請求，在這個範例的程式碼中，會預期使用者帶一個 `param1` 的參數，這個參數會寫到回應的 stream (`w`)。

由於在這裡沒有明確指定 HTTP 回應的 `Content-Type`，Go 會使用 `http.DetectContentType` 當成預設值，這個預設值會遵守 [WhatWG spec][5] 標準。

所以，將 `param1` 設為 "test" 會讓 `Content-Type` 自動判定為 `text/plain`。

![Content-Type: text/plain][content-type-text-plain]

但如果將 `param1` 設定為 "&lt;h1&gt;"，`Content-Type` 就會變成 `text/html`.

![Content-Type: text/html][content-type-text-html]

你可能會想，如果將 `param1` 設定為任意的 HTML 標籤都會是一樣的行為。但其實不是，如果你將 `param1` 設定為 "&lt;h2&gt;", "&lt;span&gt;" or "&lt;form&gt;"，`Content-Type` 將會是 `plain/text`，而不是 `text/html`。

現在，我們讓 `param1` 設定為 `<script>alert(1)</script>`。

根據 [WhatWG spec][5] 標準，`Content-Type` 會被標示為 `text/html`，`param1` 的值將會被顯示出來，這樣，的確就是 XSS - 跨站腳本攻擊。

![XSS - Cross-Site Scripting][cross-site-scripting]

在跟 Google 討論這個問題後，他們告知：

> 這實際上非常方便，目的在於讓應用程式能夠顯示 HTML，並且自動設定內容的型態。我們期望開發者能夠使用 html/template 來進行正確的轉換。

Google 表示他們讓開發者自己對自己的程式碼善盡保護的責任，我們完全同意，但讓語言本身允許 `Content-Type` 自動設置除了 `text/plain` 以外的預設值不是最好的做法。

讓我們說得更清楚一點：`text/plain` 或是 [text/template package][6] 不會讓你遠離 XSS 攻擊，因為他們不會處理使用者的輸入。

```go
package main

import "net/http"
import "text/template"

func handler(w http.ResponseWriter, r *http.Request) {
        param1 := r.URL.Query().Get("param1")

        tmpl := template.New("hello")
        tmpl, _ = tmpl.Parse(`{{define "T"}}{{.}}{{end}}`)
        tmpl.ExecuteTemplate(w, "T", param1)
}

func main() {
        http.HandleFunc("/", handler)
        http.ListenAndServe(":8080", nil)
}
```

當 `param1` 為 "&lt;h1&gt;" 時，會讓 `Content-Type` 被自動判斷為 `text/html`，這讓你暴露在 XSS 攻擊的風險中。

![XSS while using text/template package][text-template-xss]

將 [text/template package][6] 改為 [html/template][2]，你會發現比較安全。

```go
package main

import "net/http"
import "html/template"

func handler(w http.ResponseWriter, r *http.Request) {
        param1 := r.URL.Query().Get("param1")

        tmpl := template.New("hello")
        tmpl, _ = tmpl.Parse(`{{define "T"}}{{.}}{{end}}`)
        tmpl.ExecuteTemplate(w, "T", param1)
}

func main() {
        http.HandleFunc("/", handler)
        http.ListenAndServe(":8080", nil)
}
```

不僅僅因為當使用者的 `param1` 輸入是 "&lt;h1&gt;" 時，`Content-Type` 會被判定為 `text/plain`，

![Content-Type: text/plain while using html/template package][html-template-plain-text]

而且 `param1` 會被正確的編碼並輸出到瀏覽器上。

![No XSS while using html/template package][html-template-noxss]

[exploit-of-a-mom]: images/exploit-of-a-mom.png
[content-type-text-plain]: images/text-plain.png
[content-type-text-html]: images/text-html.png
[cross-site-scripting]: images/xss.png
[text-template-xss]: images/text-template-xss.png
[html-template-plain-text]: images/html-template-plain-text.png
[html-template-noxss]: images/html-template-text-plain-noxss.png

[0]: https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project
[1]: https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_(XSS)
[2]: https://golang.org/pkg/html/template/
[3]: https://golang.org/pkg/net/http/
[4]: https://golang.org/pkg/io/
[5]: https://mimesniff.spec.whatwg.org/#rules-for-identifying-an-unknown-mime-typ
[6]: https://golang.org/pkg/text/template/
