#开始编写Golang代码

##介绍

本文主要讲述如何写一个简单的Go包和如何使用golang的工具，如何获取、编译和安装Go的包，以及如何使用go的命令。

Go的工具需要将代码按照一定的方式来组织。所以请认真阅读本文。

##代码的组织

###workspace

go工具是设计用来处理公开代码库的开源代码的，虽然你不是一定要公开你的代码，但是工作的模式是一样的。

Go代码必须保存在一个workspace中。一个workspace必须要在根目录下包含三个子目录：

* src 包含了Go的源文件，这些源文件以package的方式存在
* pkg 包含了包对象
* bin 包含了可执行文件

Go工具会编译源文件并把编译成的二进制文件存放在pkg和bin目录中。

如下是一个典型的go项目的目录：
```
bin/
    hello                           # 命令可执行文件<br/>
    outyet                          # 命令可执行文件
```
```
pkg/
    linux_amd64/
        github.com/golang/example/
            stringutil.a            # 包对象
```
```
src/
    github.com/golang/example/
        .git/
        hello/
            hello.go                # 源文件
        outyet/
            main.go                 # 源文件
            main_test.go            # 测试 源文件
        stringutil/
            reverse.go              # 源文件
            reverse_test.go         # 测试 源文件
```

这个workspace只包含了一个代码库（example），其中包含了两个包hello和outyet以及一个库stringutil。

一个典型的workspace可以包含多个代码库，以及每个库中的多个包和库。多数的Go开发者把这些都放在一个单独的workspace中。

包和库是从不同的源文件包中编译而成的。我们会稍后讨论这一点。

##GOPATH环境变量

GOPATH环境变量指定的就是你的workspace的位置。这个极可能是你开发Go项目的时候唯一的一个需要设定的环境变量。

开始前先创建一个workspace目录，然后在GOPATH中指定这个workspace目录的位置。你可以把GOPATH指定在任何的你喜欢的位置。

但是本文会使用$HOME/work为workspace目录的位置。注意，这个目录绝对不可以和你的go的安装目录相同。
```
$ mkdir $HOME/work
```
```
$ export GOPATH=$HOME/work
```
然后把workspace的bin目录添加到PATH中。
```
$ export PATH=$PATH:$GOPATH/bin
```
###包路径

标准库的包路径都会用简写：“fmt”和“net/http”。你自己的包就需要选择一个路径了。因为，你不会希望以后和标准库的包或者其他
使用的包的名称互相冲突的。如果你把你的代码保存在某代码库中，那么你应该使用这个代码库的根作为目录使用：
```
$ mkdir -p $GOPATH/src/github.com/user
```

###你的第一个程序

要编译、运行你的程序，首先选择一个包路径（我们会使用github.com/user/hello）并在你的workspace里创建一个相应的包路径
```
$ mkdir $GOPATH/src/github.com/user/hello
```
然后，创建一个hello.go的文件。并在这个文件中添加如下的代码：

```go

package main

import “fmt”

func main() {

fmt.Println(“Hello, world!”)

}

```

现在你可以编译并运行你的代码了：
```
go install github.com/user/hello
```

注意你可以在你的系统的任何地方来运行这个命令。go工具可以根据GOPATH从workspace的github.com/user/hello找到源文件
上面的命令行执行之后，会在workspace的bin目录下生成一个可以自行文件，这里是hello（或者，windows下的hello.exe）。在我们的
例子中目录为：$GOPATH/bin/hello。

go工具只会在出错的时候打印输出信息。所以，如果没有打印任何信息的话那就是编译成功了。
现在你可以运行你的程序了：
```
$　$GOPATH/bin/hello 
Hello, world! 
```

或者，你已经在PATH中添加了GOPATH/bin，只需要输入可执行文件的名字：
```
$ hello
Hello, world!
```
如果你使用了代码管理工具，那么这就可以初始化一个代码库了。添加文件；并commit你的第一次更改。但是，这不是一定要的，但是
你可以不用代码工具编写go代码。
```
$ cd $GOPATH/src/github.com/user/hello
$ git init

Initialized empty Git respository in /home/user/work/src/github.com/user/hello/.git/

$ git add hello.go
$ git commit -m “initial commit”

[master (root-commit) 0b4507d] initial commit
1 file changed, 1 insertion(+)
    create mode 100644 hello.go
```
把代码发布到远端代码库，让其他的读者可以看到你的代码。

###你的第一个库

