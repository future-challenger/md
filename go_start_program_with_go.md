#��ʼ��дGolang����

##����

������Ҫ�������дһ���򵥵�Go�������ʹ��golang�Ĺ��ߣ���λ�ȡ������Ͱ�װGo�İ����Լ����ʹ��go�����

Go�Ĺ�����Ҫ�����밴��һ���ķ�ʽ����֯�������������Ķ����ġ�

##�������֯

###workspace

go������������������������Ŀ�Դ����ģ���Ȼ�㲻��һ��Ҫ������Ĵ��룬���ǹ�����ģʽ��һ���ġ�

Go������뱣����һ��workspace�С�һ��workspace����Ҫ�ڸ�Ŀ¼�°���������Ŀ¼��

* src ������Go��Դ�ļ�����ЩԴ�ļ���package�ķ�ʽ����
* pkg �����˰�����
* bin �����˿�ִ���ļ�

Go���߻����Դ�ļ����ѱ���ɵĶ������ļ������pkg��binĿ¼�С�

������һ�����͵�go��Ŀ��Ŀ¼��
```
bin/
    hello                           # �����ִ���ļ�<br/>
    outyet                          # �����ִ���ļ�
```
```
pkg/
    linux_amd64/
        github.com/golang/example/
            stringutil.a            # ������
```
```
src/
    github.com/golang/example/
        .git/
        hello/
            hello.go                # Դ�ļ�
        outyet/
            main.go                 # Դ�ļ�
            main_test.go            # ���� Դ�ļ�
        stringutil/
            reverse.go              # Դ�ļ�
            reverse_test.go         # ���� Դ�ļ�
```

���workspaceֻ������һ������⣨example�������а�����������hello��outyet�Լ�һ����stringutil��

һ�����͵�workspace���԰����������⣬�Լ�ÿ�����еĶ�����Ϳ⡣������Go�����߰���Щ������һ��������workspace�С�

���Ϳ��ǴӲ�ͬ��Դ�ļ����б�����ɵġ����ǻ��Ժ�������һ�㡣

##GOPATH��������

GOPATH��������ָ���ľ������workspace��λ�á�������������㿪��Go��Ŀ��ʱ��Ψһ��һ����Ҫ�趨�Ļ���������

��ʼǰ�ȴ���һ��workspaceĿ¼��Ȼ����GOPATH��ָ�����workspaceĿ¼��λ�á�����԰�GOPATHָ�����κε���ϲ����λ�á�

���Ǳ��Ļ�ʹ��$HOME/workΪworkspaceĿ¼��λ�á�ע�⣬���Ŀ¼���Բ����Ժ����go�İ�װĿ¼��ͬ��
```
$ mkdir $HOME/work
```
```
$ export GOPATH=$HOME/work
```
Ȼ���workspace��binĿ¼��ӵ�PATH�С�
```
$ export PATH=$PATH:$GOPATH/bin
```
###��·��

��׼��İ�·�������ü�д����fmt���͡�net/http�������Լ��İ�����Ҫѡ��һ��·���ˡ���Ϊ���㲻��ϣ���Ժ�ͱ�׼��İ���������
ʹ�õİ������ƻ����ͻ�ġ���������Ĵ��뱣����ĳ������У���ô��Ӧ��ʹ����������ĸ���ΪĿ¼ʹ�ã�
```
$ mkdir -p $GOPATH/src/github.com/user
```

###��ĵ�һ������

Ҫ���롢������ĳ�������ѡ��һ����·�������ǻ�ʹ��github.com/user/hello���������workspace�ﴴ��һ����Ӧ�İ�·��
```
$ mkdir $GOPATH/src/github.com/user/hello
```
Ȼ�󣬴���һ��hello.go���ļ�����������ļ���������µĴ��룺

```go

package main

import ��fmt��

func main() {

fmt.Println(��Hello, world!��)

}

```

��������Ա��벢������Ĵ����ˣ�
```
go install github.com/user/hello
```

ע������������ϵͳ���κεط�������������go���߿��Ը���GOPATH��workspace��github.com/user/hello�ҵ�Դ�ļ�
�����������ִ��֮�󣬻���workspace��binĿ¼������һ�����������ļ���������hello�����ߣ�windows�µ�hello.exe���������ǵ�
������Ŀ¼Ϊ��$GOPATH/bin/hello��

go����ֻ���ڳ����ʱ���ӡ�����Ϣ�����ԣ����û�д�ӡ�κ���Ϣ�Ļ��Ǿ��Ǳ���ɹ��ˡ�
���������������ĳ����ˣ�
```
$��$GOPATH/bin/hello 
Hello, world! 
```

���ߣ����Ѿ���PATH�������GOPATH/bin��ֻ��Ҫ�����ִ���ļ������֣�
```
$ hello
Hello, world!
```
�����ʹ���˴�������ߣ���ô��Ϳ��Գ�ʼ��һ��������ˡ�����ļ�����commit��ĵ�һ�θ��ġ����ǣ��ⲻ��һ��Ҫ�ģ�����
����Բ��ô��빤�߱�дgo���롣
```
$ cd $GOPATH/src/github.com/user/hello
$ git init

Initialized empty Git respository in /home/user/work/src/github.com/user/hello/.git/

$ git add hello.go
$ git commit -m ��initial commit��

[master (root-commit) 0b4507d] initial commit
1 file changed, 1 insertion(+)
    create mode 100644 hello.go
```
�Ѵ��뷢����Զ�˴���⣬�������Ķ��߿��Կ�����Ĵ��롣

