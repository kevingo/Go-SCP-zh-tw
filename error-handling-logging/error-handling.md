錯誤處理
==============

在 Go 中有一個內建的 `error` 型別。不同的 `error` 型別值代表不正常的狀態。當 `error` 的值不是 `nil` 時，通常代表有錯誤產生，我們必須要捕捉錯誤並且處理他，避免系統產生無法預期的錯誤。

一個簡單的例子如下：

```go
if err != nil {
    // 處理錯誤
}
```

你不僅可以用內建的錯誤型別，也可以建立自己的錯誤型別。透過 `errors.New` 函式就可以達成。

範例：

```go
{...}
if f < 0 {
    return 0, errors.New("math: square root of negative number")
}
// 當錯誤產生時，將它印出來
if err != nil {
    fmt.Println(err)
}
{...}
```

一旦我們需要將錯誤的字串進行格式化後再印出時，在 `fmt` 套件中的 `Errorf` 函式可以幫助我們達到這樣的目的。

```go
{...}
if f < 0 {
    return 0, fmt.Errorf("math: square root of negative number %g", f)
}
{...}
```

當我們處理錯誤日誌時，開發者必須確保沒有敏感的資訊被揭露在日誌中，同時也要確保沒有錯誤 handler 所產生的資訊(例如：debugging 資訊、stack trace 的資訊等)

在 Go 中，還有一些其他錯誤處理的函式，那就是 `panic`、`recover` 和 `defer`。當系統的狀態變為 `panic` 時，正常的流程會被終止，任何 `defer` 的描述句會被執行，同時正在執行的函式會被返回到呼叫他的呼叫者。`recover` 通常會用在 `defer` 敘述中，讓系統重 `panicking` 的 routine 回復到正常執行的狀態。

以下的程式片段描述了大致的流程：

```go
func main () {
    start()
    fmt.Println("Returned normally from start().")
}

func start () {
    defer func () {
        if r := recover(); r != nil {
            fmt.Println("Recovered in start()")
        }
    }()
    fmt.Println("Called start()")
    part2(0)
    fmt.Println("Returned normally from part2().")
}

func part2 (i int) {
    if i > 0 {
        fmt.Println("Panicking in part2()!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in part2()")
    fmt.Println("Executing part2()")
    part2(i + 1)
}
```

輸出：

```
Called start()
Executing part2()
Panicking in part2()!
Defer in part2()
Recovered in start()
Returned normally from start().
```

藉由上面的輸出我們可以了解 Go 是如何處理 `panic` 的狀況，並且從 `panic` 回復到正常狀態。`recover` 函式讓你可以平順的回復到正常的狀態。

值得注意的是，`defer` 的功用還包含 `Mutex Unlocking`。或是執行周圍的函式後再讀取等功用。

在 `log` 套件中也有一個 `log.Fatal` 的函式，`Fatal` 等級的記錄也是有用的，接著可以呼叫 `os.Exit(1)`。這代表了：

* Defer 描述不會被執行
* Buffers 不會被清空
* 暫存檔案或目錄不會被移除 

考慮到上述幾點，我們可以知道 `log.Fatal` 跟 `panic` 有所不同，因此你必須要小心使用。一些 `log.Fatal` 可能的使用情境有：

* 設定日誌檢查，確認我們確認我們有正確的配置環境變數和參數，如果沒有時，就不需要執行 main 函式。
* 一個不應該發生的錯誤，同時我們知道這個錯誤是無法被回復到正常狀態的。
* 如果在一個非互動的過程中發生錯誤，同時這個時間點無法通知使用者，最好的方式就是停止執行，以免發生更多額外的錯誤。

底下是一個範例：

```go
func init(i int) {
    ...
    // 這是一個故意造成錯誤結束的函式
    if i < 2 {
        fmt.Printf("Var %d - initialized\n", i)
    } else {
        // 這裡不應該被發生，所以我們強制終止程式
        log.Fatal("Init failure - Terminating.")
    }
}

func main() {
    i := 1
    for i < 3 {
        init(i)
        i++
    }
    fmt.Println("Initialized all variables successfully")
```

重要的是在確保任何跟安全有關的錯誤發生的情況下，預設必須要拒絕任何存取。
