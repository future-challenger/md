#Kotlin, Android的Swift

苹果已经用Swift代替Objective-C，一种古老的语言，来进行iOS的开发了。明显Android开发也有这个趋势。

虽然现在已经可以选择Scala或者Groovy等基于JVM的语言开发Android应用来尝尝鲜，但是弊端却显而易见。
要引入一个全新的开发语言，那么就意味着需要引入这个语言的全部的运行时。这简直就是噩梦。因为这会给
极大的增加应用包的大小，还不说65535方法问题。小的应用还可以，但是这不是一个合适的替代语言应该有的问题。

##Kotlin
介绍一下Kotlin--一个基于JVM的语言。由JetBrains（他们开发了IntelliJ IDEA的一个系列和Android Studio）
开发，并由圣彼得堡附近的一个小岛命名，负责这个项目的开发团队就在这个小岛办公。在2011年为外界
所致，并在几年后的第二个里程碑（M2）发布了对Android的支持（翻译的时候已经发布了1.0正式版）。

##特性
这个新的语言有什么特色呢，你可能会问。有很多！不过本文会集中讲述Java应该有而一直没有的上面。

##命名和可选参数
命名参数是一个很简单的语言特性，但是却让代码更加易读，尤其是那些有很多参数的函数签名。

看下面的例子：
```java
void circle(int x, int y, int rad, int stroke) {
    ...
}
```
在java调用的时候看起来是这个样子的：
```java
circle(15, 40, 20, 1);
```
要多次查看函数签名才能知道每个参数的具体作用。在Kotlin里，定义是这样的：
```kotlin
fun circle(x: Int, y: Int, rad: Int, stroke: Int) {
    ...
}
```
调用就更好了：
```kotlin
circle(15, 40, rad = 20, stroke = 1)
```
假设我们现在需要把`stroke`参数设定为可选的，在Java里你只能overload出另一个方法，这个方法
少一个参数，然后在里面调用第一个方法：
```java
public void circle(int x, int y, int rad, int stroke) {
    ...
}

public void circle(int x, int y, int rad) {
    circle(x, y, rad, 1);
}
```
但是在Kotlin里，一切变得简单：
```kotlin
fun circle(x: Int, y: Int, rad: Int, stroke: Int = 1) {
    ...
}
```
不是什么大的功能改进，但是Java始终没有。

##Lambda表达式
最近函数式编程日渐流行。Kotlin也支持lambda表达式。

我们先来看一个简单的。假设你有一个整数列，而且要删除全部的奇数。
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

println(numbers.filter { it % 2 != 0 })
// 输出：[1, 3, 5]
```
这个函数把list的元素类型为输入参数类型，这里是integer。并输出一个boolean型的结果。如果我们把
filter的lambda表达式转化为一个明显的变量写出来的就更容易理解了：
```kotlin
val predicate: (Int) -> Boolean = { it -> it % 2 != 0 }
numbers.filter(predicate)
```
这一语法和其他的而支持lambda表达式的语言的语法相差不大，所以不多做解释。

##Null和类型安全
总是要检查变量是否为null也是Java里一个很烦人的事。这个问题在Kotlin里被部分解决。

如果你用传统的方法定义一个变量，那么编译器是不允许其值为null的。
```kotlin
var text1: String = "something" // 木有问题
var text2: String = null // 无法编译
```
那么定义的变量的值需要为null怎么办呢？你要专门定义一个值可为null的变量。值可为null的变量，用变量
类型后面的问号来明确指定。
```kotlin
var text2: String? = null // 木有问题
```
调用的时候也需要问号。
```kotlin
text2?.replace(" ", "-")
```
这句的意思是：如果`text2`为空，那么`replace()`方法会被忽略并且不会有`NullPointerException`异常抛出。

如果你接手了一个可选类型（optional type）的变量，你可以百分之百的确认变量的值不会为null（这在Adnroid的API里
非常多见，虽然方法定义返回值为空可空类型，但是从来不会返回null），然后可以强制作为非空调用方法。
```kotlin
text2!!.replace(" ", "-")
```
这个时候，如果`text2`为null的话就会抛出`NulPointerException`异常。所以小心使用。

类型安全的另一个意义是类型检测。假设你又一个`Context`的实例并且需要检测这个实例是否为`Activity`类型，然后调用
一些activity才有的方法。
```kotlin
val context = getContext()
if (context is Activity) {
    context.finish()
}
```
类型检测之后你就可以把context作为一个activity类型的实例来使用了，不需要强制类型转换。和Java的
`instanceOf`方法一样，`is`也是null安全的。即使`getContext()`返回一个null，上面的代码也
不会崩溃。

##数据对象（Data object）
在写数据对象的时候，很多你需要手动实现的，比如：`toString()`, 'hashCode()`以及`equals()`。即使
现在很多的IDE已经减轻了一部分工作量，但是添加了新的成员后再更新这些实现也是非常的麻烦。

