#��ʶAndroid��ReactiveX
����һ������һ���AndroidӦ�ö����õ��������󣬽����Ͷ�������Щ����ζ��
Ҫд�ܶ�Ļص�Ƕ�ס������Ĵ���Ҳ����Ϊ*callback hell*���ص���������������
���벻������������⣬����Ҳ�Ǵ���߷��ĵط���[ReactiveX](http://reactivex.io/intro.html)
�ṩ��һ��������׼ȷ�����첽������¼��ķ�����

RxJava��һ��ReactiveX��JVM�ϵ�ʵ�֣���NetFlix�������������Java��������
��Ϊ����������̳������ѧ�������AndroidӦ�ÿ�����ʹ��RxJava������Android�е�RxJava
���Լ��ΪRxAndroid��

##1. ����RxAndroid
Ҫ��android��ʹ��RxAndroid����Ҫ��**build.gradle**�����compile�����
```groovy
compile 'io.reactivex:rxjava:1.1.1'
```
##2. Oberser��Observable�Ļ���
��ʹ��RxJava��ʱ����ᾭ������`Observer`��`Observable`�ĸ��
����԰�`Observable`���Ϊ���ݵ������ߣ���`Observer`��������Ϊ���ݵ������ߡ���RxJava��
���ݵ������߶���`Observable`���ʵ���������߶��ǽӿ�`Observer`��ʵ����

��`Observable`�кܶྲ̬��������Ϊ**operator**����������`Observable`��ʵ��������
������ʾ�����ʹ�÷���`just`���������������˵����**operator**��������һ���ǳ���
��ʵ�������ṩһ��`String`���ݣ�
```java
Observable<String> theObservable = Observable.just("hello world!");
```
���Ǹոմ�����ʵ��������������һ���۲��ߵ�ʱ�򷢳����ݡ�Ҫ����һ���۲��ߣ�����Ҫ����
һ��ʵ���˽ӿ�`Observer`���ࡣ�ӿ�`Observer`��ķ������Դ����`Observable`ʵ��������֪ͨ��
����Ĺ۲��߿��԰�`Observable`ʵ�����������ݴ�ӡ������
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
1. ��Observableʵ��û�������ڷ�����ʱ����á�
2. ��Observableʵ�����������ʱ����á�
3. ��Observableʵ��ÿ�η������ݵ�ʱ����á�

Ҫ���۲���ָ��һ��observable��ʵ����������Ҫ����`subscribe`��������������᷵��һ��`Subscription`
ʵ��������Ĵ�����`theObserver`�۲��߹۲�`theObservable`��
```java
Subscription theSubscription = theObservable.subscribe(theObserver);
```
���۲�����ӵ���observableʵ���У�observableʵ���ͷ������ݡ���ˣ������������Ĵ������
����**hello world!**����ӡ�������������£�
```java
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- hello world!
okhttp.demo.com.okhttpdemo D/RxJavaActivity: completed
```
һ����˵����`onCompleted`��`onError`�����õ�������ĳЩ�������Ҳ���õ����������ﲻ��������
���ԣ��и��Ӽ��Ŀ����õ�`Action1`�ӿڣ�����ӿ�ֻ��һ��������
```java
Action1<String> theAction = new Action1<String>() {
    @Override
    public void call(String s) {
        Log.d(TAG, "action1 data:- " + s);
    }
};

Subscription theSubscription = theObservable.subscribe(theAction);
```
�����ύ��һ���ӿ�`action1`��ʵ��֮��`Observable`��ʵ���ͷ������ݡ�

Ҫ��observable��ȥ��һ���۲���ֻ��Ҫ��`Subscription`ʵ���ϵ��÷���`unsubscribe`��
```java
theSubscription.unsubscribe();
```

##3. ʹ��Operator
�������Ѿ�֪����δ����۲��ߺ�observable���ɹ۲���󣩡��������������ʹ��RxJava��operator����������
��������ת��observable�ģ���Ȼ����������һЩ���������ڴ���һ������һ���`Observable`����
������󷢳�һ��`Integer`�������ݡ�����`from`����������ܡ�
```java
Observable<Integer> listObservable = Observable.from(new Integer[]{1, 2, 3, 4, 5});
listObservable.subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        Log.d(TAG, "data:- " + String.valueOf(integer));
    }
});
```
������δ�����ᷢ�������е�����һ����һ���Ĵ�ӡ�˳�����
```java
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 1
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 2
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 3
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 4
okhttp.demo.com.okhttpdemo D/RxJavaActivity: data:- 5
```
�������Ϥjavascript����kotlin, ��ͻ��`map`��`filter`���������������ʹ����һ������ʶ��
RxJavaҲ�����Ƶ�operator������observable���ɹ۲���󣩡�����java 7û��lambda���ʽ��������Ҫ
ģ��һ�����lambda���ʽ��Ҫģ��ֻ����һ��������lambda���ʽֻҪ����һ��ʵ���˽ӿ�`Func1`��ʵ����

�����������`map`����������`listObservable`��ÿһ��Ԫ�ء�������ӻ����observableÿ�����������������ֵ�ƽ��ֵ��
```java
listObservable.map(new Func1<Integer, Integer>() { // 1
    @Override
    public Integer call(Integer integer) {
        return integer * integer; // 2
    }
});
```
1. ����������ֵ����`Integer`���͵ġ�
2. ������ֵ�ƽ��ֵ��

������һ����Ҫע��ĵط���`map`�������ص���һ���µ�`Observable`������������������ı�
ԭ����`Observable`��ʵ���������`listObsevable`����˹۲��ߣ��ͻ᷵����Щ���ֵ�ƽ��ֵ��

Operator�������������Գ���ʽ���á����磬���´���ʹ����`skip`��������ǰ�������֣���ʹ��`filter`����
������������
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
�����
```java
okhttp.demo.com.okhttpdemo D/RxJavaActivity: skip & filter data:- 4
```
##4. �ô����첽������
����ǰ�洴����observer���۲��ߣ���observable���ɹ۲���󣩶����ڵ������߳������еģ�����Android��UI�߳��С�
�������Ҫ��RxJava������һЩ���̵߳����⡣ͬʱ����ʾ���callback hell���ص������������⡣

����������һ������`fetchData`�ķ��������������������API�ϻ�ȡ���ݡ������������һ��URL�ַ���
Ϊ������������һ���ַ������������������ôʹ�ã�
```java
String content = fetchData("https://api.github.com/orgs/octokit/repos");
```
�����ִ������*curl -i https://api.github.com/orgs/octokit/repos*�ǻ᷵��һ��json�ַ����ġ�
�ǵģ����URL��github��API��һ�����ӡ��������������ʡ�

��Android�������������UI�߳��У�ֻ�����⿪��һ���̻߳���ʹ��`AsyncTask`������RxJava֮��
��Ͷ���һ��ѡ�����`subscribeOn`��`observeOn`��������operator�����������ʾ��ָ����̨�������ĸ��߳�
���У����½�����������ĸ��̣߳���Ȼ���������UI�̣߳���

�����ڼ�������֮ǰ��������Ҫ��RxAndroid����չ��ӵ���Ŀ������������RxJava��Android�ϵ���չ��
```groovy
compile 'io.reactivex:rxandroid:1.1.0'
```

����Ĵ�����ʾ���ʹ��`create`��������operator)����һ��`Observable`����ʹ���������������
`Observable`�������ʵ��`Observable.OnSubscribe`�ӿڣ�������`onNext`, `onError`��`onComplete`
�����Ʒ���ʲô���ݵȲ�����
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
1. �ѷ���`fetchData`��ȡ�����ݷ���ȥ��
2. ����û��ʲôҪ���ġ�
3. ��������˴�����ô�ͷ���������Ϣ��

Ȼ��Ϳ��Ը����`asyncObservable`, `Observable`����������`subscribeOn`��`observeOn`����������ָ��ִ�е�
�̡߳�
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
��Ҳ��������û���ı�`AsyncTask`����`Thread`+`Handler`����Ϻá��ǵģ������ֻ����Ҫ�򵥵���Ҫһ��
��̨���е��̣߳�����RxJavaҲ���ԡ�

��ô���ǿ���һ������һ��ĳ���������Ҫ�����������ϲ�ͬ��API���еĻ�ȡ���ݣ�����ֻ������ЩAPI������ȫ�������ػ���֮���
�Ÿ���һ��view�������ʹ�ô�ͳ�ķ�ʽ����������ҪЩ�ܶ಻��Ҫ�Ĵ�����ȷ����Щ������ɶ���û��ʲô����

�ٿ�������һ��������һ����̨������Ҫ��ǰһ����̨����ɹ�ִ��֮��ʼִ�С�����ô�ͳ�ķ�������
�����߽��ص�������callback hell����

ʹ��RxJava��operator�����������������������������ŵĽ�������磬�����`fetchData`������������ͬ��վ���ȡ���ݣ�����������վ����
Yahoo��Google�������Ҫ��������`Observable`���󣬲�ʹ��`subscribeOn`����������ԣ���ڲ�ͬ���߳��ϡ�
```java
 yahooObservable.subscribeOn(Schedulers.newThread());
 googleObservable.subscribeOn(Schedulers.newThread());
```
Ҫ�����һ������������������Ҫ���е����С����ǿ���ʹ��`zip`��������operator��Ȼ����ӹ۲��ߡ�
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
Ҫ�������һ�ֳ��������⣬����ʹ�÷���`concat`��������operator����������������ϵ�������̡߳�
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

##5. �����¼�
RxAndroid��һ�������`ViewObservable`��ר���������㴦��view����ĸ�������¼��������
���뽫��ʾ���ʹ��`ViewObservable`����������`Button`�ĵ���¼���
����RxAndroid�ĸĶ�������Ҫ����Ŀ��������������µĿ⣺
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
`RxView.clicks(eventButton)`����һ��`Observable`�Ķ������ǿ��Ը����������ø���ǰ��
ѧ���Ĳ�������operator�������磬����Ҫ��ť����ǰ���ε����������ô����
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

##���
�����õ���RxJava�Ĺ۲���`Observer`��`Observable`������صĲ�������operator���������첽������
�¼���ʹ��Rxjava���漰������ʽ��̡���Ӧʽ��̵�һЩAndroid�����߼��������漰���ı��ģʽ��
��һ�νӴ����������һЩ���ѣ��ⶼ����ν��ֻҪ����ѧϰ������ݣ�����ϰ��Ϊ����

[ԭ��](http://code.tutsplus.com/tutorials/getting-started-with-reactivex-on-android--cms-24387)�е����ݺܶ��Ѿ����ٿ��á����������Ѿ������˸��ֿ��޸�֮�������������޸�֮���API��д��ȫ����ص�ʾ�����롣

��ش��붼��[����](https://github.com/future-challenger/okhttp-retrofit-rxandroid)��























