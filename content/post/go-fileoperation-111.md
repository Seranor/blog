---
title: golang文件操作
lastmod: 2025-05-09T16:43:23+08:00
date: 2025-05-09T17:52:03+08:00
tags:
  - Golang
categories:
  - Golang
url: post/go-fileoperation.html
toc: true
---

## 文件操作

### 写文件

io/write_file.go

```go
package io

import (
	"fmt"
	"os"
)

func WriteFile() {
    // 如果使用go test 则相对路径是相对于xxx_test.go 文件的路径
    // 如果使用go run 或者编辑后直接运行，则相对路径是相对于执行命令所在的路径
	if fout, err := os.OpenFile("verse.txt", os.O_CREATE|os.O_TRUNC|os.O_WRONLY, 0o666); err != nil {
		fmt.Printf("open file failed: %s\n", err.Error())
	} else {
		defer fout.Close()
		fout.WriteString("纳兰性德\n")
		fout.WriteString("明月多情应笑我")
		fout.WriteString("\n")
		fout.WriteString("笑我如今")
		fout.WriteString("\n")
	}
}
/*
os.O_CREATE  创建文件 如果删除，文件不存在就保存
os.O_TRUNC   清空写入
os.O_APPEND  追加写入
os.O_WRONLY  只写操作
0o666 权限
*/
```

io/io_test.go

```go
package io_test

import (
	"goPractice/io"
	"testing"
)

func TestWriteFile(t *testing.T) {
	io.WriteFile()
}
```

运行测试

```shell
go test -v ./io -run=^TestWriteFile$
```

verse.txt

```
纳兰性德
明月多情应笑我
笑我如今

```

### 读文件

io/read_file.go

```go
package io

import (
	"fmt"
	"io"
	"os"
)

func ReadFile() {
	if fin, err := os.Open("verse.txt"); err != nil {
		fmt.Printf("open file faild: %s\n", err.Error())
	} else {
		defer fin.Close()
		bs := make([]byte, 200)
		fin.Read(bs)
		fmt.Println(string(bs))

		// 一个汉字占三个字节
		fin.Seek(0, 0) // 跳过前10个字节，从起始位置开始
		fin.Read(bs)
		fmt.Println(string(bs))

		fin.Seek(0, 0) // 重新回到起始位置
		const BATCH = 10
		bufer := make([]byte, BATCH)
		for {
			n, err := fin.Read(bufer)
			if n > 0 {
				fmt.Println(bufer[0:n])
			}
			if err == io.EOF {
				break
			}
		}
	}
}
```

io/io_test.go

```go
package io_test

import (
	"goPractice/io"
	"testing"
)

func TestReadFile(t *testing.T) {
	io.ReadFile()
}
```

运行测试

```shell
go test -v ./io -run=^TestReadFile$
```

### 使用bufio读写文件

#### 写文件

```go
package io

import (
	"bufio"
	"fmt"
	"os"
)

func WriteFileWithBuffer() {
	if fout, err := os.OpenFile("verse.txt", os.O_CREATE|os.O_TRUNC|os.O_WRONLY, 0o666); err != nil {
		fmt.Printf("open file failed: %s\n", err.Error())
	} else {
		defer fout.Close()
		writer := bufio.NewWriter(fout)
		writer.WriteString("纳兰性德\n")
		writer.Write([]byte("明月多情应笑我\n"))
		writer.WriteString("笑我如今\n")
		writer.Flush() // 写入文件
	}
}

```

#### 读文件

```go
package io

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"strings"
)

func ReadFileWithBuffer() {
	if fin, err := os.Open("verse.txt"); err != nil {
		fmt.Printf("open file faild: %v\n", err)
	} else {
		defer fin.Close()
		reader := bufio.NewReader(fin)
		for {
			line, err := reader.ReadString('\n')
			if len(line) > 0 {
				line = strings.TrimRight(line, "\n") // 去掉多余的换行符
				fmt.Println(line)
			}
			if err == io.EOF { // End Of File
				break
			}
		}

	}
}

```

test

```go
func TestReadFileWithBuffer(t *testing.T) {
	io.ReadFileWithBuffer()
}

func TestWriteFileWithBuffer(t *testing.T) {
	io.WriteFileWithBuffer()
}
```

```shell

go test -v ./io -run=^TestReadFileWithBuffer$
go test -v ./io -run=^TestWriteFileWithBuffer$
```

### 文件管理和目录遍历

```go
package io

import (
	"fmt"
	"io/fs"
	"os"
	"path/filepath"
)

func CreateFile(fileName string) {
	os.Remove(fileName) // 操作系统负责管理文件，相关操作用os  读写流用io
	if file, err := os.Create(fileName); err != nil {
		fmt.Printf("create file failed: %v\n", err)
	} else {
		defer file.Close()
		file.Chmod(0o666)
		fmt.Printf("fd=%d\n", file.Fd()) // FD是有上限的，I/O流不用了要及时关闭包括文件和网络IO
		file.WriteString("多情应笑我\n")
		info, _ := file.Stat()
		fmt.Printf("is dir %t\n", info.IsDir())
		fmt.Printf("modify time %s\n", info.ModTime())
		fmt.Printf("mode %v\n", info.Mode())
		fmt.Printf("file name %s\n", info.Name())
		fmt.Printf("size  %dB\n", info.Size())
	}

	os.Mkdir("../data/sys", os.ModePerm)
	os.MkdirAll("../data/sys/a/b/c", os.ModePerm)
	os.Rename("../data/sys/a", "../data/sys/p")
	os.Rename("../data/sys/p/b/c", "../data/sys/p/c")

	// os.Remove("../data/sys")    // 必须是一个空目录
	// os.RemoveAll("../data/sys") // 可以删除非空目录
}

// 遍历一个目录
func WalkDir(path string) error {
	filepath.Walk(path, func(subpath string, info fs.FileInfo, err error) error {
		if err != nil {
			return err
		}
		if info.Mode().IsDir() && subpath != path {
			fmt.Printf("path is dir %s\n", subpath)
		} else if info.Mode().IsRegular() {
			fmt.Printf("path is file %s basename %s\n", subpath, info.Name())
		}
		return nil
	})
	return nil
}
```



