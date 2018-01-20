#使用Golang+Mongodb打造你的第一个站点

很多人推荐MEAN来开发站点。MEAN就是M：mongodb，E：expressjs、A：angular最后的N：nodejs。
但是如果你亲身的体会到了js的嵌套回调的话你就会想换换别的办法了。虽然说可以用promise等框架
破解深深的嵌套，但毕竟不是语言本身支持的。

为什么说用Golang呢？因为Golang是一个有着动态语言的特点的静态语言。语法简单，语法糖就是尽量减少语法。
编译的时候就可以把错误排查掉很多。不用像动态语言那样运行的时候才掉进坑里。

在Golang众多的Web框架中发现了[Martini](https://github.com/go-martini/martini)。
他后来觉得这个框架用了太多反射影响了效率，就有写了另一个框架或者按照作者的说法“中间件”，
叫做[Negroni](https://github.com/codegangsta/negroni)。具体的开发原因看[这里](https://codegangsta.io/blog/2014/05/19/my-thoughts-on-martini/)。
可能是这个作者爱喝酒把他写的框架都叫了个酒水的名字。Margini这个框架是作者受了Expressjs的启发写出来的。
所以在很多使用上的东西非常类似，比如route的绑定语法，支持类中间件的绑定语法等。也看过Golang的其他框架比如Revel，
有很多的酷炫的功能，和一站式的很多的支持，但是因为Martini的灵活决定选用Margini。

##数据库MongoDB
数据库方面，对于事物和数据的一致性要求没有那么强的话，还是选用比较简单的NoSQL数据库比较合适。所以，就用MongoDB了。
也许在严谨的工作中不应该这么用。但是MongoDB可以简单到，只要把Http POST过来的json数据直接写入数据库中。
如果json比较简单的话，完全不用做什么设计！当然，实际工作的时候是不可以这样的。我们讨论的只是这样的一个可行性。

那么我们就开始用Martini（golang)和MongoDB来打造一个web APP。这个app的名称就叫做navigator（领航者）。
是用来接受、存储和分析用户反馈以及用户的各种问题的，比如各种客户反馈回来的用户问题。这些问题的上下文信息十分有限。
所以我们让用户通过手机客户端来调用navigator的接口来同时提交相关的上下文信息。
关于Golang的一些基础的东西就不在这里探讨了。
go get获取Martini，下载MongoDB。之后使用命令run起来MongoDB。给DB起一个名字：NDB。然后在里面填充一个document。
MongoDB如果没有insert一个Document的话，那么这个数据库其实是没有创建的。这个Document就是一个Json。格式为：
```json
{
	"feedbackcode" : "123",
	"username" : "",
	"phonenum" : "",
	"deviceinfo" : {
		"systemtype" : "",
		"systemver" : "",
		"appver" : ""
	},
	"imagelist" : [ ],
	"desc" : "testing",
	"feedbacktype" : "",
	"other" : ""
}
```
使用`feedbackCode`来标示一个唯一的feedback。数据库的问题就彻底解决了。

##站点的目录结构
下面开始我们的站点部分。首先了解一下目录结构：
```
navigator
|———conf
|-————–Conf.go				# 配置信息
|—————–ConfigError.go		 # 错误名称
|———controllers
|—————–FeedbackController.go
|—————–IndexController.go
|–––models
|––––––FeedbackModel.go
server.go
```
这个目录结构式典型的REST API的MVC模式体现。我们这里不需要处理View，也就是Html的东西。但是，
还是显示的区分了Model和Controller。view其实还是有的，只不过是最后render后的json。

###RUN起来
首先，让我们的web app跑起来。
给server.go添加如下的代码：
```go
package main

import "github.com/go-martini/martini"

func main() {
  m := martini.Classic()
  m.Get("/", func() string {
    return "Hello world!"
  })
  m.Run()
}

```
`go run server.go`命令就可以让这个web app运行起来了。默认的是运行在3000端口上的。
在页面上也只能看到光秃秃的几个字*Hello world!*
`martini.Classic()`调用之后，Martini会为你默认提供一些工具。其中最重要的一个就是**路由**。
在这之后就可以使用`m.Get("/", func()string{return "Hello, world!"}`来指定什么路径映射到什么处理方法上。
最后调用`m.Run()`方法在3000端口上运行整个app。

现在开始要真正的开发这个app了。首先我们需要有一个简单的需求分析，我们到底是要做什么。
如前所述，用户可以用这个API发送*feedback*上来，然后在另外一个接口里把这些内容以列表的形式展示出来。
需求就是这么简单。我们下面来一步一步实现这些功能。

##Controllers
由于insert一个feedback，需要的信息量还是很大的。我们用POST得方法发送所需要的feedback数据。
所以，我们需要一个POST的路径和处理函数**Handler**(以后都叫做Handler）。
```go
m.Post("/", func(req *http.Request) string{
	// 从req中取出参数，插入到数据库
})
```
现实feedback的列表的接口只需要响应一个GET请求。所以是一个GET的路径和Handler。
看起来应该是这样的：
```go
m.Get("/", func() string {
	// 假设最后的字典会被转换成Json串
	return []map[string]string{map[string]string{"feedbackcode": "1", "username": "John"}, 
							   map[string]string{"feedbackcode": "2", "username": "Tony"}}
}
```
以上列出的Handler都是对于API添加数据和数据列表的一些模拟。下面开始开始正式写Controller。
###列表Controller
先看代码，然后再来详细说明：
```go
package controllers

import (
	// "fmt"
	"fmt"
	"github.com/martini-contrib/render"
	"navigator/conf"
	"navigator/models"
	"net/http"
)

type FeedbackController struct {
}

func (c *FeedbackController) FeedbackList(req *http.Request, r render.Render) {

	f := new(models.FeedbackModel)
	r.JSON(200, map[string]interface{}{"data": f.ModelList(nil)})
}
```
引入`github.com/martini-contrib/render`, *render*是一个渲染的包。
如果有html的视图的话，可以把数据转换成html模板可用的数据。我们这里用这个包来把字典转换成json数据。
`r.JSON(200, map[string]interface{}{"data": f.ModelList(nil)})`中200，是状态码。
需要转换的字典是一个string为key，Model列表为值的字典。

这里可以先了解一部分feedback的Model类：
```go
type DeviceInfo struct {
	SystemType string `json: "systemType"`    // ios or android
	SystemVer  string `json: "systemVersion"` // system version eg, ios 9.1
	AppVer     string `json: "appVersion"`    // app version eg, 2.9.0
}

type FeedbackModel struct {
	FeedbackCode string     `json: "feedbackCode"`
	UserName     string     `json: "userName"`
	PhoneNum     string     `json: "phoneNum"`
	DeviceInfo   DeviceInfo `json: "deviceInfo"`
	ImageList    []string   `json: "imageList"`
	Desc         string     `json: "description"`
	FeedbackType string     `json: "feedbackType"` // normal feedback or a problem in use
	Other        string     `json: "other"`        // other infomation
}
```
在最开始的json数据定义，在Golang的Model类定义中可以定义为上面的代码。
用**"``"**标示出来的，是在Model转换为json字符串的过程中Model字段和json串的key值得对应关系。
最后Controller的功能是需要处理请求过来的数据的，
也就是Controller的方法会作为Get或者Post的Handler的。这两者之间需要关联起来：
```go
package main 

import (
	// other packages
	"navigator/controllers"
)

func main() {
	var m = martini.Classic()
	var r = martini.NewRouter()
	
	feedbackController := new(controllers.FeedbackController)
	
	r.Get("/nav/feedback", feedbackController.FeedbackList)
	r.Post("/nav/feedback", feedbackController.Feedback)
	m.Action(r.Handle)
	
	m.RunOnAddr(":9090")
}
```
这里`r.Get("/nav/feedback", feedbackController.FeedbackList)`，修改了之前的访问路径。
**/nav/feedback**是在路径上加了一个项目的名称简写作为限定。因为直接叫做**feedback**有可能会和其他的同名接口混了。
也就是接口的污染。所以加一个项目名称来避免这个问题。
`feedbackController := new(controllers.FeedbackController)`初始化FeedbackController。
把`feedbackController`的方法`FeedbackList`和访问的路径绑定到一起。
**GET**的Http访问和`FeedbackList`方法绑定，**POST**的Http访问和`Feedback`方法绑定。
最后`m.RunOnAddr(":9090")`代替了之前使用的`m.Run()`。之前app是运行在3000端口上的。
现在通过`m.RunOnAddr(":9090")`则app将运行在9090端口上。也就是可以用这一语句来制定app的运行端口。

##Models和MongoDB
处理MongoDB我们用mgo这个库。Golang虽然这么多年了已经。还出了这么多的明星产品。
但是，处理MongoDB的成熟的库还是可以说没有。完全没有想python、nodejs等语言的库那么丰富。
有标准的，还有很多github上的其他开发者开发的库。Golang里也就是mgo相对来说这个库比较可用的了。

当POST请求的feedback的json字符串传上来的时候，会被转换为一个`FeedbackModel`的对象。
Golang的json库会帮助我们做到这一点。
`json.Unmarshal([]byte(dict), &f)`这里的`Unmarshal`方法会把一个json字符串转化为FeedbackModel对象。
这里需要用到一个工厂方法。生成一个Model对象，并且这个对象应该是一个**富Model对象**。
后面这个Model对象将具有可以操作操作数据库，增、删、改、查的方法。代码：

```go
func NewFeedbackModel(dict string) *FeedbackModel {
	var f FeedbackModel
	err := json.Unmarshal([]byte(dict), &f)
	if err != nil {
		return nil
	}

	return &f
}
```
以上代码会用到库`"encoding/json"`。当收到feedback的json串的时候就用这个方法将其转换为FeedbackModel对象。
上文说过，会给这个Model增加数据库操作的方法。那么我们就可以通过json串得到的Model来执行数据的操作。
代码：
```go
func (m *FeedbackModel) ModelList(conditions map[string]string) []FeedbackModel {
	confInstance := conf.ConfigInstance()
	mgoUri := conf.ConfigInstance().MongoDBConnectionString()
	// var f FeedbackModel
	session, err := mgo.Dial(mgoUri)
	if err != nil {
		fmt.Printf("can not connect to server %v\n", err)
		panic(err)
	}
	defer session.Close()

	var feedbackModelList []FeedbackModel

	collection := session.DB(confInstance.DBName).C("feedback")
	err = collection.Find(conditions).All(&feedbackModelList)

	return feedbackModelList
}

func (m *FeedbackModel) GetModel(modelId string) (*FeedbackModel, error) {
	//此处省略一部分代码
	collection := session.DB(confInstance.DBName).C("feedback")
	err = collection.Find(bson.M{"feedbackcode": modelId}).One(&f)
	
	return &f, nil
}

func (m *FeedbackModel) InsertModel(model *FeedbackModel) error {
	//此处省略一部分代码
	f := session.DB(confInstance.DBName).C("feedback")
	err = f.Insert(model)

	return err
}
```
这里需要稍微的提到一部分MongoDB的知识。在MongoDB中，库还是库，但是表不叫做表，叫做Collection。
关键字不翻译。翻译出来的看着太烂，还不能清楚表达意思。每一条记录也不叫记录，叫做Document。
一个Document的大小不能超过2M。

在MongoDB里查找是这样的：
```javascript
db.feedback.find({"feedbackCode": "1"})	//查找条件就是{"feedbackCode":"1"})	//查找条件就是
```
如果在find里什么都不写，那么就是无条件的全部查找。插入是这样的：
```javascript
db.feedback.insert({"feedbackCode": "2"})
```
Golang的**mgo**当然在具体操作数据库方面和MongoDB本身是不会差太多了，除了这是Go语法的封装。
```go
collection := session.DB(confInstance.DBName).C("feedback")
```
这样就得到了一个Collection。查找、插入、修改等动作都是在Collection上执行的。所以，查找就是这样的：
```go
collection.Find(conditions).All(&feedbackModelList)
```
查找到的列表会存放在一个事先声明好的feedback的Model列表里。查找一个可以这样：
```go
collection.Find(bson.M{"feedbackcode": modelId}).One(&f)
```
插入：
```go
collection.Insert(model)
```
insert的时候，这个model需要是指针类型的，也就是：`var model *FeedbackModel`。

到这里就可以把整个的feedback从请求到返回的过程都串联起来了。

##全部串联起来
在整个的app入口处。请求的路径和具体的Controller处理函数绑定在了一起。每一个请求来了之后
，如果路径和绑定的路径是对应好了的。就会被发送给Controller的某个绑定好的函数里处理。

这个负责处理http请求的Controller方法会把http请求发送上来的json字符串发送给Model模块处理。
Model模块会把json字符串转换为一个Model实例。这个Model是一个**富Model**，
可以调用其函数把Model本身的数据插入到Mongo数据库中，当然也可以从数据库中查询这些数据。

##到这里
到这里你已经熟悉了如何用Martini这个Golang框架开发一个简单地站点，好吧，其实是一个REST Api。
使用我们文中提到的Render框架，你可以非常非常容易的把Model数据转换之后并显示在html的template中。
而且你已经熟悉了MongoDB数据库。一个非常简单易用的数据库。无论你是要开发一个REST Api还是要开发一个html的站点。
需要存储的东西都可以存如MongoDB数据库中。到这里你已经可以顺利的继续下面的功能的开发了。