###��ĵ�һ����

�����������Ǵ���һ���⣬Ȼ����hello������ʹ������⡣
���ճ���������Ҫѡ��һ����·������������ʹ��github.com/user/stringutil����������·��
```
$ mkdir $GOPATH/src/github.com/user/stringutil
```
Ȼ�󣬴���һ���ļ���reverse.go, ����Ϊ��
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
���ڣ���go build�������һ�´����Ƿ��ܹ����롣
```
$ go build github.com/user/stringutil
```
���������������·���µĻ�ֻ��Ҫ��
```
$ go build
```
�ⲻ�������κ��ļ���Ҫ�����ļ��Ļ��ͱ���Ҫʹ��go install��ִ�����������֮��ͻ������������������pkgĿ¼�С�
��stringutil����֮���޸�hello.go�Ĵ��룺
```go
package main

import (
    ��fmt��
    ��github.com/user/stringutil��
)

func main() {
    fmt.Println(stringutil.Reverse(��!oG ,olleH��))
}
```
�κ�ʱ��go���߰�װһ������һ���������ļ���ʱ������ص�����Ҳ����һ��װ����ˣ����㰲װhello��ʱ��

```
$ go install github.com/user/hello
```
stringutil��Ҳ���Զ���װ�������°汾�ĳ�����ῴ��һ���µģ���ת����Ϣ��
```
$ hello

Hello, Go!
```

���ϵĲ��趼ִ�����֮�����Ŀ¼�������������ģ�
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
ע��go install��stringutil.a������pkg/linux_amd64Ŀ¼�У����һ���linux_amd64�´�����Դ�ļ����ڵ�Ŀ¼��ͬ�Ľṹ��
go����֮��ı������������pkg���Ѿ�������һ�ļ�����ô�Ͳ����ٴα��롣�������Ա��ⲻ��Ҫ���ظ����롣linux_amd64Ŀ¼��Ϊ�˽������
�����ҷ�ӳ�����ڵĲ���ϵͳ��ϵͳ�ܹ���

Go�Ŀ�ִ���ļ��Ǿ�̬���ӵġ�

##������

Go��Դ�ļ���һ�б����ǣ�
```
package name
```
��������import��ʱ��ʹ�õ�Ĭ�����ƣ�ͬһ�����е��ļ�����ʹ��ͬ���İ����ơ�
Go�Ĺ����ǰ�������import�������һ��Ԫ�ء�crypto/rot13����İ���Ӧ������Ϊrot13.
main�������ڵİ�������package main��Ϊ�����ơ�
��������û��Ҫ�����ҪΨһ�����Ǳ�����������·������ҪΨһ��

##����

Go��һ���������Ĳ��Կ�ܣ���go test��testing����ɡ�
дһ������ֻ��Ҫ������һ��_test.go��β���ļ�������ļ������������ΪTestXXX�ķ�������Щ���Է�����ǩ��Ϊfunc (t *testing.T)��
���Կ�ܻ�����ÿһ�������ķ��������ĳ�����Է���������t.Error����t.Fail, ��ô������Ա���Ϊ��ʧ�ܵġ�
��stringutil���һ�����ԡ���$GOPATH/src/github.com/user/sringutil/Ŀ¼�����ļ�reverse_test.go������ļ�����
���´��룺
```go
package stringutil

import ��testing��

func TestReverse(t *testing.T) {
    case := []struct {
        in, want string
    }

    {
        {��Hello, world��, ��dlrow ,olleH��},
        {��Hello, ���硱, ������ ,olleH��},
        {����, ����},
    }

    for _, c := range cases {
        got := Reverse(c.in)
        if got != c.want{
            t.Errorf(��Reverse(%q) == %q��, c.in, got, c.want)
        }
    }
}
```
Ȼ��ʹ������go test�����в��ԣ�
```
$ go test github.com/user/stringutil

PASS
ok          stringutil          2.223s 
```
���ʵ��package�ڲ���������Ļ�������ʡȥ����·����
```
$ go test

PASS
ok          stringutil          2.223s
```
��������go help test���Կ�������Ĺ��ڲ��������ϸ�ڡ�

##Remote packages Զ�˴����

һ��import·����ʹ��Git��ȡԶ�̴����·����һ���ġ�go����ʹ�����·���Զ���Զ�˴�����ȡ���롣���磬����ʹ�õĴ���
�й���GitHub��·��Ϊ��`github.com/golang/example`������ڴ����import����а�����Զ�˴��룬go get������Զ���ȡ������
��install����װ����Щ���롣

```
$ go get github.com/golang/example/hello

$ $GOPATH/bin/hello
Hello, Go examples!
```
�����Ҫ�İ�����workspace�У�`go get`��������Щ������GOPATHָ���ĵ�һ��workspace�С�������Ѿ����ڣ���ôgo get

���������Զ�̻�ȡ��ֱ��ִ��`go install`���Ƶ����

��ִ�����ϵ�go get����֮��workspaceĿ¼���������������ģ�
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
GitHub���йܵ�hello������ͬһ�����е�stringutil��hello.go�е�import���ʹ�õ���һ���Ĺ��������go get����
���Զ�λ����װ��install����������
`import ��github.com/golang/example/stringutil`
��һ����������Ĵ���ǳ����ױ�����ʹ�á�

������������

Effective Go�и���Ĺ�����Ч��дGo��������ݡ�<br/>
A Tour of Go�и���Ĺ���Go��֪ʶ��
