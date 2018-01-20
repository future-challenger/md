#为什么要用GCD-Swift

当今世界，多核已然普及。但是APP却不见得很好的跟上了这个趋势。APP
想要利用好多核就必须可以保证任务能有效的分配。并行执行可以让APP同时执行很多
的任务。这个其实很难，但是有了GCD一切都变得简单了很多。

你并不是一定要写一个大并发的APP才需要用GCD。使用GCD可以让你的APP更快的
响应用户的操作，不用要等到你的UI或者服务等到执行完成。一般来说你会把各种任务
都分配给其他核心去执行，而你的主线程（UI线程）可以随时处理用户的操作。
GCD可以让这些变得简单。

###线程
传统的多任务分发方式是使用线程。在一个多核的设备上，每一个新的线程都可以被分配在一个CPU核心
上，与其他的线程并行执行。
```    
单核心的CPU麻烦一些，系统会不停地在几个线程之间切换以保证每个线程都有机会执行。这样做的效果
是看起来像是并行执行的，但是其实多个线程的不同任务之间是顺序执行的。
```
    
但是使用线程也会遇到一个很大的问题。数据在线程之间的正确传递就是一个很大的难题了。线程之间的
同步和互斥又会变得很诡异难以调试。而且，为了保证APP的高效快速运行，开辟多少线程也是一个需要思考的
问题。因为，创建和销毁线程也是有很大的资源消耗的。于是很多的系统都提供了一个叫做**线程池**
的概念来解决这个问题。所有的线程创建后都放在这个池子里统一管理。

###同步
当你有多个线程在执行的时候，你一般都会遇到一个问题**Race Condition**，实际的运算结果
取决于那个线程先获得共享数据。一个经典的例子就是：银行账户问题。

```swift
class BankAccount {
    var balance: Double?
    
    //...
}

// 创建一个账号，给这个账号存100块
var account = BankAccount()
account.balance = 100

// 第一个线程：取10块
func withdraw() {
    let balance = account.balance
    account.balance = balance! - 10
}

// 第二个线程：增加10%的利息
func accrue() {
    let balance = account.balance
    account.balance = balance! * 1.10
}

// 那么最后：account.balance = ?
```
最后的结果取决于这两个线程哪一个先执行。在并行的条件下执行的先后顺序是不定的。但是执行顺序不同
最后的balance值就是不同的。
```
上面的代码会有多少个可能的不同结果？balance都会是什么值？
```
```
Race Condition即使在单核设备下也会发生。
```
对于这些问题，传统的解决方法如下：
* 信号量（semaphore）- 用来控制一组有限资源的消费。线程等待，知道资源可用。
* 互斥（Mutex）- 一次只允许一个线程执行。当一个线程持有mutex的时候其他线程等待。
* 条件变量（Condvar）- 线程等待直到某些条件为真。

###队列
GCD把线程的创建、回收以及线程的同步等进一步抽象为队列（Queue）统一管理。

队列，简单而言，就是一个可以让数据按照先进先出的顺序执行。一个APP可以创建多个队列，并且多个
队列之间可以并行的处理各自的任务，每个队列内部的任务顺序执行。

队列比线程有很多的优势。第一，GCD库屏蔽了线程管理的繁琐部分。队列会在需要的时候自动创建线程
并且在不需要的时候回收。其次，GCD库会根据系统的CPU核心数创建最佳数量的线程。最后，队列只会
在需要的时候创建线程，所以资源利用会得到优化。

总之，队列给你了你线程能给的，但是又不用考虑具体线程的操作。

GCD有三种队列：
* 顺序队列（Serial）- 每次执行一个任务，按照任务加入队列的顺序。一个任务执行完成后执行下一个。
* 并发（Concurrent）- 按照任务加入队列的顺序开始，但是后面的任务不用等到前面的任务执行完成就可以开始。
* 主线程（Main）- 一个预先开启的序列线程。这个队列中包含一个NSRunLoop实例。你的APP总是在这个队列中运行。

系统提供了几种并发队列。这些队列有自己专属的QoS（服务质量种类）。这个服务质量种类是用来表示你提交的
任务的意图是什么，这样GCD可以有针对性的优化。

* **QOS_CLASS_USER_INTERACTIVE** 这个*用户交互（user interactive）*表示任务需要立即执行，以便APP
给用户一个良好的用户体验。一般用于更新UI、处理事件和小延迟的处理。在你的整个APP中，这以种类的任务应该保持
一个较低的总量。

