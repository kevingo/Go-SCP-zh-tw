資料清理
============

資料清理指的是移除或置換使用者提交上來的資料。當處理資料時，在進行適當的驗證後，另外一個步驟通常會進行資料清理。最常見的清理方法有：

## 將 `<` 轉為 entity

在內建的 `html` 套件中，有兩個函式是用來做資料清理的，一個是用來跳脫 HTML 中特殊的字元、另一個是用來將跳脫後的字元還原為原本的特殊字元。`EscapeString()` 函式接受一個字串的參數，並且回傳跳脫特殊字元後的內容。比如說 `<` 會變為 `&lt;`。注意，這個函式只會跳脫以下五個特殊符號：`<`、`>`、`&`、`'` 和 `"`。相反的，`UnescapeString()` 函式則是將上述五個字元還原為原本的符號。

## 移除所有的 tags

即使 `html/template` 套件有一個 `stripTags()` 函式，但它並不是公開的，所以你沒辦法透過 `template.stripTags()` 來使用它。而有一些第三方套件可以達成此目的，或是你可以複製整個函式來使用。

下面所列出的第三方套件可以達成上述的目標：

* [https://github.com/kennygrant/sanitize](https://github.com/kennygrant/sanitize)
* [https://github.com/maxwells/sanitize](https://github.com/maxwells/sanitize)

## 移除換行符號、tabs 和多餘的空白

`text/template` 和 `html/template` 兩個套件中都包含了從模板中移除空白的方法，你可以透過 `-` 來達成。

執行以下的模板：

```
{{- 23}} < {{45 -}}
```

你會看到這樣的輸出：

```
23<45
```

**注意**：如果 `-` 沒有跟在 `{{` 後面，或是 `}}` 前面時，它就會被附加到數值上。

以下：

```
{{ -3 }}
```

會變成：

```
-3
```

## URL 請求路徑

在 `net/http` 套件中，有一個處理 HTTP 請求的路由框架，叫做 `ServerMux`。`ServerMux` 用來將請求註冊應對到某個 pattern，並且呼叫對應的 handler 來處理這個請求。此外，他最主要的目的是處理 URL 請求路徑的清理，將包含 `.` 或 `..` 的路徑進行重新指向，產生一個更乾淨的 URL。

一個簡單的例子如下：

```go
func main() {
  mux := http.NewServeMux()

  rh := http.RedirectHandler("http://yourDomain.org", 307)
  mux.Handle("/login", rh)

  log.Println("Listening...")
  http.ListenAndServe(":3000", mux)
}
```

另外你也可以試試看第三方套件：

* [Gorilla Toolkit - MUX][1]

[1]: http://www.gorillatoolkit.org/pkg/mux
