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
**Ở đây là chưa xuất bản module example.com/greetings nên cần chỉnh sửa để module Hello gọi được module Greetings ở thư mục local**

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

# (lesson-2) Return and handle an error

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

# (lesson-2) Return a random greeting

**Sử dụng Go slice để trả về ngẫu nhiên 1 chuỗi từ mảng cho trước**

Thay đổi file greetings/greetings.go như sau

```
package greetings

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    message := fmt.Sprintf(randomFormat(), name)
    return message, nil
}

// init sets initial values for variables used in the function.
func init() {
    rand.Seed(time.Now().UnixNano())
}

// randomFormat returns one of a set of greeting messages. The returned
// message is selected at random.
func randomFormat() string {
    // A slice of message formats.
    formats := []string{
        "Hi, %v. Welcome!",
        "Great to see you, %v!",
        "Hail, %v! Well met!",
    }

    // Return a randomly selected message format by specifying
    // a random index for the slice of formats.
    return formats[rand.Intn(len(formats))]
}
```

**Trong file hello/hello.go truyền tên vào hàm**

Code sẽ như sau: 

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
    message, err := greetings.Hello("Gladys")
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

**Đứng ở thư mục hello chạy chương trình**

```
$ go run .
Great to see you, Gladys!
```

```
$ go run .
Hi, Gladys. Welcome!
```

```
$ go run .
Hail, Gladys! Well met!
```

# Return greetings for multiple people

**Sửa file greetings/greetings.go**

```
package greetings

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    message := fmt.Sprintf(randomFormat(), name)
    return message, nil
}

// Hellos returns a map that associates each of the named people
// with a greeting message.
func Hellos(names []string) (map[string]string, error) {
    // A map to associate names with messages.
    messages := make(map[string]string)
    // Loop through the received slice of names, calling
    // the Hello function to get a message for each name.
    for _, name := range names {
        message, err := Hello(name)
        if err != nil {
            return nil, err
        }
        // In the map, associate the retrieved message with
        // the name.
        messages[name] = message
    }
    return messages, nil
}

// Init sets initial values for variables used in the function.
func init() {
    rand.Seed(time.Now().UnixNano())
}

// randomFormat returns one of a set of greeting messages. The returned
// message is selected at random.
func randomFormat() string {
    // A slice of message formats.
    formats := []string{
        "Hi, %v. Welcome!",
        "Great to see you, %v!",
        "Hail, %v! Well met!",
    }

    // Return one of the message formats selected at random.
    return formats[rand.Intn(len(formats))]
}
```

**Sửa file hello.go**

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

    // A slice of names.
    names := []string{"Gladys", "Samantha", "Darrin"}

    // Request greeting messages for the names.
    messages, err := greetings.Hellos(names)
    if err != nil {
        log.Fatal(err)
    }
    // If no error was returned, print the returned map of
    // messages to the console.
    fmt.Println(messages)
}
```

**Đứng ở thư mục hello chạy**

```
$ go run .
```

# Add a test

**Thêm test với hàm Hello**

**Ở thư mục greetings tạo file greetings_test.go**

```
package greetings

import (
    "testing"
    "regexp"
)

// TestHelloName calls greetings.Hello with a name, checking
// for a valid return value.
func TestHelloName(t *testing.T) {
    name := "Gladys"
    want := regexp.MustCompile(`\b`+name+`\b`)
    msg, err := Hello("Gladys")
    if !want.MatchString(msg) || err != nil {
        t.Fatalf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
    }
}

// TestHelloEmpty calls greetings.Hello with an empty string,
// checking for an error.
func TestHelloEmpty(t *testing.T) {
    msg, err := Hello("")
    if msg != "" || err == nil {
        t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
    }
}
```

**Đứng ở thư mục greetings chạy lệnh go test**

thêm hậu tố -v để xem các bước kết quả test

```
$ go test
PASS
ok      example.com/greetings   0.467s
$ go test -v
=== RUN   TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok      example.com/greetings   0.086s
```

**Thay đổi code để có kết quả test FAIL**

Sửa code file greetings/greetings.go ở hàm Hello

```
// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    // message := fmt.Sprintf(randomFormat(), name)
    message := fmt.Sprint(randomFormat())
    return message, nil
}
```

**Chạy test lại để thấy kết quả**

```
$ go test
--- FAIL: TestHelloName (0.00s)
    greetings_test.go:15: Hello("Gladys") = "Hail, %v! Well met!", <nil>, want match for `\bGladys\b`, nil
