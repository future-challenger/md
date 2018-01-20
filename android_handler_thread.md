一般`Handler`+`Thread`放在一起用于打造一个线程并在线程运行到某个阶段
的时候使用Handler更新UI。`AsyncTask`也是差不多一个用处。在后台执行一个任务，
也可以在某一个进度的时候更新UI。两者机器相似，这里我们就用`Thread`和`Handler`来模拟一个
`AsyncTask`。

```java
public class NormalActivity extends Activity {
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            // ...
        }
    };
}
```
在一个Activity里，这里是`NormalActivity`，初始化一个`Handler`。然后在这个`Handler`
里重写`handleMessage`方法。在这里处理线程发出来的更新UI的消息（Message msg）。

这里我们把发出的更新消息更新到一个`TextView`上。没次显示一个进度百分比