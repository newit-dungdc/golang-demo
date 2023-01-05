# Tutorial: Get started with Go



**Cài đặt môi trường Go**

Link download https://go.dev/doc/install

cài xong chạy lệnh kiểm tra version
```
$ go version
```

go version go1.19.4 darwin/arm64


**Khởi tạo module mới : go mod init <ten_module>**
```
$ go mod init dungdc/demo
```
go: creating new go.mod: module dungdc/demo

Import các package đã dùng vào module trên
```
$ go mod tidy
```
go: finding module for package rsc.io/quote (tìm thấy package cho module)

go: found rsc.io/quote in rsc.io/quote v1.5.2

go: downloading rsc.io/sampler v1.3.0


**Tạo file hello.go**
```
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```
**Chạy chương trình**
```
$ go run . 
```
Hello, World!

Command xem các thông tin giúp đỡ thêm:
```
$ go help
```

**add 1 package vào module: go get <ten_package>**
```
$ go get rsc.io/quote
```
go: added golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c

go: added rsc.io/quote v1.5.2

go: added rsc.io/sampler v1.3.0
