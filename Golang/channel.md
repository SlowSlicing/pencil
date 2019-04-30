[toc]

　　channel 是 Go 语言在语言级别提供的 goroutine 间的通信方式。我们可以使用 channel 在两个或多个 goroutine 之间传递消息。channel 是进程内的通信方式，因此通过 channel 传递对象的过程和调用函数时的参数传递行为比较一致，比如也可以传递指针等。如果需要跨进程通信，建议用分布式系统的方法来解决，比如使用 Socket 或者 HTTP 等通信协议。Go 语言对于网络方面也有非常完善的支持。

　　channel 是类型相关的。也就是说，一个 channel 只能传递一种类型的值，这个类型需要在声明 channel 时指定。如果对 Unix 管道有所了解的话，就不难理解 channel，可以将其认为是一种类型安全的管道。

　　在了解 channel 的语法前，我们先看下用 channel 的方式重写上面的例子是什么样子的，以此对 channel 先有一个直感的认识，如下代码清单所示。

```
package main

import "fmt"

func Count(ch chan int) {
	ch <- 1
	fmt.Println("Counting")
}

func main() {
	chs := make([]chan int， 10)
	for i := 0; i < 10; i++ {
		chs[i] = make(chan int)
		go Count(chs[i])
	}
	
	for _, ch := range(chs) {
		<-ch
	}
}
```

　　在这个例子中，定义了一个包含10个 channel 的数组（名为 chs），并把数组中的每个 channel 分配给10个不同的 goroutine。在每个 goroutine 的 Add() 函数完成后，我们通过 `ch <- 1` 语句向对应的 channel 中写入一个数据。在这个 channel 被读取前，这个操作是阻塞的。在所有的 goroutine 启动完成后，我们通过 `<-ch` 语句从10个 channel 中依次读取数据。在对应的 channel 写入数据前，这个操作也是阻塞的。这样，我们就用 channel 实现了类似锁的功能，进而保证了所有 goroutine 完成后主函数才返回。是不是比共享内存的方式更简单、优雅呢?

　　我们在使用 Go 语言开发时，经常会遇到需要实现条件等待的场景，这也是 channel 可以发挥作用的地方。对 channel 的熟练使用，才能真正理解和掌握 Go 语言并发编程。

# 基本语法

　　一般 channel 的声明形式为：

```
var chanName chan ElementType
```

　　与一般的变量声明不同的地方仅仅是在类型之前加了 chan 关键字。ElementType 指定这个 channel 所能传递的元素类型。举个例子，声明一个传递类型为 int 的 channel：

```
var ch chan int
```

　　或者，声明一个 map，元素是 bool 型的 channel:

```
var m map[string] chan bool
```

　　上面的语句都是合法的。

　　定义一个 channel 也很简单，直接使用内置的函数 make() 即可：

```
ch := make(chan int)
```

　　这就声明并初始化了一个 int 型的名为 ch 的 channel。

　　在 channel 的用法中，最常见的包括写入和读出。将一个数据写入（发送）至 channel 的语法很直观，如下：

```
ch <- value
```

　　向 channel 写入数据通常会导致程序阻塞，直到有其他 goroutine 从这个 channel 中读取数据。从 channel 中读取数据的语法是：

```
value := <-ch
```

　　如果 channel 之前没有写入数据，那么从 channel 中读取数据也会导致程序阻塞，直到 channel 中被写入数据为止。之后还会提到如何控制 channel 只接受写或者只允许读取，即`单向 channel`。

# select

　　早在 Unix 时代，select 机制就已经被引入。通过调用 select() 函数来监控一系列的文件句柄，一旦其中一个文件句柄发生了 IO 动作，该 select() 调用就会被返回。后来该机制也被用于实现高并发的 Socket 服务器程序。Go 语言直接在语言级别支持 select 关键字，用于处理异步 IO 问题。

　　select 的用法与 switch 语言非常类似，由 select 开始一个新的选择块，每个选择条件由 case 语句来描述。与 switch 语句可以选择任何可使用相等比较的条件相比，select 有比较多的限制，其中最大的一条限制就是每个 case 语句里必须是一个 IO 操作，大致的结构如下：

```
select {
	case <-chan1:
		// 如果chan1成功读到数据，则进行该case处理语句
	case chan2 <- 1:
		// 如果成功向chan2写入数据，则进行该case处理语句
	default:
		// 如果上面都没有成功，则进入default处理流程
}
```

