---
title: "现代服务端技术栈简介 - Golang，Protobuf 和 gRPC"
date: 2019-04-22T11:09:48+08:00
draft: false
tags: ["microservice"]
categories: ["Microservice"]
---

> 原文链接：[Introduction to the Modern Server-side Stack — Golang, Protobuf, and gRPC](https://medium.com/velotio-perspectives/introduction-to-the-modern-server-side-stack-golang-protobuf-and-grpc-40407486568)

---

城里有一些新的服务器编程玩家，而这一次都是关于谷歌的。自谷歌开始将其用于自己的生产系统以来，Golang 迅速普及。自微服务架构问世以来，人们一直专注于现代数据通信解决方案，如 gRPC 和 Protobuf。在这篇文章中，我将对他们进行简要的介绍。

## 1. Golang

Golang 或 Go 是 Google 开源的通用编程语言。由于各种原因，它最近越来越受欢迎。它可能会给大多数人带来一个惊喜，根据谷歌的消息，Go 语言已诞生将近10年，并已投入生产使用将近7年。

Golang 设计简单，现代，易于理解，并且易于掌握。语言的创造者以这样的方式设计这门语言：一个一般的程序员学习一个周末的时间就可以拥有上手开发的知识。我可以证明他们肯定成功了。说到创作者，这些都是已经参与了 C 语言原草案的专家，所以我们可以很放心，这些家伙知道自己在做什么。

#### 1.1 这一切都很好，但为什么我们需要另一种语言？

对于大多数用例，我们实际上没有。实际上，Go 并没有解决之前其他语言/工具尚未解决的任何新问题。但它确实试图以高效，优雅和直观的方式解决人们通常面临的一系列相关问题。Go 的主要关注点如下：

* 对并发的一等支持
* 优雅，现代的语言，其核心是简单
* 高效的性能
* 对现代软件开发所需工具的第一手支持

我将简要解释Go如何提供以上所有内容。您可以从[Go 的官方](https://golang.org/)网站上详细了解该语言及其功能。

#### 1.2 并发

并发性是大多数服务器应用程序中的主要问题之一，考虑到现代微处理器，它应该是该语言的主要关注点。Go 引入了一个名为 'goroutine' 的概念。'goroutine' 类似于'轻量级用户空间线程'。它事实上要复杂得多，因为多个 goroutine 在一个线程上复用，上面的表述应该能给你一个大致的想法。它们足够轻，实际上可以同时旋转一百万个 goroutines，因为它们以非常小的堆开始。事实上，这是推荐的。Go 中的任何函数/方法都可用于生成 Goroutine。你只需要 'go myAsyncTask()' 就可以从 'myAsyncTask' 函数​​中生成一个 goroutine。以下是一个例子：

```
// 此函数通过为每个任务生成 goroutine 来同时执行给定的任务

func performAsyncTasks(task []Task) {
  for _, task := range tasks {
    // 这将生成一个单独的 goroutine 来执行此任务。
    // 此调用是非阻塞的
    go task.Execute()
  }
}
```

是的，它很简单，它就是这样的，因为 Go 是一种简单的语言，你应该为每一个独立的异步任务产生一个 goroutine 而不需要太多关注。如果有多个核心可用，Go 的运行时会自动并行运行 goroutine。但这些 goroutines 如何交流呢？答案是 channels。

'Channel' 也是一种语言原语，用于 goroutines 之间的通信。您可以将任何内容通过 channel 传递到另一个 goroutine（原始 Go 类型或 Go struct 甚至其他 channel）。通道本质上是一个阻塞的双端队列（也可以是单端的）。如果您希望 goroutine 等待某个条件得到满足，然后继续运行，您可以在 channel 的帮助下实现 goroutine 的协同阻塞。

这两个原语在编写异步或并行代码时提供了很多灵活性和简单性。可以从上面的原语轻松创建其他辅助库，如 goroutine 池。一个基本的例子是：

```
package executor

import (
	"log"
	"sync/atomic"
)

// Executor 结构是任务的主要执行者。
// 'maxWorkers' 表示并发 goroutine 的最大数量。
// 'ActiveWorkers' 告诉执行者在给定时间产生的活动 goroutines 的数量。
// 'Tasks' 是 Executor 接收任务的 channel。
// 'Reports' 是 Executor 发布每个任务报告的 channel。
// 'signals' 是可用于控制执行程序的 channel。现在，只有终止
// signal 是受支持的，主要是客户端在此 channel 上发送'1'。
type Executor struct {
	maxWorkers    int64
	ActiveWorkers int64

	Tasks   chan Task
	Reports chan Report
	signals chan int
}

// NewExecutor 创建一个新的 Executor。
// 'maxWorkers' 描述并发 goroutines 的最大数量。
// 'signals' channel 可用于控制 Executor。
func NewExecutor(maxWorkers int, signals chan int) *Executor {
	chanSize := 1000

	if maxWorkers > chanSize {
		chanSize = maxWorkers
	}

	executor := Executor{
		maxWorkers: int64(maxWorkers),
		Tasks:      make(chan Task, chanSize),
		Reports:    make(chan Report, chanSize),
		signals:    signals,
	}

	go executor.launch()

	return &executor
}

// launch 启动主循环以轮询所有相关 channels 并处理不同的消息
func (executor *Executor) launch() int {
	reports := make(chan Report, executor.maxWorkers)

	for {
		select {
		case signal := <-executor.signals:
			if executor.handleSignals(signal) == 0 {
				return 0
			}

		case r := <-reports:
			executor.addReport(r)

		default:
			if executor.ActiveWorkers < executor.maxWorkers && len(executor.Tasks) > 0 {
				task := <-executor.Tasks
				atomic.AddInt64(&executor.ActiveWorkers, 1)
				go executor.launchWorker(task, reports)
			}
		}
	}
}

// handleSignals 都会被调用，无论在 "signals" channel 上收到任何内容
// 它根据收到的信号（请求）执行相关任务，然后用0或1来响应，并志明请求是被接受（0）还是被拒绝（1）。
func (executor *Executor) handleSignals(signal int) int {
	if signal == 1 {
		log.Println("Received termination request...")

		if executor.Inactive() {
			log.Println("No active workers, exiting...")
			executor.signals <- 0
			return 0
		}

		executor.signals <- 1
		log.Println("Some tasks are still active...")
	}

	return 1
}

// 只要收到一个新任务，就会调用 launchWorker，Executor 可以生成更多的 workers 来生成一个新的 worker
// 每个 worker 都在新的 goroutine 上启动。它执行给定的任务并在 Executor 的内部报告 channel 上发布报告
func (executor *Executor) launchWorker(task Task, reports chan<- Report) {
	report := task.Execute()

	if len(reports) < cap(reports) {
		reports <- report
	} else {
		log.Println("Executor's report channel is full...")
	}

	atomic.AddInt64(&executor.ActiveWorkers, -1)
}

// AddTask 用于向 Executor 提交新任务，它是一种非阻塞方式。客户可以使用 Executor 的任务 channel 直接提交一个新任务，但是如果任务 channel 已满的情况下会阻塞
// 如果任务 channel 已满，则应该认为此方法不会添加给定任务，并由客户端稍后再试。
func (executor *Executor) AddTask(task Task) bool {
	if len(executor.Tasks) == cap(executor.Tasks) {
		return false
	}

	executor.Tasks <- task
	return true
}

// Executor 使用 addReport 以非阻塞方式发布报告。客户端没有读取报告 channel 或者读取速度比 Executor 发布报告的速度慢，报告 channel 将被填满。在这种情况下，该方法不会阻塞，该报告也不会被添加。
func (executor *Executor) addReport(report Report) bool {
	if len(executor.Reports) == cap(executor.Reports) {
		return false
	}

	executor.Reports <- report
	return true
}

// Inactive检查 Executor 是否空闲。当没有待处理任务，活动 workers 和要发布的报告时，会发生这种情况。
func (executor *Executor) Inactive() bool {
	return executor.ActiveWorkers == 0 && len(executor.Tasks) == 0 && len(executor.Reports) == 0
}
```

#### 1.3 简单的语言

与许多其他现代语言不同，Golang 没有很多功能。实际上，对于语言在其功能集中的限制性过高而言，这是一个令人信服的案例。它不是围绕像 Java 这样的编程范例设计的，或者是为了支持 Python 等多种编程范例而设计的。它只是简单的结构编程。只是基本功能被抛入语言而不是一件事。

在查看语言之后，您可能会觉得该语言没有遵循任何特定的理念或方向，并且感觉每个功能都包含在此处以解决特定问题，仅此而已。例如，它有方法和接口，但没有类; 编译器生成一个静态链接的二进制文件，但仍然有一个垃圾收集器; 它具有严格的静态类型，但不支持泛型。该语言确实具有瘦运行时但不支持异常。

这里的主要想法是开发人员应该花费最少的时间来表达他/她的想法或算法作为代码，而不考虑“在x语言中这样做的最佳方式是什么？”并且应该很容易理解其他人。它仍然不完美，它确实不时地受到限制，并且正在考虑将“Go 2”等一些基本功能（如泛型和异常）考虑在内。

#### 1.4性能

单线程执行性能不是判断语言的好指标，尤其是当语言集中在并发性和并行性时。但是，Golang 令人印象深刻的基准数据仅受到C，C ++，Rust等核心系统编程语言的打击，并且仍在不断改进。考虑到它是带垃圾收集的语言，性能实际上非常令人印象深刻，并且对于几乎每个用例都足够好。

![]()

#### 1.5 开发人员工具

采用新工具/语言直接取决于其开发人员的经验。Go 的采用确实代表了它的工具。在这里，我们可以看到相同的想法和工具是非常小但足够的。这都是通过'go'命令及其子命令实现的。所有的工具都是命令行。

没有像 pip，npm 这样的语言的包管理器。但你可以通过这样做得到任何社区包

```
go get github.com/farkaskid/WebCrawler/blob/master/executor/executor.go
```

是的，它有效。您可以直接从 GitHub 或其他任何地方拉取 package。它们只是源文件。

但是 package.json 呢？我没有看到任何“go get”的等价物。因为没有。您无需在单个文件中指定所有依赖项。你可以直接使用：

```
import "github.com/xlab/pocketsphinx-go/sphinx"
```

在您的源文件里面，并且当您执行 "go build" 时，它将自动为您 "go get"。您可以在此处查看完整的源文件：

```
package main

import (
	"encoding/binary"
	"bytes"
	"log"
	"os/exec"

	"github.com/xlab/pocketsphinx-go/sphinx"
	pulse "github.com/mesilliac/pulse-simple" // pulse-simple
)

var buffSize int

func readInt16(buf []byte) (val int16) {
	binary.Read(bytes.NewBuffer(buf), binary.LittleEndian, &val)
	return
}

func createStream() *pulse.Stream {
	ss := pulse.SampleSpec{pulse.SAMPLE_S16LE, 16000, 1}
	buffSize = int(ss.UsecToBytes(1 * 1000000))
	stream, err := pulse.Capture("pulse-simple test", "capture test", &ss)
	if err != nil {
		log.Panicln(err)
	}
	return stream
}

func listen(decoder *sphinx.Decoder) {
	stream := createStream()
	defer stream.Free()
	defer decoder.Destroy()
	buf := make([]byte, buffSize)
	var bits []int16

	log.Println("Listening...")

	for {
		_, err := stream.Read(buf)
		if err != nil {
			log.Panicln(err)
		}

		for i := 0; i < buffSize; i += 2 {
			bits = append(bits, readInt16(buf[i:i+2]))
		}

		process(decoder, bits)
		bits = nil
	}
}

func process(dec *sphinx.Decoder, bits []int16) {
	if !dec.StartUtt() {
		panic("Decoder failed to start Utt")
	}
	
	dec.ProcessRaw(bits, false, false)
	dec.EndUtt()
	hyp, score := dec.Hypothesis()
	
	if score > -2500 {
		log.Println("Predicted:", hyp, score)
		handleAction(hyp)
	}
}

func executeCommand(commands ...string) {
	cmd := exec.Command(commands[0], commands[1:]...)
	cmd.Run()
}

func handleAction(hyp string) {
	switch hyp {
		case "SLEEP":
		executeCommand("loginctl", "lock-session")
		
		case "WAKE UP":
		executeCommand("loginctl", "unlock-session")

		case "POWEROFF":
		executeCommand("poweroff")
	}
}

func main() {
	cfg := sphinx.NewConfig(
		sphinx.HMMDirOption("/usr/local/share/pocketsphinx/model/en-us/en-us"),
		sphinx.DictFileOption("6129.dic"),
		sphinx.LMFileOption("6129.lm"),
		sphinx.LogFileOption("commander.log"),
	)
	
	dec, err := sphinx.NewDecoder(cfg)
	if err != nil {
		panic(err)
	}

	listen(dec)
}
```

这将依赖声明与源代码本身绑定在一起。

正如你现在所看到的那样，它简单，简约而又充足而优雅。对火焰图表的单元测试和基准测试也有第一手的支持。就像功能集一样，它也有其缺点。例如，'go get'不支持版本，您将被锁定到源文件中传递的导入URL。它正在发展，其他工具已经出现在依赖管理中。

Golang 最初旨在解决 Google 在其庞大的代码库中遇到的问题以及编写高效并发应用程序的迫切需求。它使编码应用程序/库更容易利用现代微芯片的多核特性。它是一种简单的现代语言，它从不试图成为更多的东西。

## 2. Protobuf (Protocol Buffers)

Protobuf 或 Protocol Buffers 是 Google 的二进制通信格式。它用于序列化结构化数据。它是一种通讯格式？有点像JSON？是。它已经超过10年了，谷歌已经使用了一段时间了。

但是我们不是拥有JSON而且它无处不在......

就像 Golang 一样，Protobufs 并没有真正解决任何新问题。它只是以现代的方式更有效地解决现有问题。与 Golang 不同，它们不一定比现有解决方案更优雅。以下是 Protobuf 的重点：

* 它是一种二进制格式，与 JSON 和 XML 不同，它是基于文本的，因此它具有极大的空间效率。
* 对模式的第一手和复杂支持。
* 第一手支持以各种语言生成解析和消费者代码。
* 二进制格式和速度

Protobuf真的那么快吗？简短的回答是，是的。根据 Google Developers 的说法，它们比 XML 小3到10倍，比 XML 快20到100倍。这并不奇怪，因为它是二进制格式，序列化数据不是人类可读的。

![]()

Protobufs 采取更有计划的方法。您定义 '.proto' 文件，这些文件是一种模式文件，但功能更强大。您基本上定义了如何构建消息，哪些字段是可选的或必需的，它们的数据类型等。之后，Protobuf 编译器将为您生成数据访问类。您可以在业务逻辑中使用这些类来促进通信。

查看与服务相关的 `.proto` 文件还可以让您清楚地了解通信的细节和暴露的功能。典型的 `.proto` 文件如下所示：

```
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}
```

有趣的事实：Stack Overflow 之王 Jon Skeet 是该项目的主要贡献者之一。

## 3. gRPC

正如您所猜测的那样，gRPC是一个现代RPC（远程过程调用）框架。它是一个包含框架，内置支持负载平衡，跟踪，运行状况检查和身份验证的电池包。它于2015年由谷歌开源，从那时起它就越来越受欢迎。

#### 3.1 一个RPC框架...？REST呢？

使用 WSDL 的 SOAP 已经被长时间用于面向服务的体系结构中的不同系统之间的通信。当时，使用的合约都是严格定义的，而且系统庞大而单一，暴露出大量此类接口。

然后出现了“浏览”的概念，其中服务器和客户端不需要紧密耦合。即使客户是独立编码的，客户也应该能够浏览服务产品。如果客户要求提供有关书籍的信息，则该服务以及所请求的内容也可以提供相关书籍的列表，以便客户可以浏览。REST 范式对此至关重要，因为它允许服务器和客户端在没有使用某些原始动词的严格限制的情况下自由通信。

正如您在上面所看到的，该服务的行为类似于单片系统，它与所需的一样，还可以执行许多其他操作，以便为客户提供预期的“浏览”体验。但这并不是常见的使用场景。是吗？

#### 3.2 进入微服务

采用微服务架构的原因有很多。突出的一点是，整体系统的扩展非常困难。在设计具有微服务架构的大型系统时，每个业务或技术要求旨在作为几个原始“微型”服务的合作组合来执行。

这些服务在响应中不需要全面。他们应该按照预期的响应履行具体职责。理想情况下，它们应该像纯函数一样表现无缝可组合性。

现在使用 REST 作为此类服务的通信范例并没有为我们带来很多好处。但是，为服务公开 REST API 确实为该服务提供了大量的表达能力，但如果既不需要也不打算使用这种表达能力，我们可以使用更多关注其他因素的范例。

gRPC 打算在传统 HTTP 请求方面改进以下技术方面：

* 默认情况下 HTTP/2 及利用其所有优点。
* Protobuf 作为用于服务器交流。
* 借助 HTTP/2，专门支持流媒体呼叫。
* 可插拔的身份验证，跟踪，负载均衡和健康检查，因为您总是需要这些。

由于它是一个 RPC 框架，我们再次拥有服务定义和接口描述语言等概念，这些概念可能会让那些在 REST 之前不存在的人感到陌生，但这次感觉不那么笨拙，因为 gRPC 使用 Protobuf 来实现这两者。

Protobuf 被设计为既可以用作通信格式，也可以用作协议规范工具，而无需引入任何新内容。典型的gRPC 服务定义如下所示：

```
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

您只需为您的服务编写一个 `'.proto'` 文件，用它描述接口名称，它所期望的内容以及它作为 Protobuf 消息返回的内容。然后，Protobuf 编译器将生成客户端和服务器端代码。客户端可以直接调用它，服务器端可以实现这些 API 来填充业务逻辑。

## 4. 结论

Golang 和使用 Protobuf 的 gRPC 是现代服务器编程的新兴技术栈。Golang 简化了并发/并行应用程序，使用 Protobuf 实现 gRPC，可实现高效的通信和令人愉悦的开发人员体验。

*******************************************************************

这篇文章最初发表在 [Velotio Blog上](https://velotio.com/blog/2019/3/25/server-side-stack-golang-protobuf-grpc)。

[Velotio Technologies](https://velotio.com/) 是技术创业公司和企业的外包软件产品开发合作伙伴。我们专注于企业 B2B 和 SaaS 产品开发，专注于人工智能和机器学习，DevOps 和测试工程。