* **QOS_CLASS_USER_INITIATED** *用户初始化（user initiated）*表示任务是在UI初始化的，并且可以
异步执行。这个一般用于处理用户等待的需要理解给出运行结果，或者任务需要理解完成用户交互的情况。

* **QOS_CLASS_UTILITY** *通用（utility）*表示长期执行的任务，一般来说用户可以见到任务执行的比例。
一般用来处理大的计算、I/O、网络以及不简单的数据提交等类似任务。这一种类做了电量优化。

* **QOS_CLASS_BACKGROUND** *后台运行（background）*表示用户并不知道任务在运行中。这个一般用于
数据的提前加载、维护，以及其他无需用户交互和任务完成时间无明显限制的情况。

这里需要注意一点，苹果的API也会用到这些全局分发的队列。所以，在这些队列里并不是只有你的任务在执行。当然，
如前文所述你也可以创建你自己的队列。你可以选择4种全局队列，主队列，还有两种自定义队列可以选择。

###Closure
队列的任务使用closure封装。
```
你也可以使用方法加入队列，不过这里只使用block。
```
这里有一个closure的例子，让你对closure有一个大概的印象。swift的closure就和Objective-C的block
差不多。
```swift
func makeIncrementer() -> ((Int) -> Int) {
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    
    return addOne
}

var increment = makeIncrementer()

increment(9)
```
上面的例子`makeIncrementer`返回一个closure，这个closure需要一个整型参数。

###Hello dispatch world！
这里需要注意一点：Objective-C的block和Swift的closure。
Swift的closure和Objective-C的block是兼容的。所以你完全可以把Swift的closure传入Objective-C
中需要block参数的方法中。Swift的closure和方法是同一类型的，所以你甚至可以把swift的方法名传递过去。

Closure和block的上下文处理语法基本一样。唯一不同的是变量在closure中直接就是可变的。也就是说Objective-C
中的`__block`关键字在Swift的closure中是默认行为。
```swift
dispatch_async(dispatch_get_global_queue(QOS_CLASS_USER_INTERACTIVE, 0), {
    print("hello dispatch:- user interactive")
})

dispatch_async(dispatch_get_main_queue(), {
    print("hello dispatch:- main queue")
})
```
调用`dispatch_async()`，GCD就会给队列添加一个closure任务，然后继续代码的执行完全不会等待closure
的结束。在我们的例子中，第一次`dispatch_async`是给`QOS_CLASS_USER_INTERACTIVE`的类型的全局
队列添加了一个任务。第二次是给`dispatch_get_main_queue`，也就是主线程添加了一个任务。

下面看一个自创队列的例子：
```swift
@IBAction func concurrentAction(sender: AnyObject) {
    let concurrentQueue = dispatch_queue_create("concurrent.test.queue", DISPATCH_QUEUE_CONCURRENT) //1
    for i in 0...1000 { 
        dispatch_async(concurrentQueue){ //2
            NSThread.sleepForTimeInterval(1) //3 
            print("print concurrent queue:- \(i)") //4
        }
    }
}
```
这里一步一步的介绍一下：
1. 创建一个名称为`concurrent.test.queue`的并发队列。
2. 做一个1001次的循环，每个循环给这个队列添加一个closure。以上的写法只是一个简写，其实是这样的：
```swift
dispatch_async(concurrentQueue,{ //2
        NSThread.sleepForTimeInterval(1) //3
        print("print concurrent queue:- \(i)") 
})
```
3. 当前线程休眠一秒


###Barriers
很多人看到这里就会想，并发队列在哪儿体现出来队列的概念了呢？这分明就是一个把一堆closure扔进去分开执行的*堆*或者*Set（集合）*
而已嘛。

目前来看是的，但是当你遇到`barrier`的时候对列就真的变成队列了。你可以使用`dispatch_barrier_sync`和
`dispatch_barrier_async`两个方法把closure加入队列中。这个时候就很有意思了。扔进去的closure并
不会立刻执行，而是要等。等到在这个closure之前扔进队列的全部closure都执行完成之后才开始执行。然后，
在这个barrier的closure执行完成之后，在它后面扔进队列的closure才会被执行。可以把barrier的closure认为
是一个特殊的点，在这个点的从前到后都是顺序执行的。除此之外的点，还是并行执行。

