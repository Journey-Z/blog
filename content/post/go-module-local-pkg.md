---
title: "Go Module 引入本地包"
date: 2019-11-15T16:05:43+08:00
categories:
- Golang
tags:
- go
- go module
keywords:
- go module
thumbnailImagePosition: left
thumbnailImage: img/gopher10th-small.jpg

---

项目在 $GOPATH 之外，如何通过 go module 将本地的自定义包引入到项目中使用...
<!--more-->

#### Go Module 引入本地包

本地包替换

举个例子，假如本地有项目是建立在 $GOPATH 之外，有如下结构：

* * *

```
cd mygo.localpkg

mygo.localpkg
├── mypuls
    ├── plus.go
├── main.go
```

main.go 负责调用 /mygo.localpkg/myplus/plus.go 中的 Plus 方法，将两个整数相加， plus.go 属于 myplus 的包；

main.go:

```
package main

import (
	"fmt"
	"mygo.localpkg/myplus"
)

func main() {
	a,b := 3,6
	result := myplus.Plus(a, b)
	fmt.Printf("result is %d", result)
}

```

plus.go:

```
package myplus

func Plus (a,b int) int {
	return  a + b
}
```

那么 "mygo.localpkg/myplus" 这个 package 是怎么来的呢？下面通过 go module 去引入本地的自定义包：

```
cd mygo.localpkg

go mod init mygo.localpkg
```

执行完 go mod 命令后，会在该目录生成一个 go.mod 文件
我们稍微对 go mod 进行一下编辑，虽然不推荐直接编辑mod文件，但在这个例子中与使用go mod edit的效果几乎没有区别：

```
module mygo.localpkg

go 1.13

replace mygo.localpkg/myplus => ./pkg

```

在 go.mod 中添加：
```
replace mygo.localpkg/myplus => ./pkg
```
replace my/example/pkg => ./pkg这句，与替换远程包时一样，其实只是将替换用的包名改为了本地module所在的绝对或相对路径，此时执行：

```
go run main.go

result is 9
```

再回头看下 IDE 中的 plus.go 文件，我用的是 Goland，发现虽然代码成功执行了，但是 IDE 中的

```
import (
	"fmt"
	"mygo.localpkg/myplus"
)
```

其中 “mygo.localpkg/myplus” 还是处于红色的 Cannot resolve directory 'mygo.localpkg' 状态, 这样就无法通过 ide 进行方法的跳转，想想都会很崩溃，但是可以开启 Goland 的 Enable Go Modules (vgo) integration，这样 ide 就可以正常在本地包之间进行跳转了

