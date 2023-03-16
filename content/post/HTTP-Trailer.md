---
title: HTTP-Trailer
date: 2020-10-20 21:49:46
---

## Concept

**Trailer** 是一个响应首部，允许发送方在分块发送的消息后面添加额外的元信息，这些元信息可能是随着消息主体的发送动态生成的，比如消息的完整性校验，消息的数字签名，或者消息经过处理之后的最终状态等。
<!--more-->

## Grammar

```
Trailer: header-names
```

header-names 是出现在分块信息挂载部分的消息首部。以下首部字段**不允许**出现：

- 用于信息分帧的首部 (例如 [`Transfer-Encoding`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Transfer-Encoding) 和 [`Content-Length`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Length))，
- 用于路由用途的首部 (例如 [`Host`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Host))，
- 请求修饰首部 (例如控制类和条件类的，如 [`Cache-Control`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)，[`Max-Forwards`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Max-Forwards)，或者 [`TE`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/TE))，
- 身份验证首部 (例如 [`Authorization`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Authorization) 或者 [`Set-Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie))，
- [`Content-Encoding`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Encoding)，[`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type)，[`Content-Range`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Range)，以及 `Trailer` 自身。

## Example

```http
HTTP/1.1 200 OK 
Content-Type: text/plain 
Transfer-Encoding: chunked
Trailer: Expires

7\r\n 
Mozilla\r\n 
9\r\n 
Developer\r\n 
7\r\n 
Network\r\n 
0\r\n 
Expires: Wed, 21 Oct 2015 07:28:00 GMT\r\n
\r\n
```

解释：

1. `Header` 里面的 `Transfer-Encoding` 必须是 `chunked`，也就是说不能指定 `Content-Length`。
2. `Trailer` 的字段名字必须在 `Header` 里面提前声明，比如上面的 `Trailer: Expires`。
3. `Trailer` 在 `Body` 发完之后再发，格式和 `Header` 类似。

## Implement in Go

用 Go 实现一个 HTTP 客户端，对所发的 `Body` 计算 `MD5` 并通过 `Trailer` 传给服务端。服务端收到请求并对 Body 进行校验。

### Server Code

```go
package main

import (
	"crypto/md5"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func checkMd5(w http.ResponseWriter, req *http.Request) {
	fmt.Printf("Header:%+v\n", req.Header)
	fmt.Println("Trailer before read body:")
	fmt.Println(req.Trailer)

	data, err := ioutil.ReadAll(req.Body)
	bodyMd5 := fmt.Sprintf("%x", md5.Sum(data))
	defer req.Body.Close()

	fmt.Println("body:", string(data))
	fmt.Println("md5:", bodyMd5)
	fmt.Println("error:", err)
	fmt.Println("Trailer after read body:")
	fmt.Println(req.Trailer)

	if req.Trailer.Get("md5") != bodyMd5 {
		panic("body md5 not equal")
	}
}

func main() {
	http.HandleFunc("/", checkMd5)
	log.Fatal(http.ListenAndServe(":12345",nil))
}
```

### Client Code

```go
package main

import (
	"crypto/md5"
	"fmt"
	"hash"
	"io"
	"net/http"
	"os"
	"strconv"
	"strings"
)

type headerReader struct {
	reader io.Reader
	md5    hash.Hash
	header http.Header
}

func (r *headerReader) Read(p []byte) (n int, err error) {
	n, err = r.reader.Read(p)
	if n > 0 {
		r.md5.Write(p[:n])
	}
	if err == io.EOF {
		r.header.Set("md5", fmt.Sprintf("%x", r.md5.Sum(nil)))
	}
	return
}

func main() {
	h := &headerReader{
		reader: strings.NewReader("body"),
		md5:    md5.New(),
		header: http.Header{"md5": nil, "size": []string{strconv.Itoa(len("body"))}},
	}
	req, err := http.NewRequest("POST", "http://localhost:12345", h)
	if err != nil {
		panic(err)
	}
	req.ContentLength = -1
	req.Trailer = h.header
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		panic(err)
	}
	fmt.Println(resp.Status)
	_, err = io.Copy(os.Stdout, resp.Body)
	if err != nil {
		panic(err)
	}
}
```

### Result

#### Server result

```
$ go run server.go
Header:map[Accept-Encoding:[gzip] User-Agent:[Go-http-client/1.1]]
Trailer before read body:
map[Md5:[] Size:[]]
body: body
md5: 841a2d689ad86bd1611447453c22c6fc
error: <nil>
Trailer after read body:
map[Md5:[841a2d689ad86bd1611447453c22c6fc] Size:[4]]
```

#### Client result

```
$ go run client.go
200 OK
```

#### nc result

```
$ nc -l 12345
POST / HTTP/1.1
Host: localhost:12345
User-Agent: Go-http-client/1.1
Transfer-Encoding: chunked
Trailer: Md5,Size
Accept-Encoding: gzip

4
body
0
Md5: 841a2d689ad86bd1611447453c22c6fc
size: 4
```

## Conclusion

可以看到服务端在读完 body 之前只能知道有 `Md5` 这个 `Trailer`，值为空；读完 body 之后，能正常拿到 `Trailer` 的 `Md5` 值。

Go 语言使用 `Trailer` 也有几个注意事项：

1. `req.ContentLength` 必须设置为 `0` 或者 `-1`，这样 `body` 才会以 `chunked` 的形式传输。
2. `req.Trailer` 需要在发请求之前声明所有的 key 字段，在 body 发完之后设置相应的 value，如果客户端提前知道 `Trailer` 的值的话也可以提前设置，比如上面例子里面的 `size` 字段。
3. 发完 `body` 之后 `Trailer` 不允许再更改，否则可能会因为 map 并发读写，导致程序 panic，同样的道理服务端在读 `body` 的时候也不应该对 `Trailer` 有引用。
4. 服务端**必须**读完 `body` 之后才能知道 `Trailer` 的值。

## References

+ https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Trailer
+ https://github.com/ma6174/blog/issues/22