　　可以看出，select 不像 switch，后面并不带判断条件，而是直接去查看 case 语句。每个 case 语句都必须是一个面向 channel 的操作。比如上面的例子中，第一个 case 试图从 chan1 读取一个数据并直接忽略读到的数据，而第二个 case 则是试图向 chan2 中写入一个整型数 1，如果这两者都没有成功，则到达 default 语句。

　　基于此功能，我们可以实现一个有趣的程序：

```
ch := make(chan int, 1)

for {
	select {
		case ch <- 0:
		case ch <- 1:
	}
	i := <-ch
	fmt.Println("Value received:", i)
}
```

　　能看明白这段代码的含义吗？其实很简单，这个程序实现了一个随机向 ch 中写入一个 0 或者 1 的过程。当然，这是个死循环。

# 缓冲机制

　　之前示范创建的都是不带缓冲的 channel，这种做法对于传递单个数据的场景可以接受，但对于需要持续传输大量数据的场景就有些不合适了。接下来介绍如何给 channel 带上缓冲，从而达到消息队列的效果。

　　要创建一个带缓冲的 channel，其实也非常容易：

```
c := make(chan int, 1024)
```

　　在调用 make() 时将缓冲区大小作为第二个参数传入即可，比如上面这个例子就创建了一个大小为 1024 的 int 类型 channel，即使没有读取方，写入方也可以一直往 channel 里写入，在缓冲区被填完之前都不会阻塞。

　　从带缓冲的 channel 中读取数据可以使用与常规非缓冲 channel 完全一致的方法，但我们也可以使用 range 关键来实现更为简便的循环读取：

```
for i := range c {
	fmt.Println("Received:", i)
}
```

# 超时机制

　　在之前对 channel 的介绍中，我们完全没有提到错误处理的问题，而这个问题显然是不能被忽略的。在并发编程的通信过程中，最需要处理的就是超时问题，即向 channel 写数据时发现 channel 已满，或者从 channel 试图读取数据时发现 channel 为空。如果不正确处理这些情况，很可能会导致整个 goroutine 锁死。

　　虽然 goroutine 是 Go 语言引入的新概念，但通信锁死问题已经存在很长时间，在之前的 C/C++ 开发中也存在。操作系统在提供此类系统级通信函数时也会考虑入超时场景，因此这些方法通常都会带一个独立的超时参数。超过设定的时间时，仍然没有处理完任务，则该方法会立即终止并返回对应的超时信息。超时机制本身虽然也会带来一些问题，比如在运行比较快的机器或者高速的网络上运行正常的程序，到了慢速的机器或者网络上运行就会出问题，从而出现结果不一致的现象，但从根本上来说，解决死锁问题的价值要远大于所带来的问题。

　　使用 channel 时需要小心，比如对于以下这个用法：

```
i := <-ch
```

　　不出问题的话一切都正常运行。但如果出现了一个错误情况，即永远都没有人往 ch 里写数据，那么上述这个读取动作也将永远无法从 ch 中读取到数据，导致的结果就是整个 goroutine 永远阻塞并没有挽回的机会。如果 channel 只是被同一个开发者使用，那样出问题的可能性还低一些。但如果一旦对外公开，就必须考虑到最差的情况并对程序进行保护。

　　Go 语言没有提供直接的超时处理机制，但我们可以利用 select 机制。虽然 select 机制不是专为超时而设计的，却能很方便地解决超时问题。因为 select 的特点是只要其中一个 case 已经完成，程序就会继续往下执行，而不会考虑其他 case 的情况。

　　基于此特性，我们来为 channel 实现超时机制：

```
// 首先，我们实现并执行一个匿名的超时等待函数
timeout := make(chan bool, 1)

go func() {
	time.Sleep(1e9) // 等待1秒钟
	timeout <- true
}()

// 然后我们把timeout这个channel利用起来
select {
	case <- ch:
	// 从ch中读取到数据
	case <- timeout:
	// 一直没有从ch中读取到数据，但从timeout中读取到了数据
}
```

　　这样使用 select 机制可以避免永久等待的问题，因为程序会在 timeout 中获取到一个数据后继续执行，无论对 ch 的读取是否还处于等待状态，从而达成1秒超时的效果。

　　这种写法看起来是一个小技巧，但却是在 Go 语言开发中避免 channel 通信超时的最有效方法。在实际的开发过程中，这种写法也需要被合理利用起来，从而有效地提高代码质量。

