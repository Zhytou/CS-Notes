# 包 & 工具

- [包 \& 工具](#包--工具)
  - [工具](#工具)
  - [包](#包)
    - [包的导入声明](#包的导入声明)
    - [包的组织结构](#包的组织结构)
    - [包的初始化](#包的初始化)
  - [包管理](#包管理)
    - [传统的包管理](#传统的包管理)
    - [go mod管理](#go-mod管理)

## 工具

Go语言的工具箱集合了一系列的功能的命令集。它可以看作是一个包管理器（类似于Linux中的apt和rpm工具），用于包的查询、计算的包依赖关系、从远程版本控制系统和下载它们等任务。

此外，它也是一个构建系统，计算文件的依赖关系，然后调用编译器、汇编器和连接器构建程序。同时，它也是一个单元测试和基准测试的驱动程序。

**基本命令**：

``` bash
# compile packages and dependencies
go build
# remove object files
go clean 
# show documentation for package or symbol
go doc 
#  print Go environment information
go env
# run gofmt on package sources
go fmt
# download and install packages and dependencies
go get
# compile and install packages and dependencies
go install
# list packages
go list
# compile and run Go program
go run
# test packages
go test
# print Go version
go version
# run go tool vet on packages
go vet 
```

## 包

Go 是一个包类语言。Go代码可以被组织成多个模块或包（Package），每个模块或包包含一组相关的类、函数、变量等。这种模块化的组织方式可以提高代码的可维护性、可复用性和可扩展性。

一个包可能由多个文件组成。组成它的文件往往放在同一个目录下，且按习惯包名和目录名一般相同。

### 包的导入声明

每个导入声明可以单独指定一个导入路径，也可以通过圆括号同时导入多个导入路径。

``` golang
import "fmt"
import "time"

import {
    "fmt"
    "time"
}
```

**包的匿名导入**：

如果只是导入一个包而并不使用导入的包将会导致一个编译错误。

但是有时候我们只是想利用导入包而产生的副作用：它会计算包级变量的初始化表达式和执行导入包的init初始化函数。此时，我们可以用我下划线 _ 来重命名导入的包。

**包重命名**：

如果我们想同时导入两个有着名字相同的包，例如math/rand包和crypto/rand包，那么导入声明必须至少为一个同名包指定一个新的包名以避免冲突。这叫做导入包的重命名。

``` go
import {
    "crypto/rand"
    mrand "math/rand"
}
```

### 包的组织结构

包由一个或多个保存在同一目录下（不含子目录）的源码文件组成。包的用途类似命名空间，是成员的作用域和访问权限边界。所有成员在包内均可访问，无论是否在同一源码文件中。

虽说包名与所在目录名无关，但习惯上保持一致。

### 包的初始化

**init函数**：

init函数的作用是完成包的初始化工作，例如初始化包级别的变量、执行一些初始化操作等等。

init函数有以下几个特点：

- init函数没有参数和返回值。
- init函数不能被显式调用，而是在包被导入时自动执行。
- 如果一个包定义了多个init函数，它们的执行顺序是不确定的。

**初始化顺序**：

- 首先，初始化包级别的变量和常量。这些变量和常量的初始化按照它们在程序中出现的顺序进行。
- 接着，执行包级别的init函数。每个包可以定义多个init函数，这些函数会在包被导入时自动执行。init函数的执行顺序与它们在程序中的顺序无关，而是按照它们在包中出现的顺序依次执行。
- 最后，如果该包依赖其他包，则会先初始化依赖的包，再按照上述步骤依次初始化当前包。

## 包管理

### 传统的包管理

**go工作区结构**：

在配置Go的路径时，一般需要配置以下几个路径：

- GOROOT：Go的安装路径，它指向Go语言的安装目录。在GOROOT下包含了Go语言的标准库和工具等。
- GOPATH：Go的工作路径，它是用户自定义的路径，用于存储Go项目的源代码、二进制文件和依赖库等。可以有多个GOPATH路径，每个路径之间使用冒号（Windows上是分号）进行分隔。
- GOBIN：Go二进制文件的安装路径，它指向用户自定义的路径，用于存储由Go源代码编译而成的可执行文件。

**下载包**：

使用命令`go get`可以下载一个单一的包或者用`...`下载整个子目录里面的每个包

**编译包**：

`go build`命令编译命令行参数指定的每个包。

- 如果包是一个库，则忽略输出结果；这可以用于检测包的可以正确编译的。
- 如果包的名字是main，`go build`将调用连接器在当前目录创建一个可执行程序；以导入路径的最后一段作为可执行程序的名字。

### go mod管理

Golang 1.11 版本引入的 go mod ，其思想类似maven：摒弃vendor和GOPATH，拥抱本地库。从 Go 1.11 开始，Go 允许在 $GOPATH/src 外的任何目录下使用 go.mod 创建项目。

**go.mod的优势**：

具体来说，当您编译和运行程序时，Go 语言会检查代码中所使用的所有依赖项，如果某个依赖项不在 go.mod 文件中，Go 语言会自动将其添加到 go.mod 文件中，并使用指定的版本号下载相应的依赖项。

换句话说，go.mod文件一旦创建后，它的内容将会被go toolchain全面掌控。go toolchain会在各类命令执行时，比如go get、go build、go mod等修改和维护go.mod文件。

**go.mod文件标签**：

go.mod 提供了module, require、replace和exclude 四个命令

- module  语句指定包的名字（路径）
- require 语句指定的依赖项模块
- replace 语句可以替换依赖项模块
- exclude 语句可以忽略依赖项模块

**常用go mod命令**：

``` bash
// 在项目根目录下创建go.mod文件，开始自动追踪并更新所需的包
go mod init [项目名称]

// 更新依赖包版本
go get -u

// 增加丢失的依赖，删除不需要的依赖
go mod tidy
```

**设置go代理**：

通过 go get 命令下载各种依赖时，经常需要访问 github/google 等网站，如果使用默认的代理，下载速度很慢，甚至无法下载，此时就需要设置国内代理以实现高效下载.

``` bash
go env -w GOPROXY=https://goproxy.cn,direct
```
