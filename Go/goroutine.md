# goroutine

Go语言从语言层面原生提供了协程支持，即 `goroutine`，执行goroutine只需极少的栈内存(大概是4~5KB)，所以Go可以轻松的运行多个并发任务。

Go中以关键字 `go` 开启协程：

~~~go
func say(s string) {
    for i := 0; i < 3; i++ {
        fmt.Println(s)
    }
}

func main() {
    go say("Go")						// 以协程方式执行say函数
    say("noGo")							// 以普通方式执行say函数
    time.Sleep(5 * time.Second)         // 睡眠5秒：防止协程未执行完毕，主程序退出
}
~~~

上述的程序的打印结果并不是按照顺序的，因为go关键字开启了协程，两个say函数并不是在一个控制流中！

