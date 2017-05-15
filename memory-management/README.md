記憶體管理
=================

關於記憶體管理有許多重要的面向你需要考量。根據 OWASP 的指導原則，第一步我們必須要保護使用者的輸入和輸出，確保沒有任何惡意的內容。更多細節可以參考[輸入驗證][1] 和 [輸出編碼][2] 等章節。

另一個關於記憶體管理的重要面向是緩存邊界的檢查。通常以 C 風格的程式語言處理接受多個位元組複製的函式時，必須要檢查目標陣列的大小，以確保我們不會超過分配的空間。在 Go 中，像是 `String` 這種資料型態不會因為 NULL 而中止，同時他的 `header` 會包含以下資訊：

```go
type StringHeader struct {
    Data uintptr
    Len  int
}
```

儘管如此，邊界檢查還是必須要進行(例如：在迴圈中)，如果你超過邊界時，Go 會拋出 `panic`。

一個範例如下：

```go
func main() {
    strings := []string{"aaa", "bbb", "ccc", "ddd"}
    // Our loop is not checking the MAP length -> BAD
    for i := 0; i < 5; i++ {
        if len(strings[i]) > 0 {
            fmt.Println(strings[i])
        }
    }
}
```

輸出：

```
aaa
bbb
ccc
ddd
panic: runtime error: index out of range
```

當我們的應用程式中有使用到其他資源時，必須要確保程式碼中有正確的關閉該資源，而不僅僅是依賴編譯器的垃圾回收機制。這當你在處理物件連線或檔案處理時都適用。在 Go 中，你可以使用 `defer` 來進行這樣的處理。`defer` 會在函式結束離開前執行他的敘述。

```go
defer func() {
    // Our cleanup code here
}
```

更多關於 `defer` 的資訊可以在 [錯誤處理][3] 的章節中閱讀。

而一些已知較為不安全的函式也應該避免使用，在 Go 中，`unsafe` 套件中就包含了這些函式。這些函式不應該被用在正式環境中。

另一方面，記憶體釋放是由垃圾處理器來進行，代表開發者不需要去擔心這一部份。而有趣的事，開發者自己也可以去手動的釋放記憶體資源，儘管這並不建議。

引用 [Golang's Github](https://github.com/golang/go/issues/13761) 這個 issue：

> 如果你真的想要自己手動來管理記憶體資源，你可以根據 syscall.Mmap 或是 cgo malloc/free 來實作。
>
> 停用 GC 對於像 Go 這樣的並行語言來說是一個很糟糕的決定，而 Go 的 GC 只會越來越好。

[1]: /input-validation/README.md
[2]: /output-encoding/README.md
[3]: /error-handling-logging/README.md
