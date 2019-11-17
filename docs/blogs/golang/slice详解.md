# slice详解

转自：https://www.jianshu.com/p/843aa553b461

## 1、简介

Go的 **Slice（切片）**类型提供了一种方便有效的方法来处理类型化数据序列。 slice类似于其他语言中的数组，但具有一些不寻常的属性。 本文将介绍切片是什么以及如何使用它们。

## 2、Slices

数组有它们的位置，但是它们有点不灵活，所以你不会在Go代码中经常看到它们。 然而，Slice无处不在。 它们以阵列为基础，提供强大的功能和便利性。
 Slice的类型规范是[] T，其中T是Slice元素的类型。 与数组类型不同，Slice类型没有指定的长度。
 Slice文字声明就像数组文字一样，除了省略元素数：

```go
letters := []string{"a", "b", "c", "d"}
```

可以使用名为make的内置函数创建切片，该函数具有如下定义，

```go
func make([]T, len, cap) []T
```

其中T代表要创建的切片的元素类型。 make函数采用类型，长度和可选容量。 调用时，make会分配一个数组并返回一个引用该数组的切片。

```go
var s []byte
s = make([]byte, 5, 5)
// s == []byte{0, 0, 0, 0, 0}
```

省略capacity参数时，默认为指定的长度。 这是相同代码的更简洁版本：

```go
s := make([]byte, 5)
```

可以使用内置的len和cap函数检查切片的长度和容量。

```go
len(s) == 5
cap(s) == 5
```

接下来的两节讨论长度和容量之间的关系。
 切片的零值为nil。 对于nil，len和cap函数都将返回0。
 还可以通过“切片”现有切片或阵列来形成切片。 通过指定半开放范围来完成切片，其中两个索引用冒号分隔。 例如，表达式b [1：4]创建包括b的元素1到3的切片（得到的切片的索引将是0到2）。

```go
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, sharing the same storage as b
```

切片表达式的开始和结束索引是可选的; 它们分别默认为零和切片长度：

```bash
// b[:2] == []byte{'g', 'o'}
// b[2:] == []byte{'l', 'a', 'n', 'g'}
// b[:] == b
```

这也是给定数组创建切片的语法：

```go
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // a slice referencing the storage of x
```

## 3、Slice内部结构剖析

切片是数组段的描述符。 它由指向数组的指针，段的长度及其容量（段的最大长度）组成。





