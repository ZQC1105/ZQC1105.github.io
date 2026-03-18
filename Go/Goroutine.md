# Go Goroutine 完整总结

## 一、什么是 Goroutine？

**Goroutine** 是 Go 语言中的轻量级线程，由 Go 运行时管理。它可以让你轻松实现并发编程。

```go
// 启动一个 goroutine
go function()
// 或
go func() {
    // 并发执行的代码
}()
```

## 二、核心特性

| 特性         | 说明                             |
| ------------ | -------------------------------- |
| **轻量级**   | 初始栈大小仅 2KB，可动态增长     |
| **低成本**   | 创建数十万个 goroutine 也没问题  |
| **简单语法** | 只需在函数调用前加 `go` 关键字   |
| **并发模型** | 基于 CSP 模型，通过 channel 通信 |

## 三、基本用法

### 1. 启动 Goroutine

```go
// 方式1：使用命名函数
func printMessage() {
    fmt.Println("Hello")
}
go printMessage()

// 方式2：使用匿名函数
go func() {
    fmt.Println("World")
}()

// 方式3：带参数的匿名函数
go func(name string) {
    fmt.Println("Hello", name)
}("张三")
```

### 2. 等待 Goroutine 完成 - sync.WaitGroup

```go
func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 3; i++ {
        wg.Add(1) // 计数器+1

        go func(n int) {
            defer wg.Done() // 完成后计数器-1
            fmt.Println("goroutine", n)
            time.Sleep(time.Second)
        }(i)
    }

    wg.Wait() // 等待所有 goroutine 完成
    fmt.Println("所有任务完成")
}
```

### 3. Goroutine 间通信 - Channel

```go
// 无缓冲 channel
ch := make(chan string)

// 有缓冲 channel
bufferedCh := make(chan int, 10)

// 示例：生产者-消费者
func producer(ch chan<- int) {
    for i := 1; i <= 5; i++ {
        ch <- i
        fmt.Println("生产:", i)
        time.Sleep(time.Millisecond * 500)
    }
    close(ch)
}

func consumer(ch <-chan int) {
    for num := range ch {
        fmt.Println("消费:", num)
        time.Sleep(time.Second)
    }
}

func main() {
    ch := make(chan int, 3)
    go producer(ch)
    consumer(ch)
}
```

## 四、常见陷阱与解决方案

### 1. 循环变量捕获问题

```go
// ❌ 错误：所有 goroutine 共享同一个 i
for i := 1; i <= 3; i++ {
    go func() {
        fmt.Println(i) // 可能打印 4,4,4
    }()
}

// ✅ 正确：通过参数传递值
for i := 1; i <= 3; i++ {
    go func(n int) {
        fmt.Println(n) // 打印 1,2,3
    }(i)
}

// ✅ 正确：创建新变量
for i := 1; i <= 3; i++ {
    i := i // 创建副本
    go func() {
        fmt.Println(i) // 打印 1,2,3
    }()
}
```

### 2. 主 goroutine 提前退出

```go
// ❌ 错误：主函数退出，goroutine 没机会执行
func main() {
    go fmt.Println("不会执行")
}

// ✅ 正确：等待 goroutine 完成
func main() {
    var wg sync.WaitGroup
    wg.Add(1)

    go func() {
        defer wg.Done()
        fmt.Println("会执行")
    }()

    wg.Wait()
}
```

### 3. 数据竞争

```go
// ❌ 错误：多个 goroutine 同时修改同一变量
counter := 0
for i := 0; i < 1000; i++ {
    go func() {
        counter++ // 数据竞争！
    }()
}

// ✅ 正确：使用互斥锁
var mu sync.Mutex
counter := 0
for i := 0; i < 1000; i++ {
    go func() {
        mu.Lock()
        counter++
        mu.Unlock()
    }()
}

// ✅ 正确：使用原子操作
var counter int64
for i := 0; i < 1000; i++ {
    go func() {
        atomic.AddInt64(&counter, 1)
    }()
}
```

## 五、并发模式