接下来，我们创建一个库，然后在hello代码中使用这个库。
按照常理我们需要选择一个包路径（我们这里使用github.com/user/stringutil）并创建包路径
```
$ mkdir $GOPATH/src/github.com/user/stringutil
```
然后，穿件一个文件：reverse.go, 内容为：
```go
package stringutil

func Reverse(s string) string {
    var r = []rune(s)

    for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }

    return string(r)
}
```
现在，用go build命令测试一下代码是否能够编译。
```
$ go build github.com/user/stringutil
```
或者你在这个包的路径下的话只需要：
```
$ go build
```
这不会生成任何文件。要生成文件的话就必须要使用go install。执行完这个命令之后就会把这个包编译后生成在pkg目录中。
在stringutil编译之后，修改hello.go的代码：
```go
package main

import (
    “fmt”
    “github.com/user/stringutil”
)

func main() {
    fmt.Println(stringutil.Reverse(“!oG ,olleH”))
}
```
任何时候go工具安装一包或者一个二进制文件的时候，其相关的依赖也都会一起安装。因此，当你安装hello的时候：

```
$ go install github.com/user/hello
```
stringutil包也会自动安装。运行新版本的程序，你会看到一个新的，反转的消息：
```
$ hello

Hello, Go!
```

以上的步骤都执行完成之后，你的目录看起来是这样的：
```
bin/
    hello
```
pkg/
    linux_amd64/
        github.com/user/
            stringutil.a

src/
    github.com/user/
        hello/
            hello.go
        stringutil/
            reverse.go
```
注意go install把stringutil.a放在了pkg/linux_amd64目录中，并且会在linux_amd64下创建和源文件所在的目录相同的结构。
go工具之后的编译中如果发现pkg中已经存在这一文件，那么就不会再次编译。这样可以避免不必要的重复编译。linux_amd64目录是为了交叉编译
，并且反映出所在的操作系统和系统架构。

Go的可执行文件是静态链接的。

##包名称

Go的源文件第一行必须是：
```
package name
```
包名称是import的时候使用的默认名称，同一个包中的文件必须使用同样的包名称。
Go的惯例是包名称是import语句的最后一个元素。crypto/rot13引入的包就应该命名为rot13.
main函数所在的包必须用package main作为包名称。
包的名称没有要求必须要唯一，但是报的整个引入路径必须要唯一。

##测试

Go有一个轻量级的测试框架，由go test和testing包组成。
写一个测试只需要，创建一个_test.go结尾的文件，这个文件里包含了名称为TestXXX的方法，这些测试方法的签名为func (t *testing.T)。
测试框架会运行每一个这样的方法。如果某个测试方法调用了t.Error或者t.Fail, 那么这个测试被认为是失败的。
给stringutil添加一个测试。在$GOPATH/src/github.com/user/sringutil/目录创建文件reverse_test.go。这个文件包含
如下代码：
```go
package stringutil

import “testing”

func TestReverse(t *testing.T) {
    case := []struct {
        in, want string
    }

    {
        {“Hello, world”, “dlrow ,olleH”},
        {“Hello, 世界”, “界世 ,olleH”},
        {“”, “”},
    }

    for _, c := range cases {
        got := Reverse(c.in)
        if got != c.want{
            t.Errorf(“Reverse(%q) == %q”, c.in, got, c.want)
        }
    }
}
```
然后使用命令go test来运行测试：
```
$ go test github.com/user/stringutil

PASS
ok          stringutil          2.223s 
```
如果实在package内部运行命令的话，可以省去包的路径。
```
$ go test

PASS
ok          stringutil          2.223s
```
运行命令go help test可以看到更多的关于测试命令的细节。

##Remote packages 远端代码库

一个import路径和使用Git获取远程代码的路径是一样的。go工具使用这个路径自动从远端代码库获取代码。比如，上面使用的代码
托管在GitHub的路径为：`github.com/golang/example`。如果在代码的import语句中包含了远端代码，go get命令会自动获取、编译
并install（安装）这些代码。

```
$ go get github.com/golang/example/hello

$ $GOPATH/bin/hello
Hello, Go examples!
```
如果需要的包不在workspace中，`go get`命令会把这些包放在GOPATH指定的第一个workspace中。如果包已经存在，那么go get

命令会跳过远程获取而直接执行`go install`类似的命令。

在执行以上的go get命令之后，workspace目录看起来就是这样的：
```
bin/
    hello

pkg/
    linux_amd64/
        github.com/golang/example/
            stringutil.a
        github.com/user/
            stringutil.a

src/
    github.com/golang/example/
        .git/
        hello/
            hello.a
        stringutil/
            reverse.go
            reverse_test.go
        github.com/user/
            hello/
                hello.go
            stringutil/
                reverse.go
                reverse_test.go
```
GitHub上托管的hello依赖于同一个包中的stringutil。hello.go中的import语句使用的是一样的惯例。因此go get命令
可以定位并安装（install）依赖包。
`import “github.com/golang/example/stringutil`
这一惯例会让你的代码非常容易被别人使用。

接下来。。。

Effective Go有更多的关于有效编写Go代码的内容。<br/>
A Tour of Go有更多的关于Go的知识。
