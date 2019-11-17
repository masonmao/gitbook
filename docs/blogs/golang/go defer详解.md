# 理解Golang中defer的使用

转自：https://blog.csdn.net/huang_yong_peng/article/details/82950743

之前一直对Go中的defer不太理解，所以我单独弄出来整理一下。
在golang当中，defer代码块会在函数调用链表中增加一个函数调用。这个函数调用不是普通的函数调用，而是会在函数正常返回，也就是return之后添加一个函数调用。因此，defer通常用来释放函数内部变量。
通过defer，我们可以在代码中优雅的关闭/清理代码中所使用的变量。defer作为golang清理变量的特性，有其独有且明确的行为。以下是defer三条使用规则。

## 规则一 当defer被声明时，其参数就会被实时解析

我们通过以下代码来解释这条规则:

```go
func a() {
	i := 0
	defer fmt.Println(i)
	i++
	return
}
```

运行结果是0
这是因为虽然我们在defer后面定义的是一个带变量的函数: fmt.Println(i). 但这个变量(i)在defer被声明的时候，就已经确定其确定的值了。 换言之，上面的代码等同于下面的代码:

```go
func a() {
	i := 0
	defer fmt.Println(0) //因为i=0，所以此时就明确告诉golang在程序退出时，执行输出0的操作
	i++
	return
}
```

为了更为明确的说明这个问题，我们继续定义一个defer:

```go
func a() {
	i := 0
	defer fmt.Println(i) //输出0，因为i此时就是0
	i++
	defer fmt.Println(i) //输出1，因为i此时就是1
	return
}
```

通过运行结果，可以看到defer输出的值，就是定义时的值。而不是defer真正执行时的变量值(很重要，搞不清楚的话就会产生于预期不一致的结果)

再看一个例子：

```go
package main

import "fmt"
func f1() (result int) {
	defer func() {
		result++
	}()
	return 0
}

func f2() (r int) {
	t := 5
	defer func() {
		t = t+5
	}()
	return t
}

func f3() (t int) {
	t = 5
	defer func() {
		t = t+5
	}()
	return t
}
func f4() (r int) {
	defer func(r int) {
		r = r + 5
	}(r)
	return 1
}

func main() {
	fmt.Println(f1())
	fmt.Println(f2())
	fmt.Println(f3())
	fmt.Println(f4())
}
```

运行结果是:

```
1
5
10
1
```

函数返回的过程是这样子的：先给返回值赋值，然后调用defer表达式，最后才是返回到调用函数中。

defer表达式可能会在设置函数返回值之后，在返回到调用函数之前，修改返回值，使最终的函数返回值与你想象的不一致。

可以将return xxx改成
返回值=xxx
调用defer函数
空的return

那上面的例子就可以改成：

```go
func f11() (result int) {
	result = 0   //先给返回值赋值
	func(){               //再执行defer 函数
		result++
	}()
	return                //最后返回
}

func f22() (r int) {
	t := 5
	r = t //赋值指令
	func() {  //defer 函数被插入到赋值与返回之间执行，这个例子中返回值r没有被修改
		t = t+5
	}
	return   //返回
}

func f33() (t int) {
	t = 5    //赋值指令
	func(){
		t = t+5  //然后执行defer函数,t值被修改
	}
	return
}
func f44() (r int) {
	r = 1    //给返回值赋值
	func(r int){   //这里的r传值进去的，是原来r的copy，不会改变要返回的那个r值
		r = r+5
	}(r)
	return
}
```

## 规则二 defer执行顺序为先进后出

当同时定义了多个defer代码块时，golang安装先定义后执行的顺序依次调用defer。不要为什么，golang就是这么定义的。我们用下面的代码加深记忆和理解:

```go
func b() {
	for i := 0; i < 4; i++ {
		defer fmt.Print(i)
	}
}
```

在循环中，依次定义了四个defer代码块。结合规则一，我们可以明确得知每个defer代码块应该输出什么值。 安装先进后出的原则，我们可以看到依次输出了3210.

再看之前的那个例子：

```go
package main
 
import "fmt"
 
func main() {
    fmt.Println("a return:", a()) // 打印结果为 a return: 0
}
 
func a() int {
    var i int
 
    defer func() {
        i++
        fmt.Println("a defer2:", i) // 打印结果为 a defer2: 2
    }()
 
    defer func() {
        i++
        fmt.Println("a defer1:", i) // 打印结果为 a defer1: 1
    }()
 
    return i
}
```

结果是：

```
a defer1: 1
a defer2: 2
a return: 0
```

## 规则三 defer可以读取有名返回值

先看下面的代码:

```go
func c() (i int) {
	defer func() { i++ }()
	return 1
}
```

输出结果是12. 在开头的时候，我们说过defer是在return调用之后才执行的。 这里需要明确的是defer代码块的作用域仍然在函数之内，结合上面的函数也就是说，defer的作用域仍然在c函数之内。因此defer仍然可以读取c函数内的变量(如果无法读取函数内变量，那又如何进行变量清除呢…)。
当执行return 1 之后，i的值就是1. 此时此刻，defer代码块开始执行，对i进行自增操作。 因此输出2.

看这个例子：

```go
package main

import "fmt"
func trace(s string) string {
	fmt.Println("entering:",s)
	return s
}

func un(s string) {
	fmt.Println("leaving:",s)
}

func a() {
	defer un(trace("a"))
	fmt.Println("in a")
}

func b() {
	defer un(trace("b"))
	fmt.Println("in b")
	a()
}

func main() {
	b()
}
```

运行结果是：

```
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

