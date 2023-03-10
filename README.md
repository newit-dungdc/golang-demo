# Lesson 1 : Bắt đầu với Go

**Cài đặt môi trường Go**

Link download https://go.dev/doc/install

Cài xong chạy lệnh kiểm tra version
```
$ go version
go version go1.19.4 darwin/arm64
```

**Chú ý nếu chạy lệnh go không được báo lỗi zsh: command not found: go**

Set môi trường để chạy và check lại go version nếu OK ra version là đã chạy được

```
$ export PATH=$PATH:/usr/local/go/bin
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


# Lesson-4 Developing a RESTful API with Go and Gin

**Điều kiện tiên quyết**

- Cài đặt Go phiên bản 1.16 hoặc mới hơn
- Editor sửa code ví dụ https://code.visualstudio.com/
- Terminal chạy lệnh (Terminal Linux và Mac, Cmd Windows)
- Curl tool: Linux and Mac đã có sẵn, trên windows cài https://docs.microsoft.com/en-us/virtualization/community/team-blog/2017/20171219-tar-and-curl-come-to-windows

**Thiết kế API endpoint**

/albums

GET – Get danh sách tất cả các album, data trả về dạng JSON.
POST – Tạo mới 1 album, data trả về dạng JSON.

/albums/:id

GET – Get 1 album theo id, data trả về dạng JSON.

**Tạo thư mục code của bạn**

Tạo thư mục web-service-gin

```
$ mkdir web-service-gin
$ cd web-service-gin
```

**Tạo module example/web-service-gin đứng tại thư mục web-service-gin**

```
$ go mod init example/web-service-gin
go: creating new go.mod: module example/web-service-gin
```

**Tạo data cho chương trình**

Lưu ý data lưu vào bộ nhớ nên sẽ bị clear khi đóng chương trình

Tạo file main.go và chèn đoạn code vào

```
package main
```

Thêm cấu trúc cho album

```
// album represents data about a record album.
type album struct {
    ID     string  `json:"id"`
    Title  string  `json:"title"`
    Artist string  `json:"artist"`
    Price  float64 `json:"price"`
}
```

Set dữ liệu ban đầu

```
// albums slice to seed record album data.
var albums = []album{
    {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}
```

**API trả về danh sách tất cả items**

GET /albums trả về dạng json

đoạn code get danh sách albums viết dưới phần khai báo data ở trên

```
// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
    c.IndentedJSON(http.StatusOK, albums)
}
```

gin.Context mang thông tin chi tiết của request, validate và tuần tự hóa JSON

IndentedJSON mang cấu trúc json để có thể trả về dạng json


**xử lý cho đường dẫn endpoint**

```
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)

    router.Run("localhost:8080")
}
```

Khởi tạo route Gin là mặc định

Sử dụng hàm GET với method HTTP đường dẫn /albums

Ở file main.go thêm package vào phần đầu như sau

```
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)
```

**Add dependency github.com/gin-gonic/gin cho module**

```
$ go get .
go get: added github.com/gin-gonic/gin v1.7.2
```

add xong các lib cho dependency thì chạy code file main.go

```
$ go run .
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /albums                   --> main.getAlbums (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on localhost:8080
```

**Chạy curl để thử API**

```
$ curl http://localhost:8080/albums
[
    {
        "id": "1",
        "title": "Blue Train",
        "artist": "John Coltrane",
        "price": 56.99
    },
    {
        "id": "2",
        "title": "Jeru",
        "artist": "Gerry Mulligan",
        "price": 17.99
    },
    {
        "id": "3",
        "title": "Sarah Vaughan and Clifford Brown",
        "artist": "Sarah Vaughan",
        "price": 39.99
    }
]
```

**API tạo 1 item mới**

method POST đường dẫn /albums

viết hàm tạo item mới ở phần đầu của file

```
// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
    var newAlbum album

    // Call BindJSON to bind the received JSON to
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}
```

 Context.BindJSON lấy nội dung yêu cầu từ body request

 Nối item album vào danh sách albums

 Thêm mã 201 trả về, và trả về thông tin albums vừa được thêm

 **Ở hàm main thêm route.POST**

```
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}
```

**Chạy lại chương trình**

```
$ go run .
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /albums                   --> main.getAlbums (3 handlers)
[GIN-debug] POST   /albums                   --> main.postAlbums (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on localhost:8080
```

**Test thêm mới 1 album**

```
$ curl http://localhost:8080/albums \
    --include \
    --header "Content-Type: application/json" \
    --request "POST" \
    --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
```

JSON trả về

```
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Tue, 10 Jan 2023 04:43:24 GMT
Content-Length: 116

{
    "id": "4",
    "title": "The Modern Sound of Betty Carter",
    "artist": "Betty Carter",
    "price": 49.99
} 
```

**Get lại danh sách albums**

```
$ curl http://localhost:8080/albums \
    --header "Content-Type: application/json" \
    --request "GET"
```

Kết quả

```
[
        {
                "id": "1",
                "title": "Blue Train",
                "artist": "John Coltrane",
                "price": 56.99
        },
        {
                "id": "2",
                "title": "Jeru",
                "artist": "Gerry Mulligan",
                "price": 17.99
        },
        {
                "id": "3",
                "title": "Sarah Vaughan and Clifford Brown",
                "artist": "Sarah Vaughan",
                "price": 39.99
        },
        {
                "id": "4",
                "title": "The Modern Sound of Betty Carter",
                "artist": "Betty Carter",
                "price": 49.99
        }
]
```

**API trả về 1 item cụ thể**

Viết hàm get 1 item

```
// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
    id := c.Param("id")

    // Loop over the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```

Context.Param get param id từ URL

Lặp từ danh sách albums tìm album trùng id với param từ URL và trả về HTTP code 200

Nếu không có id trùng với param từ URL thì trả về HTTP 404  http.StatusNotFound

**Ở hàm main thêm route.GET album theo id**

```
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    router.GET("/albums/:id", getAlbumByID)
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}
```

**Chạy lại chương trình**

```
$ go run .
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /albums                   --> main.getAlbums (3 handlers)
[GIN-debug] GET    /albums/:id               --> main.getAlbumByID (3 handlers)
[GIN-debug] POST   /albums                   --> main.postAlbums (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on localhost:8080
```

**Curl thử 1 item**

```
$ curl http://localhost:8080/albums/2
```

kết quả trả về

```
{
    "id": "2",
    "title": "Jeru",
    "artist": "Gerry Mulligan",
    "price": 17.99
}
```

Thử với id không có trong albums

```
$ curl http://localhost:8080/albums/10
```

thấy kết quả 

```
{
    "message": "album not found"
}
```

**Code đầy đủ của file main.go**

```
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

// album represents data about a record album.
type album struct {
    ID     string  `json:"id"`
    Title  string  `json:"title"`
    Artist string  `json:"artist"`
    Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
    {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    router.GET("/albums/:id", getAlbumByID)
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}

// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
    c.IndentedJSON(http.StatusOK, albums)
}

// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
    var newAlbum album

    // Call BindJSON to bind the received JSON to
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}

// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
    id := c.Param("id")

    // Loop through the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```

# Lesson-5 Getting started with generics

Generics định nghĩa các hàm có kiểu dữ liệu dạng chung chung, không chỉ định rõ là kiểu gì. Khi sử dụng hàm sẽ set kiểu dữ liệu là gì.

**Điều kiện tiên quyết**

- Go phiên bản 1.18 hoặc mới hơn
- Editor sửa code ví dụ https://code.visualstudio.com/
- Terminal chạy lệnh (Terminal Linux và Mac, Cmd Windows)

**Tạo và vào thư mục generics**

```
$ mkdir generics
$ cd generics
```

**Tạo module example/generics**

```
$ go mod init example/generics
go: creating new go.mod: module example/generics
```

**Tạo hàm non-generic**

Trong thư mục tạo file main.go và đưa đoạn code vào khai báo package

Ghi chú : Đoạn code dưới có sử dụng map -> Map (bản đồ) là một kiểu dữ liệu được dựng sẵn trong Go, một map là tập hợp các cặp key/value (khóa/giá trị) trong đó một value được liên kết với một key. Value chỉ được truy xuất bởi key tương ứng.


```
package main
```

Dưới khai báo package viết 2 hàm khai báo như sau

```
// SumInts adds together the values of m.
func SumInts(m map[string]int64) int64 {
    var s int64
    for _, v := range m {
        s += v
    }
    return s
}

// SumFloats adds together the values of m.
func SumFloats(m map[string]float64) float64 {
    var s float64
    for _, v := range m {
        s += v
    }
    return s
}
```

- SumFloats map giá trị chuỗi thành dạng float64
- SumInts map giá trị chuỗi thành dạng int64

Viết hàm chính để gọi tính tổng

```
func main() {
    // Initialize a map for the integer values
    ints := map[string]int64{
        "first":  34,
        "second": 12,
    }

    // Initialize a map for the float values
    floats := map[string]float64{
        "first":  35.98,
        "second": 26.99,
    }

    fmt.Printf("Non-Generic Sums: %v and %v\n",
        SumInts(ints),
        SumFloats(floats))
}
```

- Khởi tạo biến map các giá trị integer, khởi tạo biến map các giá trị float
- Gọi hàm tính tổng

Import thêm package để in ra màn hình 

```
package main

import "fmt"
```

Chạy chương trình

```
$ go run .
Non-Generic Sums: 46 and 62.97
```

**Add a generic function to handle multiple types**

Tạo 1 hàm có nhiều kiểu dữ liệu, cụ thể ở đây là số nguyên và chuỗi.

Viết đoạn code sau

```
// SumIntsOrFloats sums the values of map m. It supports both int64 and float64
// as types for map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

- Khai báo hàm SumIntsOrFloats với 2 tham số loại K và V, và 1 đối số m kiểu map[K]V trả về kiểu V

- K là kiểu dữ liệu có thể so sánh, có thể sử dụng K là khóa trong map

- V là ràng buộc kiểu giữa 2 loại int64 và float64, dấu gạch dọc ý nói là được phép 1 trong 2 kiểu dữ liệu

- M là map có kiểu K[V] , K và V đã được định kiểu

Cop đoạn code vào file main.go

```
fmt.Printf("Generic Sums: %v and %v\n",
    SumIntsOrFloats[string, int64](ints),
    SumIntsOrFloats[string, float64](floats))
```

- Gọi hàm SumIntsOrFloats vừa được khai báo

- trong ngoặc vuông chỉ định rõ kiểu dữ liệu

Chạy code 

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
```

**Xóa kiểu của đối số khi gọi hàm Generic**

ở file main.go sửa fn main

```
fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
    SumIntsOrFloats(ints),
    SumIntsOrFloats(floats))
```

Chạy chương trình

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
```

**Khai báo kiểu của 1 ràng buộc**

Khai báo kiểu của 1 ràng buộc như 1 interface

Phần trên của hàm main khai báo

```
type Number interface {
    int64 | float64
}
```

- Khai báo interface của Number là dạng ràng buộc
- Khai báo liên kết int64 và float64 bên trong interface

Thêm 1 fn SumNumbers như sau

```
// SumNumbers sums the values of map m. It supports both integers
// and floats as map values.
func SumNumbers[K comparable, V Number](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

- Ở đây V có kiểu ràng buộc Number nhận kiểu int64 hoặc float64

Ở fn main thêm đoạn code 

```
fmt.Printf("Generic Sums with Constraint: %v and %v\n",
    SumNumbers(ints),
    SumNumbers(floats))
```

Chạy chương trình

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
Generic Sums with Constraint: 46 and 62.97
```

File main.go đầy đủ

```
package main

import "fmt"

type Number interface {
    int64 | float64
}

func main() {
    // Initialize a map for the integer values
    ints := map[string]int64{
        "first": 34,
        "second": 12,
    }

    // Initialize a map for the float values
    floats := map[string]float64{
        "first": 35.98,
        "second": 26.99,
    }

    fmt.Printf("Non-Generic Sums: %v and %v\n",
        SumInts(ints),
        SumFloats(floats))

    fmt.Printf("Generic Sums: %v and %v\n",
        SumIntsOrFloats[string, int64](ints),
        SumIntsOrFloats[string, float64](floats))

    fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
        SumIntsOrFloats(ints),
        SumIntsOrFloats(floats))

    fmt.Printf("Generic Sums with Constraint: %v and %v\n",
        SumNumbers(ints),
        SumNumbers(floats))
}

// SumInts adds together the values of m.
func SumInts(m map[string]int64) int64 {
    var s int64
    for _, v := range m {
        s += v
    }
    return s
}

// SumFloats adds together the values of m.
func SumFloats(m map[string]float64) float64 {
    var s float64
    for _, v := range m {
        s += v
    }
    return s
}

// SumIntsOrFloats sums the values of map m. It supports both floats and integers
// as map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}

// SumNumbers sums the values of map m. Its supports both integers
// and floats as map values.
func SumNumbers[K comparable, V Number](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

# lesson 6 - Rest API Todo List + DB Mysql

**Chạy lệnh cài docker mysql**

```
docker run -d --name todo-list-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-root-pass -e MYSQL_DATABASE=todo_db mysql:8.0
```

Trong đó: 
- todo-list-mysql: tên container
- my-root-pass: mật khẩu kết nối
- todo_db: tên DB
- mysql:8.0: mysql version 8.0

Kết nối Mysql bằng phần mềm như PHPMyAdmin, Navicat, MYSQL workbench... xem thêm tại https://codingsight.com/10-best-mysql-gui-tools/

Chạy truy vấn tạo bảng cho todo list:

```
CREATE TABLE `todo_items` (
  `id` int NOT NULL AUTO_INCREMENT,
  `title` varchar(150) CHARACTER SET utf8 NOT NULL,
  `status` enum('Doing','Finished') DEFAULT 'Doing',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

**Xây dựng Rest API service**

Danh sách các API:
- POST /v1/items tạo mới 1 Item với dữ liệu chỉ cần có title, status để mặc định là "Doing"
- GET /v1/items/:id lấy thông tin của 1 item theo id
- GET /v1/items lấy danh sách các Items
- PUT /v1/items/:id update title hoặc status của 1 Item thông qua id
- DELETE /v1/items/:id xoá 1 Item thông qua id

Ở thư mục lesson-6 tạo thư mục todo-list, trong thư mục todo-list tạo module api/todo-list:

```
$ go mod init api/todo-list
go: creating new go.mod: module api/todo-list
```

Add 2 package hỗ trợ là:
- GIN : hỗ trợ xây dựng web/api service
- GORM : hỗ trợ ORM database

```
go get -u github.com/gin-gonic/gin
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

**Tạo file main.go và viết code kết nối Mysql**

```
package main

import (
	"log"

	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

func main() {
	dsn := "root:my-root-pass@tcp(127.0.0.1:3306)/todo_db?charset=utf8mb4&parseTime=True&loc=Local"
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

	if err != nil {
		log.Fatalln("Cannot connect to MySQL:", err)
	}

	log.Println("Connected:", db)
}
```

chạy test thử kết nối

**Viết hàm API ở file main.go**

```
package main

import (
	"strings"
	"log"
	"net/http"
	"strconv"
	"time"

	"github.com/gin-gonic/gin"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

type ToDoItem struct {
	Id        int        `json:"id" gorm:"column:id;"`
	Title     string     `json:"title" gorm:"column:title;"`
	Status    string     `json:"status" gorm:"column:status;"`
	CreatedAt *time.Time `json:"created_at" gorm:"column:created_at;"`
	UpdatedAt *time.Time `json:"updated_at" gorm:"column:updated_at;"`
}

func (ToDoItem) TableName() string { return "todo_items" }

func main() {
	dsn := "root:123456@tcp(127.0.0.1:3306)/todo_db?charset=utf8mb4&parseTime=True&loc=Local"
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})

	if err != nil {
		log.Fatalln("Cannot connect to MySQL:", err)
	}

	log.Println("Connected to MySQL:", db)

	router := gin.Default()

	v1 := router.Group("/v1")
	{
		v1.POST("/items", createItem(db))           // create item
		v1.GET("/items", getListOfItems(db))        // list items
		v1.GET("/items/:id", readItemById(db))      // get an item by ID
		v1.PUT("/items/:id", editItemById(db))      // edit an item by ID
		v1.DELETE("/items/:id", deleteItemById(db)) // delete an item by ID
	}

	router.Run()
}

/*
 if err := c.BindJSON(&newAlbum); err != nil {
}*/

func createItem(db *gorm.DB) gin.HandlerFunc {
	return func(c *gin.Context) {
		var dataItem ToDoItem

		if err := c.ShouldBind(&dataItem); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		if err := c.BindJSON(&dataItem); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		// preprocess title - trim all spaces
		dataItem.Title = strings.TrimSpace(dataItem.Title)

		if dataItem.Title == "" {
			c.JSON(http.StatusBadRequest, gin.H{"error": "title cannot be blank"})
			return
		}

		// do not allow "finished" status when creating a new task
		dataItem.Status = "Doing" // set to default

		if err := db.Create(&dataItem).Error; err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		c.JSON(http.StatusOK, gin.H{"data": dataItem.Id})
	}
}
	
func readItemById(db *gorm.DB) gin.HandlerFunc {
	return func(c *gin.Context) {
		var dataItem ToDoItem

		id, err := strconv.Atoi(c.Param("id"))

		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		if err := db.Where("id = ?", id).First(&dataItem).Error; err != nil {
			c.JSON(http.StatusNotFound, gin.H{"error": err.Error()})
			return
		}

		c.JSON(http.StatusOK, gin.H{"data": dataItem})
	}
}
	
func getListOfItems(db *gorm.DB) gin.HandlerFunc {
	return func(c *gin.Context) {
		type DataPaging struct {
			Page  int   `json:"page" form:"page"`
			Limit int   `json:"limit" form:"limit"`
			Total int64 `json:"total" form:"-"`
		}

		var paging DataPaging

		if err := c.ShouldBind(&paging); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error1": err.Error()})
			return
		}

		if paging.Page <= 0 {
			paging.Page = 1
		}

		if paging.Limit <= 0 {
			paging.Limit = 10
		}

		offset := (paging.Page - 1) * paging.Limit

		var result []ToDoItem

		if err := db.Table(ToDoItem{}.TableName()).
			Count(&paging.Total).
			Offset(offset).
			Order("id desc").
			Find(&result).Error; err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		c.JSON(http.StatusOK, gin.H{"data": result})
	}
}
	
func editItemById(db *gorm.DB) gin.HandlerFunc {
	return func(c *gin.Context) {
		id, err := strconv.Atoi(c.Param("id"))

		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		var dataItem ToDoItem

		if err := c.ShouldBind(&dataItem); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		if err := c.BindJSON(&dataItem); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}	

		if err := db.Where("id = ?", id).Updates(&dataItem).Error; err != nil {
			c.JSON(http.StatusNotFound, gin.H{"error": err.Error()})
			return
		}

		c.JSON(http.StatusOK, gin.H{"data": true})
	}
}
	
func deleteItemById(db *gorm.DB) gin.HandlerFunc {
	return func(c *gin.Context) {
		id, err := strconv.Atoi(c.Param("id"))

		if err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		if err := db.Table(ToDoItem{}.TableName()).
			Where("id = ?", id).
			Delete(nil).Error; err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		c.JSON(http.StatusOK, gin.H{"data": true})
	}
}
	
```


Chạy thử chương trình 

```
$ go run main.go
```

Kết quả

```
2023/01/12 11:52:27 Connected to MySQL: &{0x14000138240 <nil> 0 0x140002f4000 1}
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /v1/items                 --> main.createItem.func1 (3 handlers)
[GIN-debug] GET    /v1/items                 --> main.getListOfItems.func1 (3 handlers)
[GIN-debug] GET    /v1/items/:id             --> main.readItemById.func1 (3 handlers)
[GIN-debug] PUT    /v1/items/:id             --> main.editItemById.func1 (3 handlers)
[GIN-debug] DELETE /v1/items/:id             --> main.deleteItemById.func1 (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
```