# channel的传递

　　需要注意的是，在 Go 语言中 channel 本身也是一个原生类型，与 map 之类的类型地位一样，因此 channel 本身在定义后也可以通过 channel 来传递。

　　我们可以使用这个特性来实现 *nix 上非常常见的管道（pipe）特性。管道也是使用非常广泛的一种设计模式，比如在处理数据时，可以采用管道设计，这样可以比较容易以插件的方式增加数据的处理流程。

　　下面利用 channel 可被传递的特性来实现我们的管道。为了简化表达，假设在管道中传递的数据只是一个整型数，在实际的应用场景中这通常会是一个数据块。

　　首先限定基本的数据结构：

```
type PipeData struct {
	value int
	handler func(int) int
	next chan int
}
```

　　然后写一个常规的处理函数。只要定义一系列 PipeData 的数据结构并一起传递给这个函数，就可以达到流式处理数据的目的：

```
func handle(queue chan *PipeData) {
	for data := range queue {
		data.next <- data.handler(data.value)
	}
}
```

　　这里只有大概的样子。同理，利用 channel 的这个可传递特性，可以实现非常强大、灵活的系统架构。相比之下，在 C++、Java、C# 中，要达成这样的效果，通常就意味着要设计一系列接口。

　　与 Go 语言接口的非侵入式类似，channel 的这些特性也可以大大降低开发者的心智成本，用一些比较简单却实用的方式来达成在其他语言中需要使用众多技巧才能达成的效果。

# 单向channel

　　顾名思义，单向 channel 只能用于发送或者接收数据。channel 本身必然是同时支持读写的，否则根本没法用。假如一个 channel 真的只能读，那么肯定只会是空的，因为你没机会往里面写数据。同理，如果一个 channel 只允许写，即使写进去了，也没有丝毫意义，因为没有机会读取里面的数据。所谓的单向 channel 概念，其实只是对 channel 的一种使用限制。

　　我们在将一个 channel 变量传递到一个函数时，可以通过将其指定为单向 channel 变量，从而限制该函数中可以对此 channel 的操作，比如只能往这个 channel 写，或者只能从这个 channel 读。

　　单向 channel 变量的声明非常简单，如下：

```
var ch1 chan int // ch1是一个正常的channel，不是单向的
var ch2 chan<- float64// ch2是单向channel，只用于写float64数据
var ch3 <-chan int // ch3是单向channel，只用于读取int数据
```

　　那么单向 channel 如何初始化呢？之前已经提到过，channel 是一个原生类型，因此不仅支持被传递，还支持类型转换。只有在了解了单向 channel 的概念后，才会明白类型转换对于 channel 的意义：就是在单向 channel 和双向 channel 之间进行转换。示例如下：

```
ch4 := make(chan int)
ch5 := <-chan int(ch4) // ch5就是一个单向的读取channel
ch6 := chan<- int(ch4) // ch6 是一个单向的写入channel
```

　　基于 ch4，通过类型转换初始化了两个单向 channel：单向读的 ch5 和单向写的 ch6。为什么要做这样的限制呢？从设计的角度考虑，所有的代码应该都遵循“最小权限原则”，从而避免没必要地使用泛滥问题，进而导致程序失控。写过 C++ 程序的人肯定就会联想起 const 指针的用法。非 const 指针具备 const 指针的所有功能，将一个指针设定为 const 就是明确告诉函数实现者不要试图对该指针进行修改。单向 channel 也是起到这样的一种契约作用。

　　下面来看一下单向 channel 的用法：

```
func Parse(ch <-chan int) {
	for value := range ch {
		fmt.Println("Parsing value", value)
	}
}
```

　　除非这个函数的实现者无耻地使用了类型转换，否则这个函数就不会因为各种原因而对 ch 进行写，避免在 ch 中出现非期望的数据，从而很好地实践最小权限原则。

# 关闭channel

　　关闭 channel 非常简单，直接使用 Go 语言内置的 close() 函数即可：

```
close(ch)
```

　　在介绍了如何关闭 channel 之后，就多了一个问题：如何判断一个 channel 是否已经被关闭？可以在读取的时候使用多重返回值的方式：

```
x, ok := <-ch
```

　　这个用法与 map 中的按键获取 value 的过程比较类似，只需要看第二个 bool 返回值即可，如果返回值是 false 则表示 ch 已经被关闭。

> 　　参考：《Go语言编程》