Kotlin里不用再为这些操心了。你需要做的只是在类定义的前面加一个`data`的修饰，上面的方法就已经隐式的生成了。
```kotlin
data class Island(val name: String? = null)
```
如果你初始化上面的类，会自动生成一个人类可以读懂的`toString()`方法。
```
val island = Island("Kotlin")
println(island.toString())  // 输出：Island(name=Kotlin)
```
同理，如果我们用同一个名字创建另外一个实例，这个实例会`equal`之前创建的实例。并且他们的`hashCode()`值也一样。
```kotlin
val island = Island("Kotlin")
val island2 = Island("Kotlin")

assert(island.equals(island2))
assert(island.hashCode() == island2.hashCode())
```
如果你要定制以上的方法，那么，当然，你还是要手写的。

##单例
单例是一个非常常用的方法。Kotlin省去了创建单例的时候需要的静态`getInstance()`方法和私有的构造函数。
Kotlin使用`object`声明。
```kotlin
object ApiConfig {
    val baseUrl: String = "https://github.com"
}
```
`object`声明也可以用来创建静态方法。
```kotlin
open class MyFragment : Fragment() {
    companion object {
        fun newInstance(): MyFragment = MyFragment()
    }
}
```
上面的方法可以这样调用`val fragment = MyFragment.newInstance()`，就和Java的静态方法一样。

##接口
虽然Kotlin不支持多继承， 但是还是支持接口的。只不过这个接口和`Trait`比较接近，可以在接口中包括默认实现。
```kotlin
interface SessionCloseable {
    fun closeSession() {
        Log.d(SessionCloseable::class.java.simpleName, "Closing...")
    }
}
```
以上接口定义了一个可以关闭session的方法，下面我们在一个activity里实现这个方法。
```kotlin
class KotlinActivity : Activity(), SessionCloseable {
    override fun onStop() {
        super.onStop()
        closeSession()
    }
}
```
有一点需要注意的是Kotlin里没有extend和implement的区分，什么时候都是逗号分隔。

##扩展（extension）
最后我们看一个例子：你可以给一个已经存在的类添加方法。

比如你要创建一个方法，这个方法可以把字符串的空白替换为下划线。在Java里你需要创建一个Utility的方法，以
原始字符串为参数。
```java
public class StringUtils {
    public static String encodeString(String str) {
        return str.replaceAll(" ", "_");
    }
}
```
在Kotlin里，你可以创建一个扩展方法（extension function），即使原类是`final`的。
```kotlin
fun String.encodeSpaces(): String {
    return this.replace(" ", "_")
}
```

##最后
为了公平起见，以上说到的大部分特性在Java8里都已经有了。但是这些什么时候可以在Android里用到可能
就遥遥无期了（其实这个问题已经有人在2014的Google I/O里问过了，但是根据Xavier Ducrohet的说法近期没有这样的计划）。
还好现在有Kotlin了。

另外，这个语言已经非常的成熟。唯一的问题就是文档经常补全而且还经常是过期的，不过因为语法和同时JVM语言的Scala多少
有些接近，多少可以猜出个大概。所以，这只是一个很小的瑕疵。

最后的最后，Kotlin将会在我（作者）后续的Android开发中尽可能的使用。显然，对已有的代码做转化不太明智。
而且Kotlin可以和Java的代码无缝互操作，这样不是什么太大的问题。Kotlin很值得一试！

原文[地址](https://blog.gouline.net/2014/08/31/kotlin-the-swift-of-android/),代码在[这里](https://github.com/future-challenger/okhttp-retrofit-rxandroid)












