# select 关键字用法

select是go语言中常用的一个关键字，其用法也一直被用作面试题来考核应聘者。今天，结合代码来分析下select的主要用法。

首先，我们来从官方文档看一下有关select的描述：

> A "select" statement chooses which of a set of possible send or receive operations will proceed. It looks similar to a "switch" statement but with the cases all referring to communication operations.
>  一个select语句用来选择哪个case中的发送或接收操作可以被立即执行。它类似于switch语句，但是它的case涉及到channel有关的I/O操作。

或者换一种说法，select就是用来监听和channel有关的IO操作，当 IO 操作发生时，触发相应的动作。

### 基本用法

```csharp
//select基本用法
select {
case <- chan1:
// 如果chan1成功读到数据，则进行该case处理语句
case chan2 <- 1:
// 如果成功向chan2写入数据，则进行该case处理语句
default:
// 如果上面都没有成功，则进入default处理流程
```

### 官方执行步骤

> Execution of a "select" statement proceeds in several steps:
>
> > 1.For all the cases in the statement, the channel operands of receive operations and the channel and right-hand-side expressions of send statements are evaluated exactly once, in source order, upon entering the "select" statement. The result is a set of channels to receive from or send to, and the corresponding values to send. Any side effects in that evaluation will occur irrespective of which (if any) communication operation is selected to proceed. Expressions on the left-hand side of a RecvStmt with a short variable declaration or assignment are not yet evaluated.
> >  所有channel表达式都会被求值、所有被发送的表达式都会被求值。求值顺序：自上而下、从左到右.
> >  结果是选择一个发送或接收的channel，无论选择哪一个case进行操作，表达式都会被执行。RecvStmt左侧短变量声明或赋值未被评估。
> >
> > 1. If one or more of the communications can proceed, a single one that can proceed is chosen via a uniform pseudo-random selection. Otherwise, if there is a default case, that case is chosen. If there is no default case, the "select" statement blocks until at least one of the communications can proceed.
> >     如果有一个或多个IO操作可以完成，则Go运行时系统会随机的选择一个执行，否则的话，如果有default分支，则执行default分支语句，如果连default都没有，则select语句会一直阻塞，直到至少有一个IO操作可以进行.
> >
> > 3.Unless the selected case is the default case, the respective communication operation is executed.
> >  除非所选择的情况是默认情况，否则执行相应的通信操作。
> >
> > 4.If the selected case is a RecvStmt with a short variable declaration or an assignment, the left-hand side expressions are evaluated and the received value (or values) are assigned.
> >  如果所选case是具有短变量声明或赋值的RecvStmt，则评估左侧表达式并分配接收值（或多个值）。
> >
> > 5.The statement list of the selected case is executed.
> >  执行所选case中的语句

### 案例分析

##### 案例1  如果有一个或多个IO操作可以完成，则Go运行时系统会随机的选择一个执行，否则的话，如果有default分支，则执行default分支语句，如果连default都没有，则select语句会一直阻塞，直到至少有一个IO操作可以进行

```go
start := time.Now()
    c := make(chan interface{})
    ch1 := make(chan int)
        ch2 := make(chan int)

    go func() {

        time.Sleep(4*time.Second)
        close(c)
    }()

    go func() {

        time.Sleep(3*time.Second)
        ch1 <- 3
    }()

      go func() {

        time.Sleep(3*time.Second)
        ch2 <- 5
    }()

    fmt.Println("Blocking on read...")
    select {
    case <- c:

        fmt.Printf("Unblocked %v later.\n", time.Since(start))

    case <- ch1:

        fmt.Printf("ch1 case...")
      case <- ch2:

        fmt.Printf("ch1 case...")
    default:

        fmt.Printf("default go...")
    }
```

运行上述代码，由于当前时间还未到3s。所以，目前程序会走default。

```bash
Blocking on read...
default go...
Process finished with exit code 0
```

修改代码，将default注释:

```cpp
//default:
 //       fmt.Printf("default go...")
```

这时，select语句会阻塞，直到监测到一个可以执行的IO操作为止。这里，先会执行完睡眠3s的gorountine,此时两个channel都满足条件，这时系统会随机选择一个case继续操作。

```bash
Blocking on read...
ch2 case...
Process finished with exit code 0
```

接着，继续修改代码，将ch的gorountine休眠时间改为5s：

```go
go func() {

        time.Sleep(5*time.Second)
        ch1 <- 3
    }()
go func() {

        time.Sleep(5*time.Second)
        ch2 <- 3
    }()
```

此时会先执行到上面的gorountine，select执行的就是c的case。

```bash
Blocking on read...
Unblocked 4.000612584s later.
Process finished with exit code 0
```

##### 示例2  所有channel表达式都会被求值、所有被发送的表达式都会被求值。求值顺序：自上而下、从左到右.

```go
var ch1 chan int
var ch2 chan int
var chs = []chan int{ch1, ch2}
var numbers = []int{1, 2, 3, 4, 5}

func main () {

    select {
    case getChan(0) <- getNumber(2):

        fmt.Println("1th case is selected.")
    case getChan(1) <- getNumber(3):

        fmt.Println("2th case is selected.")
    default:

        fmt.Println("default!.")
        }
        }

func getNumber(i int) int {
    fmt.Printf("numbers[%d]\n", i)

    return numbers[i]
}
func getChan(i int) chan int {
    fmt.Printf("chs[%d]\n", i)

    return chs[i]
}
```

此时，select语句走的是default操作。但是这时每个case的表达式都会被执行。以case1为例：

```bash
case getChan(0) <- getNumber(2):
```

系统会从左到右先执行getChan函数打印chs[0]，然后执行getNumber函数打印numbers[2]。同样，从上到下分别执行所有case的语句。所以，程序执行的结果为：

```css
chs[0]
numbers[2]
chs[1]
numbers[3]
default!.

Process finished with exit code 0
```

##### 示例3 break关键字结束select

```go
ch1 := make(chan int, 1)
    ch2 := make(chan int, 1)

    ch1 <- 3
    ch2 <- 5

    select {
    case <- ch1:

        fmt.Println("ch1 selected.")

        break

        fmt.Println("ch1 selected after break")
    case <- ch2:

        fmt.Println("ch2 selected.")
        fmt.Println("ch2 selected without break")
    }
```

很明显，ch1和ch2两个通道都可以读取到值，所以系统会随机选择一个case执行。我们发现选择执行ch1的case时，由于有break关键字只执行了一句：

```bash
ch1 selected.

Process finished with exit code 0
```

但是，当系统选择ch2的case时，打印结果为：

```bash
ch2 selected.
ch2 selected without break

Process finished with exit code 0
```

如此就显而易见，break关键字在select中的作用。