# Lesson 1 : Bắt đầu với Go

**Cài đặt môi trường Go**

Link download https://go.dev/doc/install

Cài xong chạy lệnh kiểm tra version
```
$ go version
go version go1.19.4 darwin/arm64
```



**Khởi tạo module mới : go mod init <ten_module>**
```
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
```

Import các package đã dùng vào module trên
```
$ go mod tidy
go: finding module for package rsc.io/quote (tìm thấy package cho module)
go: found rsc.io/quote in rsc.io/quote v1.5.2
go: downloading rsc.io/sampler v1.3.0
```

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
Hello, World!
```


Command xem các thông tin giúp đỡ thêm:
```
$ go help
```

**add 1 package vào module: go get <ten_package>**

Add package có tên rsc.io/quote
```
$ go get rsc.io/quote
go: added golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
go: added rsc.io/quote v1.5.2
go: added rsc.io/sampler v1.3.0
```

**Sửa lại file hello.go để gọi function của package**
```
package main

import "fmt"
import "rsc.io/quote"

func main() {
    fmt.Println("Hello, World!")
    fmt.Println(quote.Go()) // sử dụng func Go của package
}
```

Chạy để kiểm tra lại kết quả :
```
Hello, World!
Don't communicate by sharing memory, share memory by communicating.
```



# Lesson 2 : Tạo module và sử dụng module

**Tạo thư mục mới greetings**
```
mkdir greetings
cd greetings
```

**Tạo module mới**
khởi tạo : go mod init <ten_module>
```
$ go mod init example.com/greetings
go: creating new go.mod: module example.com/greetings
```

**Tạo file greetings.go**
```
package greetings

import "fmt"

// Hello returns a greeting for the named person.
func Hello(name string) string {
    // Return a greeting that embeds the name in a message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message
}
```

![Giải thích về hàm trên](https://go.dev/doc/tutorial/images/function-syntax.png)


#Cách gọi code từ module khác

**Giả sử chúng ta có thư mục như sau có 2 module greetings và hello**
```
<home>/
 |-- greetings/
 |-- hello/
 ```

 **Tạo module trong thư mục hello**  
 ```
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
 ```
 
 **Import module greetings vào module hello**
 Tạo file hello.go như sau
 ```
package main

import (
    "fmt"

    "example.com/greetings"
)

func main() {
    // Get a greeting message and print it.
    message := greetings.Hello("Gladys")
    fmt.Println(message)
}
 ```
**Ở đây là chưa xuất bản module example.com/greetings nên cần chỉnh sửa để module Hello gọi được module Greetings ở thư mục local **

Đứng tại thư mục hello bạn chạy lệnh
```
go mod edit -replace example.com/greetings=../greetings
```

Ở file go.mod của thư mục hello tự thay đổi như này là ok
```
module example.com/hello

go 1.16

replace example.com/greetings => ../greetings
```

**Chạy động bộ các dependency ở thự mục hello**
```
$ go mod tidy
go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
```

File go.mod thay đổi thế này là ok
```
module example.com/hello

go 1.16

replace example.com/greetings => ../greetings

require example.com/greetings v0.0.0-00010101000000-000000000000
```

**Chạy để ra kết quả**
```
go run .
Hi, Gladys. Welcome!
```

Như vậy là đã tạo và gọi module khác thành công!

#Return and handle an error (Folder lesson-2)

**Sửa code file greetings/greetings.go như sau**

Trả về 1 chuỗi và 1 lỗi

```
package greetings

import (
    "errors"
    "fmt"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return "", errors.New("empty name")
    }

    // If a name was received, return a value that embeds the name
    // in a greeting message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message, nil
}
```

**Sửa file hello/hello.go để xử lý lỗi từ greetings và để không bị lỗi**

```
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // Request a greeting message.
    message, err := greetings.Hello("")
    // If an error was returned, print it to the console and
    // exit the program.
    if err != nil {
        log.Fatal(err)
    }

    // If no error was returned, print the returned message
    // to the console.
    fmt.Println(message)
}
```

Trong thư mục hello chạy lệnh ra kết quả

```
$ go run .
greetings: empty name
```