### 1. Worker Pool

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("worker %d 处理任务 %d\n", id, job)
        time.Sleep(time.Second)
        results <- job * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)

    // 启动 3 个 worker
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }

    // 发送 5 个任务
    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)

    // 收集结果
    for r := 1; r <= 5; r++ {
        <-results
    }
}
```

### 2. 超时控制

```go
func doWorkWithTimeout() error {
    ch := make(chan string)

    go func() {
        // 模拟耗时操作
        time.Sleep(3 * time.Second)
        ch <- "完成"
    }()

    select {
    case result := <-ch:
        fmt.Println(result)
        return nil
    case <-time.After(2 * time.Second):
        return fmt.Errorf("操作超时")
    }
}
```

### 3. 扇出/扇入

```go
// 扇出：一个函数同时向多个 channel 发送数据
// 扇入：多个 goroutine 的结果合并到一个 channel

func fanOut(ch <-chan int, outs []chan<- int) {
    for v := range ch {
        for _, out := range outs {
            out <- v
        }
    }
}

func fanIn(chs ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup

    for _, ch := range chs {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for v := range c {
                out <- v
            }
        }(ch)
    }

    go func() {
        wg.Wait()
        close(out)
    }()

    return out
}
```

## 六、错误处理

### 1. 捕获 Panic

```go
go func() {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("goroutine panic 恢复: %v", r)
        }
    }()

    // 可能 panic 的代码
    panic("出错啦")
}()
```

### 2. 错误返回

```go
type Result struct {
    Value interface{}
    Err   error
}

func doSomething() <-chan Result {
    ch := make(chan Result)

    go func() {
        defer close(ch)

        // 执行操作
        result, err := someOperation()
        ch <- Result{Value: result, Err: err}
    }()

    return ch
}
```

## 七、性能优化

### 1. 控制并发数量

```go
func processWithLimit(items []string, maxConcurrent int) {
    sem := make(chan struct{}, maxConcurrent)
    var wg sync.WaitGroup

    for _, item := range items {
        wg.Add(1)

        go func(item string) {
            defer wg.Done()

            sem <- struct{}{}        // 获取令牌
            defer func() { <-sem }()  // 释放令牌

            process(item) // 处理项目
        }(item)
    }

    wg.Wait()
}
```

### 2. 重用 goroutine

```go
type Pool struct {
    work chan func()
    wg   sync.WaitGroup
}

func NewPool(size int) *Pool {
    p := &Pool{
        work: make(chan func()),
    }

    p.wg.Add(size)
    for i := 0; i < size; i++ {
        go p.worker()
    }

    return p
}

func (p *Pool) worker() {
    defer p.wg.Done()
    for fn := range p.work {
        fn()
    }
}

func (p *Pool) Submit(fn func()) {
    p.work <- fn
}

func (p *Pool) Close() {
    close(p.work)
    p.wg.Wait()
}
```

## 八、最佳实践总结

| 场景                   | 推荐做法                     |
| ---------------------- | ---------------------------- |
| **等待多个 goroutine** | 使用 `sync.WaitGroup`        |
| **goroutine 间通信**   | 使用 channel，而不是共享内存 |
| **避免数据竞争**       | 使用 mutex 或 atomic 操作    |
| **控制并发数量**       | 使用 worker pool 或信号量    |
| **循环中使用**         | 通过参数传递循环变量         |
| **超时控制**           | 使用 `select` + `time.After` |
| **优雅退出**           | 使用 `context.Context`       |
| **错误处理**           | 通过 channel 返回错误        |

## 九、常用模式代码模板

```go
// 标准 goroutine 模板
func standardGoroutine() {
    var wg sync.WaitGroup

    for _, item := range items {
        wg.Add(1)

        go func(item Item) {
            defer wg.Done()
            defer func() {
                if r := recover(); r != nil {
                    log.Printf("恢复 panic: %v", r)
                }
            }()

            // 处理逻辑
            process(item)
        }(item)
    }

    wg.Wait()
}

// 带取消的 goroutine
func withCancel(ctx context.Context) {
    go func() {
        select {
        case <-ctx.Done():
            log.Println("goroutine 被取消")
            return
        case result := <-doWork():
            log.Println("结果:", result)
        }
    }()
}
```

## 十、核心口诀

1. **不要通过共享内存来通信，而要通过通信来共享内存**
2. **启动 goroutine 时就要考虑如何停止它**
3. **永远不要假设 goroutine 的执行顺序**
4. **使用 WaitGroup 等待，不要用 sleep**
5. **在循环中启动 goroutine 时，传递参数的副本**

Goroutine 是 Go 语言最强大的特性之一，掌握好它，你就能轻松编写高效的并发程序！
