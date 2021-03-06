# 简介
golang 实现了自己的 runtime，功能包括 GC、goroutine、内存分配等. 极大的提高了 go 语言的效率.

## tight loop
golang 1.14 版本之前，对于 tight loop 不能被 gc 中断. 对于小的 tight loop 的也并没有进行并发处理. 下述的代码中 A 的输出和 B 的输出并非交叉而是连续的.

        var wg1 sync.WaitGroup
        func a() {
            defer wg1.Done()
            for i := 0; i < 10; i++ {
                fmt.Println("A:", i)
            }
        }
        func b() {
            defer wg1.Done()
            for i := 0; i < 10; i++ {
                fmt.Println("B:", i)
            }
        }
        func testB() {
            // fmt.Println(runtime.NumCPU())
            runtime.GOMAXPROCS(12)
            wg1.Add(2)
            go a()
            go b()
            wg1.Wait()
        }

## runtime 库
NumCPU() 
GOMAXPROCS()
Gosched()
Goexit()
NumGoroutine()
GOOS

## 并发
n++ 并发安全问题

    func race() {
        num := 1000
        wait := make(chan struct{}, num)
        n := 0
        for i := 0; i < num; i++ {
            go func() {
                // 译注：注意下面这一行
                n++ // 一次访问: 读, 递增, 写
                wait <- struct{}{}
            }()
        }

        // 译注：注意下面这一行
        // n++ // 另一次冲突的访问
        for i := 0; i < num; i++ {
            <-wait
        }
        fmt.Println(n) // 输出：未指定
    }

## pprof
gcBgMarkWorker
mallocgc