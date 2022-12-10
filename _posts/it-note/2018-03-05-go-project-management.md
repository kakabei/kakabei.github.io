---
layout: post
title: go 笔记 工程管理
date: 2018-03-05 07:21:12
categories:  go 
tags: go
excerpt: go 对项目一些管理方式
---

介绍 Go 语言所引入的工程管理思想、工具和规范：

- 代码风格
- 文档风格和管理
- 单元测试与性能测试方法
- 项目工程结构
- 打包分发

# 一、go 命令行工具

查看版本:

```sh 
> go version
go version go1.19.3 darwin/amd64
```

查看帮助：

```sh
> go help
Go is a tool for managing Go source code.

Usage:

	go <command> [arguments]

The commands are:

	bug         start a bug report
	build       compile packages and dependencies
	clean       remove object files and cached files
	doc         show documentation for package or symbol
	env         print Go environment information
	fix         update packages to use new APIs
	fmt         gofmt (reformat) package sources
	generate    generate Go files by processing source
	get         add dependencies to current module and install them
	install     compile and install packages and dependencies
	list        list packages or modules
	mod         module maintenance
	work        workspace maintenance
	run         compile and run Go program
	test        test packages
	tool        run specified go tool
	version     print Go version
	vet         report likely mistakes in packages

Use "go help <command>" for more information about a command.

Additional help topics:

	buildconstraint build constraints
	buildmode       build modes
	c               calling between Go and C
	cache           build and test caching
	environment     environment variables
	filetype        file types
	go.mod          the go.mod file
	gopath          GOPATH environment variable
	gopath-get      legacy GOPATH go get
	goproxy         module proxy protocol
	importpath      import path syntax
	modules         modules, module versions, and more
	module-get      module-aware go get
	module-auth     module authentication using go.sum
	packages        package lists and patterns
	private         configuration for downloading non-public code
	testflag        testing flags
	testfunc        testing functions
	vcs             controlling version control with GOVCS

Use "go help <topic>" for more information about that topic.
```
`Go tool` 可以完成以下几类工作：

- 代码格式化
- 代码质量分析和修复
- 工程构建
- 代码文档的提取和展示
- 依赖包管理
- 执行其他的包含指令，比如6g等

# 二、代码风格

Go 语言很可能是第一个将代码风格强制统一的语言。

Go 语言的编码规范，主要分两类：1） 由Go编译器进行强制的编码规范。2） 由 `Go tool` 推行的非强制性编码风格建议。

## 2.1 强制性编码规范

###  命名

命名规则涉及变量、常量、全局函数、结构、接口、方法等的命名。Go 语言从语法层面进行了以下限定:任何需要对外暴露的名字必须以大写字母开头，不需要对外暴露的则应该以小写 字母开头。

### 排列

Go 语言甚至对代码的排列方式也进行了语法级别的检查，约定了代码块中花括号的明确摆放位置。

```go
// 错误写法
func Foo(a, b int)(ret int, err error)
{
    if a > b
    {
        ret = a
    }
    else
    {
        ret = b
    }
}

// 正确写法
func Foo(a, b int)(ret int, err error) {
    if a > b { 
        ret = a
    } else { 
        ret = b
    }
}
```

## 2.2 非强制性编码风格建议

用 `go tool` 命令看格式化工具的用法。

```sh
go help fmt
usage: go fmt [-n] [-x] [packages]

Fmt runs the command 'gofmt -l -w' on the packages named
by the import paths. It prints the names of the files that are modified.

For more about gofmt, see 'go doc cmd/gofmt'.
For more about specifying packages, see 'go help packages'.

The -n flag prints commands that would be executed.
The -x flag prints commands as they are executed.

The -mod flag's value sets which module download mode
to use: readonly or vendor. See 'go help modules' for more.

To run gofmt with specific options, run gofmt itself.

See also: go fix, go vet.
```
代码风格很差的代码：

```go 
package main import "fmt"
func Foo(a, b int)(ret int, err error){ if a > b {
return a, nil
}else{ return b, nil
}
return 0, nil
}
func
main() { i, _ := Foo(1, 2) fmt.Println("Hello, 世界", i)}
```
执行命令之后

```go
$ go fmt hello.go
```

```go 
package main
import "fmt"

func Foo(a, b int) (ret int, err error) { 
    if a > b {
        return a, nil 
    } else {
        return b, nil 
    }
    return 0, nil 
}

func main() {
    i, _ := Foo(1, 2) fmt.Println("Hello, 世界", i)
}
```


# 三、远程 import 支持

Go 语言不仅允许我们导入本地包，还支持在语言级别调用远程的包。

```go 
package main
import ( 
    "fmt"
    "github.com/myteam/exp/crc32"
)
```
`"github.com/myteam/exp/crc32"` 就是远程的包

然后，在执行`go build`或者`go install`之前，只需要加这么一句: 

```sh
go get github.com/myteam/exp/crc32
```

# 四、工程组织

## 4.1 GOPATH

假设现在本地硬盘上有3个Go代码工程，分别为`~/work/go-proj1`、`~/work2/goproj2`和 `~/work3/work4/go-proj3`，那么`GOPATH` 可以设置为如下内容:
```sh
export GOPATH=~/work/go-proj1:~/work2/goproj2:~/work3/work4/go-proj3

```

经过这样的设置后，你可以在任意位置对以上的3个工程进行构建。

## 4.2 目录结构

```sh
<calcproj> 
    ├─README
    ├─AUTHORS 
        ├─<bin>
            ├─calc 
    ├─<pkg>
        └─<linux_amd64> 
            └─simplemath.a
    ├─<src> 
        ├─<calc>
            └─calc.go
        ├─<simplemath>
            ├─add.go 
            ├─add_test.go 
            ├─sqrt.go
            ├─sqrt_test.go
```
Go 语言工程不需要任何工程文件，一个比较完整的工程会在根目录处放置这样几个文本文件。

- README : 简单介绍本项目目标和关键的注意事项，通常第一次使用时应该先阅读本文档。
- LICENSE : 本工程采用的分发协议，所有开源项目通常都有这个文件。


# 五、 单元测试

Go 本身提供了一套轻量级的测试框架。符合规则的测试代码会在运行测试时被自动识别并执行。单元测试源文件的命名规则如下:在需要测试的包下面创建以“`_test`”结尾的 go 文件，形如 : `[^.]*_test.go`。

Go 的单元测试函数分为两类 : 功能测试函数和性能测试函数，分别为以 `Test` 和 `Benchmark` 为函数名前缀并以 `*testing.T` 为单一参数的函数。

```go
func TestAdd1(t *testing.T) { 
	r := Add(1, 2)
	if r != 2 { // 这里本该是3，故意改成2测试错误场景 
		t.Errorf("Add(1, 2) failed. Got %d, expected 3.", r)
	}
}
```

执行 `go test` 命令。

```go
func BenchmarkAdd1(b *testing.B) { 
	for i := 0; i < b.N; i++ {
		Add(1, 2) 
	}
}
```
执行 `go -test.bench` 命令。 

# 六、打包分发

GO 以二进制方式分发Go包并不是很现实。最合适的库分发方式是直接打包源代码包并进行分发，由使用者自行编译。


