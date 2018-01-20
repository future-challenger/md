# 使用redux-saga处理异步操作 (一)

我们需要面对一个现实，如果你已经按需使用技术，不是为了技术而技术，来建立React生态的话，这绝对是一件费力、费时的事。与Angular比起来更是如此！

这几天我（作者）在使用redux-saga的时候，发现redux-saga非常的有用。本文分享我（作者）在saga上的收获。希望可以对你们有所帮助。

## 首先
根据redux-saga的作者的介绍：
```
开发redux-saga就是要让React/Redux的副作用（那些异步的操作，比如获取服务器数据或者从浏览器的缓存里获取数据）处理起来更加简单。
```

### 举一个航班的例子
场景是这样的：
一个用户（`getUser()`)，准备乘坐飞机（`getFlight(user)`），在某个时间，离开（`getDeparture(user)`）。但是，还需要看看天气预报（`getForecast(date)`）。
我们要使用的API的代码基本上是这样的：
```js
class TravelServiceApi {
  static getUser() {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          email: "somemockemail@email.com",
          repository: "http://github.com/username"
        });
      }, 3000);
    })
  }

static getDeparture(user) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        userID : user.email,
        flightID : “AR1973”,
        date : “10/27/2016 16:00PM”
      });
     }, 2500);
  });
}

 static getForecast(date) {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve({
            date: date,
            forecast: "rain"
        });
      }, 2000);
    });
  }
}
```
上面的API非常简单。为了方便后面的代码使用，在里面加了些假数据。那么根据上面的API可以实现下面这样一个简单的dashboard。

