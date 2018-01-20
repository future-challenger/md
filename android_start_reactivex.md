#初识Android的ReactiveX
开发一个复杂一点的Android应用都会用到网络请求，交互和动画。这些都意味着
要写很多的回调嵌套。这样的代码也被称为*callback hell*（回调地狱）。这样的
代码不仅长，很难理解，而且也是错误高发的地方。[ReactiveX](http://reactivex.io/intro.html)
提供了一个清晰、准确处理异步问题和事件的方法。

RxJava是一个ReactiveX在JVM上的实现，由NetFlix开发。这个库在Java开发者中
广为流传。这个教程中你会学到如何在Android应用开发中使用RxJava。这里Android中的RxJava
可以简称为RxAndroid。

##1. 配置RxAndroid
要在android中使用RxAndroid，需要在**build.gradle**里添加compile依赖项。
```groovy
compile 'io.reactivex:rxjava:1.1.1'
```
##2. Oberser和Observable的基础
在使用RxJava的时候，你会经常遇到`Observer`和`Observable`的概念。
你可以把`Observable`理解为数据的生产者，而`Observer`则可以理解为数据的消费者。在RxJava里
数据的生产者都是`Observable`类的实例，消费者都是接口`Observer`的实例。

类`Observable`有很多静态方法，称为**operator**。来创建类`Observable`的实例。以下
代码演示了如何使用方法`just`方法（这就是上文说到的**operator**）来穿件一个非常简单
的实例，并提供一个`String`数据：
```java
Observable<String> theObservable = Observable.just("hello world!");
```
我们刚刚创建的实例会在用有至少一个观察者的时候发出数据。要创建一个观察者，就需要创建
一个实现了接口`Observer`的类。接口`Observer`里的方法可以处理从`Observable`实例发出的通知。
下面的观察者可以把`Observable`实例发出的数据打印出来。
```java
private final static String TAG = RxJavaActivity.class.getSimpleName();

Observer<String> theObserver = new Observer<String>() {
    @Override
    public void onCompleted() {
        Log.d(TAG, "completed");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "error");
    }

    @Override
    public void onNext(String s) {
        Log.d(TAG, "data:- " + s);
    }
};
```
1. 在Observable实例没有数据在发出的时候调用。
2. 在Observable实例遇到错误的时候调用。
3. 在Observable实例每次发出数据的时候调用。

要给观察者指定一个observable的实例，我们需要调用`subscribe`方法。这个方法会返回一个`Subscription`
实例。下面的代码让`theObserver`观察者观察`theObservable`。
```java
Subscription theSubscription = theObservable.subscribe(theObserver);
```
当观察者添加到了observable实例中，observable实例就发出数据。因此，如果运行上面的代码你会
看到**hello world!**被打印出来。具体如下：
```java
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- hello world!
okhttp.demo.com.okhttpdemo D/RxJavaActivity: completed
```
一般来说方法`onCompleted`和`onError`不会用到，不过某些特殊情况也会用到，不过这里不多叙述。
所以，有更加简洁的可以用到`Action1`接口，这个接口只有一个方法。
```java
Action1<String> theAction = new Action1<String>() {
    @Override
    public void call(String s) {
        Log.d(TAG, "action1 data:- " + s);
    }
};

Subscription theSubscription = theObservable.subscribe(theAction);
```
这样提交了一个接口`action1`的实例之后，`Observable`的实例就发出数据。

要从observable中去除一个观察者只需要在`Subscription`实例上调用方法`unsubscribe`。
```java
theSubscription.unsubscribe();
```

##3. 使用Operator
现在你已经知道如何创建观察者和observable（可观察对象）。下面来看看如何使用RxJava的operator（操作符）
来创建、转换observable的，当然还有其他的一些操作。现在创建一个复杂一点的`Observable`对象。
这个对象发出一个`Integer`数组数据。方法`from`可以这个功能。
```java
Observable<Integer> listObservable = Observable.from(new Integer[]{1, 2, 3, 4, 5});
listObservable.subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        Log.d(TAG, "data:- " + String.valueOf(integer));
    }
});
```
运行这段代码你会发现数组中的数据一个接一个的打印了出来。
```java
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 1
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 2
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 3
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 4
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 5
```
如果你熟悉javascript或者kotlin, 你就会对`map`和`filter`方法如何在数组中使用有一定的认识。
RxJava也有类似的operator来处理observable（可观察对象）。由于java 7没有lambda表达式，我们需要
模拟一下这个lambda表达式。要模拟只接受一个参数的lambda表达式只要创建一个实现了接口`Func1`的实例。

这里你可以用`map`方法来遍历`listObservable`的每一个元素。这个例子会遍历observable每个整数并输出这个数字的平方值。
```java
listObservable.map(new Func1<Integer, Integer>() { // 1
    @Override
    public Integer call(Integer integer) {
        return integer * integer; // 2
    }
});
```
1. 输入和输出的值都是`Integer`类型的。
2. 输出数字的平方值。

这里有一点需要注意的地方。`map`方法返回的是一个新的`Observable`对象，这个方法本身并不改变
原本的`Observable`的实例。如果给`listObsevable`添加了观察者，就会返回这些数字的平方值。

Operator（操作符）可以成链式调用。比如，以下代码使用了`skip`方法跳过前两个数字，并使用`filter`方法
来忽略奇数：
```java
// skip & filter
listObservable
        .skip(2) // 1
        .filter(new Func1<Integer, Boolean>(){
            @Override
            public Boolean call(Integer integer) {
                return integer % 2 == 0; // 2
            }
        });
```
输出：
```java
okhttp.demo.com.okhttpdemo D/RxJavaActivity: skip & filter data:- 4
```
##4. 该处理异步问题了
我们前面创建的observer（观察者）和observable（可观察对象）都是在单个的线程中运行的，都在Android的UI线程中。
这里，我们要用RxJava来处理一些多线程的问题。同时，演示解决callback hell（回调地狱）的问题。

假设我们有一个叫做`fetchData`的方法，这个方法被用来从API上获取数据。这个方法接受一个URL字符串
为参数，并返回一个字符串。这个方法可以这么使用：
```java
String content = fetchData("https://api.github.com/orgs/octokit/repos");
```
如果你执行命令*curl -i https://api.github.com/orgs/octokit/repos*是会返回一个json字符串的。
是的，这个URL是github的API的一个例子。在这里用正合适。

在Android里，网络请求不能在UI线程中，只能另外开辟一个线程或者使用`AsyncTask`。有了RxJava之后
你就多了一个选项。是用`subscribeOn`和`observeOn`操作符（operator），你可以显示的指出后台任务在哪个线程
运行，更新界面的任务在哪个线程（当然这必须是在UI线程）。

不过在继续往下之前，我们需要把RxAndroid的扩展添加到项目依赖里。这个库是RxJava在Android上的扩展。
```groovy
compile 'io.reactivex:rxandroid:1.1.0'
```

下面的代码演示如何使用`create`操作符（operator)创建一个`Observable`对象。使用这个方法创建的
`Observable`对象必须实现`Observable.OnSubscribe`接口，并调用`onNext`, `onError`和`onComplete`
来控制发出什么数据等操作。
```java
Observable<String> asyncObservable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        try {
            String content = fetchData("https://api.github.com/orgs/octokit/repos");
            subscriber.onStart();
            // 1
            subscriber.onNext(content);
            // 2
            subscriber.onCompleted();
        } catch (Exception e) {
            // 3
            subscriber.onError(e);
        }
    }
});
```
1. 把方法`fetchData`获取的数据发出去。
2. 这里没有什么要做的。
3. 如果出现了错误，那么就发出错误信息。

然后就可以给这个`asyncObservable`, `Observable`对象来调用`subscribeOn`和`observeOn`两个方法来指定执行的
线程。
```java
Observable<String> asyncObservable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        try {
            String content = fetchData("https://api.github.com/orgs/octokit/repos");
            subscriber.onStart();
            subscriber.onNext(content);
            subscriber.onCompleted();
        } catch (Exception e) {
            subscriber.onError(e);
        }

    }
});

asyncObservable
        .subscribeOn(Schedulers.newThread())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                Log.d(TAG, "async data:- " + s);
                Toast.makeText(RxJavaActivity.this, "async data:- " + s, Toast.LENGTH_SHORT).show();
            }
        });
```
你也许觉得这个没见的比`AsyncTask`或者`Thread`+`Handler`的组合好。是的，如果你只是需要简单的需要一个
后台运行的线程，不用RxJava也可以。

那么我们考虑一个复杂一点的场景，你需要从两个或以上不同的API并行的获取数据，并且只有在这些API的数据全部都返回回来之后才
才更新一个view。如果你使用传统的方式来处理，你需要些很多不必要的代码以确保这些请求都完成而且没有什么错误。

再考虑另外一个场景，一个后台任务需要在前一个后台任务成功执行之后开始执行。如果用传统的方法，这
将会走进回调地狱（callback hell）。