FAIL
exit status 1
FAIL    example.com/greetings   0.446s
```

# Compile and install the application

**Lệnh go build biên dịch gói cùng các dependencies nhưng không cài đặt kết quả**

**Lệnh go install biên dịch và cài các gói**

**Chạy lệnh để biên dịch và cài ứng dụng**

```
$ go build
```

**Chạy file hello vừa được tạo và thấy kết quả**

```
$ ./hello
map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
```

Hiện tại chạy chương trình cần chỉ định đường dẫn, chúng ta sẽ cài đặt để chương trình chạy không cần chỉ định đường dẫn

**Get đường dẫn sử dụng lệnh go để cài đặt chương trình**

```
$ go list -f '{{.Target}}'
```

Đường dẫn trả về ví dụ là /Users/dungdc-newit/go/bin/hello

Thư mục chạy cài đặt go là /Users/dungdc-newit/go

Thư mục bin tạo file và để chạy chương trình là /Users/dungdc-newit/go/bin

**Thêm đường dẫn cài đặt go vào môi trường chạy chương trình export PATH=$PATH:/path/to/your/install/directory**

```
$ export PATH=$PATH:/Users/dungdc-newit/go
```

**Nếu đã có thư mục bin kiểu như $HOME/bin ở môi trường chạy, bạn muốn cài đặt để chạy chương trình Go ở đó thì set biến GOBIN : go env -w GOBIN=/path/to/your/bin**

```
$ go env -w GOBIN=/Users/dungdc-newit/go/bin
```

**Chạy lệnh biên dịch và cài đặt gói**

```
$ go install
```

Trong thư mục bin sẽ tạo ra file hello

**Chuyển sang 1 thư mục khác và chạy lệnh hello để thấy kết quả chương trình (không phụ thuộc vào đường dẫn chỉ định)**

```
$ hello
map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hi, Samantha. Welcome!]
```

# lesson-3 Getting started with multi-module workspaces

**Điều kiện tiên quyết**

- Go phiên bản 1.18 hoặc mới hơn
- Editor sửa code ví dụ https://code.visualstudio.com/
- Terminal chạy lệnh (Terminal Linux và Mac, Cmd Windows)

**Tạo 1 module**

Tạo thư mục và vào thư mục workspace

```
$ mkdir workspace
$ cd workspace
```

**Tạo module hello**

```
$ mkdir hello
$ cd hello
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
```

**Add gói golang.org/x/example vào module hello**

```
$ go get golang.org/x/example
```

**Tạo file hello.go với nội dung**

```
package main

import (
    "fmt"

    "golang.org/x/example/stringutil"
)

func main() {
    fmt.Println(stringutil.Reverse("Hello"))
}
```

**Chạy chương trình trên bằng lệnh**

```
$ go run example.com/hello
olleH
```

**Tạo workspace : trong thư mục workspace tạo file go.work bằng lệnh**

```
$ go work init ./hello
```

File go.work sẽ có nội dung kiểu như:

```
go 1.19

use ./hello
```

File go.work có cú pháp tương tự file go.mod

**Chạy chương trình tại thư mục workspace**

```
$ go run example.com/hello
olleH
```

**Download 1 module từ online về local để sử dụng : golang.org/x/example**
**Đứng tại thư mục workspace chạy lệnh**

```
$ git clone https://go.googlesource.com/example
Cloning into 'example'...
remote: Total 204 (delta 93), reused 204 (delta 93)
Receiving objects: 100% (204/204), 103.24 KiB | 839.00 KiB/s, done.
Resolving deltas: 100% (93/93), done.
```

**Add module này vào workspace**

```
$ go work use ./example
```

**File go.work nhìn sẽ như thế này**

```
go 1.19

use (
	./example
	./hello
)
```

**Tạo 1 hàm mới chuyển chuỗi in hoa từ package golang.org/x/example/stringutil**

**Tạo file toupper.go ở thư mục workspace/example/stringutil với nội dung**

```
package stringutil

import "unicode"

// ToUpper uppercases all the runes in its argument string.
func ToUpper(s string) string {
    r := []rune(s)
    for i := range r {
        r[i] = unicode.ToUpper(r[i])
    }
    return string(r)
}
```

**Sửa file hello.go ở thư mục workspace/hello để sử dụng package: workspace/example/stringutil**

```
package main

import (
    "fmt"

    "golang.org/x/example/stringutil"
)

func main() {
    fmt.Println(stringutil.ToUpper("Hello"))
}
```

**Chạy chương trình module hello đứng ở thư mục workspace**

```
$ go run example.com/hello
HELLO
```

**Chú ý về phát hành module**

Để phát hành module hello thì cần phải phát hành module golang.org/x/example (ví dụ version V0.1.0)

Tại thư mục hello get module golang.org/x/example

```
cd hello
go get golang.org/x/example@v0.1.0
```