![img](https:////upload-images.jianshu.io/upload_images/7179784-8cb4e76996a1482f.png?imageMogr2/auto-orient/strip|imageView2/2/w/517/format/webp)





我们之前由 make([] byte, 5) 创建的变量s的结构如下：





![img](https:////upload-images.jianshu.io/upload_images/7179784-8cf9c39bcec5a272.png?imageMogr2/auto-orient/strip|imageView2/2/w/517/format/webp)




 长度是切片引用的元素数。 容量是底层数组中元素的数量（从切片指针引用的元素开始）。 我们将通过接下来的几个例子来说明长度和容量之间的区别。 在切片时，观察切片数据结构中的变化及其与底层数组的关系： 



![img](https:////upload-images.jianshu.io/upload_images/7179784-42e4070017fea073.png?imageMogr2/auto-orient/strip|imageView2/2/w/517/format/webp)





切片不会复制切片的数据。 它创建一个指向原始数组的新切片值。 这使切片操作与操作数组索引一样高效。 因此，修改重新切片的元素（而不是切片本身）会修改原始切片的元素：

```go
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:] 
// e == []byte{'a', 'd'}
e[1] = 'm'
// e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'}
```

早些时候我们将s切成比其容量短的长度。 我们可以通过再次切片来增加容量：

```bash
s = s[:cap(s)]
```



![img](https:////upload-images.jianshu.io/upload_images/7179784-42ce90a707f02396.png?imageMogr2/auto-orient/strip|imageView2/2/w/517/format/webp)





切片不能超出其容量。 尝试这样做会导致运行时出现混乱，就像在切片或数组的边界之外进行索引一样。 类似地，切片不能在零以下重新切片以访问数组中的早期元素。

## 4、Slice增长（复制和追加功能）

要增加切片的容量，必须创建一个新的更大的切片并将原始切片的内容复制到切片中。 这种技术是其他语言的动态数组实现在幕后工作的方式。 下一个例子通过创建一个新的切片t，将s的内容复制到t，然后将切片值t分配给s，使s的容量加倍：

```go
t := make([]byte, len(s), (cap(s)+1)*2) // +1 in case cap(s) == 0
for i := range s {
        t[i] = s[i]
}
s = t
```

通过内置复制功能，可以更轻松地完成此常用操作的循环操作。 顾名思义，复制将数据从源切片复制到目标切片。 它返回复制的元素数。

```swift
func copy(dst, src []T) int
```

复制功能支持在不同长度的切片之间进行复制（它将仅复制到较少数量的元素）。 此外，copy可以处理共享相同底层数组的源和目标片，正确处理重叠片。
 使用copy，我们可以简化上面的代码片段：

```go
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
s = t
```

常见的操作是将数据附加到切片的末尾。 此函数将字节元素附加到一个字节切片，必要时生成切片，并返回更新的切片值：

```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) { // if necessary, reallocate
        // allocate double what's needed, for future growth.
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
```

可以像这样使用AppendByte：

```go
p := []byte{2, 3, 5}
p = AppendByte(p, 7, 11, 13)
// p == []byte{2, 3, 5, 7, 11, 13}
```

像AppendByte这样的函数非常有用，因为它们可以完全控制切片的生成方式。 根据程序的特性，可能需要以更小或更大的块分配，或者对重新分配的大小设置上限。
 但是大多数程序不需要完全控制，所以Go提供了一个内置的附加功能，这对大多数用途都有好处; 它有签名

```css
func append(s []T, x ...T) []T
```

append函数将元素x附加到切片s的末尾，如果需要更大的容量，则增加切片。

```go
a := make([]int, 1)
// a == []int{0}
a = append(a, 1, 2, 3)
// a == []int{0, 1, 2, 3}
```

要将一个切片附加到另一个切片，请使用...将第二个参数展开为参数列表。

```go
a := []string{"John", "Paul"}
b := []string{"George", "Ringo", "Pete"}
a = append(a, b...) // equivalent to "append(a, b[0], b[1], b[2])"
// a == []string{"John", "Paul", "George", "Ringo", "Pete"}
```

由于切片的零值（nil）就像零长度切片一样，您可以声明切片变量，然后在循环中追加它：

```go
// Filter returns a new slice holding only
// the elements of s that satisfy fn()
func Filter(s []int, fn func(int) bool) []int {
    var p []int // == nil
    for _, v := range s {
        if fn(v) {
            p = append(p, v)
        }
    }
    return p
}
```

## 5、一个可能的“陷阱”

如前所述，重新切分切片不会复制底层数组。 完整数组将保留在内存中，直到不再引用它为止。 偶尔这会导致程序在只需要一小部分数据时将所有数据保存在内存中。
 例如，此FindDigits函数将文件加载到内存中，并在其中搜索第一组连续数字数字，并将它们作为新切片返回。

```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```

此代码的行为与广告一样，但返回的 []byte 指向包含整个文件的数组。 由于切片引用原始数组，只要切片保持在垃圾收集器周围就不能释放数组; 文件中几个有用的字节将整个内容保存在内存中。
 要解决此问题，可以在返回之前将有趣数据复制到新切片：

```go
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```

可以使用append构造此函数的更简洁版本。

## 6、他山之石

小编之所以写这片文章，主要是由于工作中遇到一个由slice用法了解不够透彻导致的bug，花费了超过9个小时的时间，在此简要介绍，供大家学习。

#### 6.1、问题介绍

公司使用的框架是thrift，rpc调用远程接口时，需要传递一个数组，本意是希望初始化一个容量为3的数组。小编使用如下方式初始化：

> op = make([]*OneStruct, 3)  // 注意：这是一种错误写法

在发起rpc调用后，代码一直提示panic，且返回值为nil。跟Server端同学联调，对方压根没有收到请求。

#### 6.2、问题分析

经过上文的分析，我们知道：make([]T, 3) 的含义是len=3，且 cap=3的数组，此时数组中已经存在3个nil的元素了。当发起thrift调用时，thrift框架检测到参数不正确，直接返回panic

#### 6.3、解决方案

将初始化方法改为：

```go
op = make([]*OneStruct, 0, 3)
```

#### 6.4、问题总结

对golang语法了解不深入，需要加强对基础语法的学习。同时吐槽一下thrift框架，参数传递不正确，直接告诉我具体某个参数不正确多好，抛了一大堆异常很难定位问题。