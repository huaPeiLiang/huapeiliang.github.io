---
title: Golang - Hello, world
description: 本篇文章将介绍：下载并安装 Go、编写 Hello World、Go 编辑器
categories:
- Golang
---

> 路漫漫其修远兮，吾将上下而求索。

## 下载并安装 Go

到官网下载 Go：https://golang.google.cn/doc/install

安装完毕后在命令行输入下面命令查看是否安装成功：
```
$ go version
```

## 编写 Hello World

以 mac 为例：

1、进入个人目录下
```
$ cd ～
```

2、创建 GoProjects 文件夹并进入
```
$ mkdir GoProjects
$ cd GoProjects
```

3、创建 hello 文件🏠并进入
```
$ mkdir hello
$ cd hello
```

4、创建 go.mod 文件来为代码启用依赖关系跟踪
```
$ go mod init example/GoProjects/hello
```
出现下面提示表示成功：
```
go: creating new go.mod: module example/GoProjects/hello
```

5、创建 hello.go 文件，编写 Hello World 代码
```
package main

import "fmt"

func main(){
    fmt.Println("Hello, World!")
}
```

6、运行
```
$ go run .
```

## Go 编辑器

- VSCode
- GoLand
- Vim

## VSCode 无法下载 Go 扩展

VSCode 下载 Go 扩展发现无法下载的问题，即使科学上网依旧不能解决问题。谷歌之后发现这可能是它的一个bug。解决办法：

1、终端输入 go env 查看 GOPATH 路径。
```
$ go env
```
2、在 GOPATH 路径下创建 golang.org/x 目录
```
$ cd GOPATH
$ mkdir -p golang.org/x
```
3、进入 golang.org/x 目录，克隆 https://github.com/golang/tools.git，最好是科学上网后克隆
```
$ cd golang.org/x
$ git clone https://github.com/golang/tools.git tools
```
4、克隆 git clone https://github.com/golang/lint.git
```
$ git clone git clone https://github.com/golang/lint.git
```
5、打开 VSCode 重新 install

成功后打开之前编写的 hello 文件夹，执行 go run . 命令，打印出 "Hello, World!" 表示成功。