使用RxJava的operator（操作符），两个场景都可以优雅的解决。比如，如果用`fetchData`方法从两个不同的站点获取数据，假设这两个站点是
Yahoo和Google。这就需要创建两个`Observable`对象，并使用`subscribeOn`方法让他们裕兴在不同的线程上。
```java
 yahooObservable.subscribeOn(Schedulers.newThread());
 googleObservable.subscribeOn(Schedulers.newThread());
```
要处理第一个场景，两个请求需要并行的运行。我们可以使用`zip`操作符（operator）然后添加观察者。
```java
yahooObservable.subscribeOn(Schedulers.newThread());
googleObservable.subscribeOn(Schedulers.newThread());

Observable<String> zippedObservable = Observable.zip(yahooObservable, googleObservable, new Func2<String, String, String>() {
    @Override
    public String call(String yahooString, String appleString) {
        String result = yahooString + "\n" + appleString;
        Log.d(TAG, result);

        return result;
    }
});
```
要处理后面一种场景的问题，可以使用方法`concat`操作符（operator）来运行有依赖关系的两个线程。
```java
Observable.concat(yahooObservable, googleObservable)
    .subscribe(new Action1<String>() {
        @Override
        public void call(String s) {
            Log.d(TAG, "concat data: " + s);
            Toast.makeText(RxJavaActivity.this, "concat data: " + s, Toast.LENGTH_SHORT).show();
        }
    });
```

##5. 处理事件
RxAndroid有一个类叫做`ViewObservable`。专门用来方便处理view对象的各种相关事件。下面的
代码将演示如何使用`ViewObservable`对象来处理`Button`的点击事件。
由于RxAndroid的改动，你需要给项目的依赖项里添加新的库：
```
compile 'com.jakewharton.rxbinding:rxbinding:0.4.0'
```
```java
Button eventButton = (Button) findViewById(R.id.event_button);
RxView.clicks(eventButton).subscribe(new Action1<Void>() {
    @Override
    public void call(Void aVoid) {
        Toast.makeText(RxJavaActivity.this, "Clicked", Toast.LENGTH_SHORT).show();
    }
});
```
`RxView.clicks(eventButton)`返回一个`Observable`的对象。我们可以给这个对象调用各种前文
学到的操作符（operator）。比如，我们要按钮跳过前三次点击，可以这么做：
```java
RxView.clicks(eventButton)
    .skip(3)
    .subscribe(new Action1<Void>() {
        @Override
        public void call(Void aVoid) {
            Toast.makeText(RxJavaActivity.this, "Clicked", Toast.LENGTH_SHORT).show();
        }
});
```

##最后
这里用到了RxJava的观察者`Observer`、`Observable`还有相关的操作符（operator）来处理异步操作和
事件。使用Rxjava会涉及到函数式编程、响应式编程等一些Android开发者几乎不会涉及到的编程模式。
第一次接触难免会遇到一些困难，这都无所谓。只要继续学习相关内容，都会习以为常。

[原文](http://code.tutsplus.com/tutorials/getting-started-with-reactivex-on-android--cms-24387)中的内容很多已经不再可用。不过这里已经补齐了各种库修改之后的依赖项并按照修改之后的API重写了全部相关的示例代码。

相关代码都在[这里](https://github.com/future-challenger/okhttp-retrofit-rxandroid)。