我们来看看具体的例子：
```swift
let concurrentQueue = dispatch_queue_create("concurrent.test.queue", DISPATCH_QUEUE_CONCURRENT)

var count: Int = 0

for _ in 0...100 {
    dispatch_async(concurrentQueue){
        NSThread.sleepForTimeInterval(1)
        print("print concurrent queue:- \(count++)")
    }
}

dispatch_barrier_async(concurrentQueue, {
    print("##ASYNC in barrier, concurrent queue - START")
    for _ in 1...10 {
        NSThread.sleepForTimeInterval(0.5)
    }
    print("##ASYNC in barrier, concurrent queue - END")
})

for _ in 0...100 {
    dispatch_async(concurrentQueue){
        NSThread.sleepForTimeInterval(1)
        print("print concurrent queue:- \(count++)")
    }
}

dispatch_barrier_sync(concurrentQueue, {
    print("##SYNC in barrier, concurrent queue - START")
    for _ in 1...10 {
        NSThread.sleepForTimeInterval(0.5)
    }
    print("##SYNC in barrier, concurrent queue - END")
})

for _ in 0...100 {
    dispatch_async(concurrentQueue){
        NSThread.sleepForTimeInterval(1)
        print("print concurrent queue:- \(count++)")
    }
}
```

注意`barrier`最好是用在自己创建的并发队列上。否则的话`dispatch_barrier_async`的效果就和`dispatch_async`一样，而
`dispatch_barrier_sync`就和`dispatch_sync`一样。`dispatch_barrier_sync`要分配的队列是当前队列
的话会造成死锁。

###读写锁
先看一个新闻累的单例的例子。这个例子的最初形态中不是一个线程安全的单例：
```swift
class NewsFeed {
    static let sharedInstance = NewsFeed() //1
    
    private init() {} //2
    
    private var _news: [String] = []
    var news: [String] {
        return _news
    }
    
    func addNews(n: String) {
        _news.append(n)
    }
}
```
有这么一个新闻的类。
1. 实现一个单例。Swift的单例明显要比Objective-C简单了很多。是的，Swift的static属性自动内置了
`dispatch_once`。所以，这么一样就实现了Swift的单例。
2. `init`方法只能给类的内部调用，但是不能给外部在调用，否则就不是单例了。

用户可以调用`news`属性来读取全部的新闻，也可以通过调用方法`addNews`来添加新闻。
当时当多个线程都可以访问`addNews`方法的时候，那么`news`属性读出来的新闻列表就有很大的可能是错的。
我们现在使用barrier来确保这个功能可以正确的执行。

解决办法就是任何的线程要添加新闻，那么就必须通过barrier这一道关口。在添加新闻的时候只有一个closure执行。
其他的添加closure等待。

为了保证线程的安全，读取新闻的操作也只能在`concurrentQueue`上执行。但是这次我们是需要立即返回结果的
没法使用`dispatch_async`。这个时候使用`dispatch_sync`就是最好的选择了。使用`dispatch_sync`
可以等到方法执行完毕，并返回我们需要的结果。dispatch sync可以知道dispatch barrier的进度如何，添加了
几条新闻。也可以让closure执行完成并返回。

但是使用dispatch sync需要很小心。如前所述，如果你给dispatch sync分配的队列是当前正在运行的队列
会造成**死锁**。因为调用会等待这个closure结束，这个closure甚至可能都没法开始。因为这个closure需要等到
它前面运行的closure结束之后才能开始。

```swift
private var _news: [String] = []
    var news: [String] {
        var newsCopy: [String]!
        dispatch_sync(concurrentQueue){ //1
            newsCopy = self._news //2
        }
        return newsCopy
    }
```
下面逐一解释：
1. dispatch sync同步分配到`concurrentQueue`上执行读取操作。
2. 保存一份新闻的拷贝给`newsCopy`并返回。

现在这个新闻单例就是线程安全的了。下面我们看一下完整的代码：
```swift
import Foundation

class NewsFeed {
    static let sharedInstance = NewsFeed()
    private let concurrentQueue = dispatch_queue_create("newsfeed.queue.concurrent",
        DISPATCH_QUEUE_CONCURRENT)
    
    private init() {}
    
    private var _news: [String] = []
    var news: [String] {
        var newsCopy: [String]!
        dispatch_sync(concurrentQueue){
            newsCopy = self._news
        }
        return newsCopy
    }
    
    func addNews(n: String) {
        dispatch_barrier_async(concurrentQueue){
            self._news.append(n)
            dispatch_async(dispatch_get_main_queue()){
                self.newsAddedNotification()
            }
        }
    }
    
    func newsAddedNotification() {
        // post notification
    }
}
```
