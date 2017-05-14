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

When our application uses resources, additional checks must also be made to ensure they have been closed and not rely solely on the Garbage Collector. This is applicable when dealing with connection objects, file handles, etc. In Go we can use `Defer` to perform these actions. Instructions in `Defer` are only executed when the surrounding functions finish execution.

```go
defer func() {
    // Our cleanup code here
}
```

More information regarding `Defer` can be found in the [Error Handling][3] section of the document.

Usage of known vulnerable functions should also be avoided. In Go, the `Unsafe` package contains these functions. They should not be used in production environments, nor should the package itself. This also applies to the `Testing` package.

On the other hand, memory deallocation is handled by the garbage collector, which means that we don't have to worry about it. An interesting note is that it _is_ possible to manually deallocate memory although it is **not** advised.

Quoting [Golang's Github](https://github.com/golang/go/issues/13761):

> If you really want to manually manage memory with Go, implement your own memory allocator based on syscall.Mmap or cgo malloc/free.
>
> Disabling GC for extended period of time is generally a bad solution for a concurrent language like Go. And Go's GC will only be better down the road.

[1]: /input-validation/README.md
[2]: /output-encoding/README.md
[3]: /error-handling-logging/README.md
