---
tags: [go, language, Notebooks/daily]
title: Effective Go
created: '2022-07-09T07:58:56.417Z'
modified: '2022-07-09T10:45:28.267Z'
---

# Effective Go

* 指针和值的方法有所不同。针对值的方法，可以作用于值和指针；针对指针的方法，只能作用于指针。  
  主要的不同是 __值的方法无法修改值本身，而指针可以__ 。  
  例外：当对值调用原本作用于指针的方法时，如果值可以取地址，则编译器会帮忙转换。  

  ```go
  type Size int
  func (s *Size) Add(a Size) {}

  var s Size
  s.Add(1) // 等价于 (&s).Add(1)，此处编译器帮忙进行了转换
  ```

* 类型转换 可以帮忙程序调用正确的方法

* 接口转换(interface conversion) vs 类型断言(type assertion)
  ```go
  type Stringer interface {
      String() string
  }

  var value interface{} // Value provided by caller.
  switch str := value.(type) {
  case string:
      return str
  case Stringer:
      return str.String()
  }
  ```

  类型断言可以帮忙把value中的指定类型成员拿出来使用，如果没有，则会抛runtime err或将ok置false
  ```go
  if str, ok := value.(string); ok {  // 尝试类型断言
      return str
  } else if str, ok := value.(Stringer); ok { //
      return str.String()
  }
  ``` 

* 除了指针和接口，方法可以针对任何类型定义，甚至可以 __为函数类型定义方法__
  ```go
  type Handler interface {
      ServeHTTP(ResponseWriter, *Request)
  }

  // The HandlerFunc type is an adapter to allow the use of
  // ordinary functions as HTTP handlers.  If f is a function
  // with the appropriate signature, HandlerFunc(f) is a
  // Handler object that calls f.
  type HandlerFunc func(ResponseWriter, *Request)

  // ServeHTTP calls f(w, req).
  func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
      f(w, req)
  }

  // Argument server.
  func ArgServer(w http.ResponseWriter, req *http.Request) {
      fmt.Fprintln(w, os.Args)
  }

  http.Handle("/args", http.HandlerFunc(ArgServer))

  ```

* _可以用来将无用的变量赋值给它。大多是为了避免编译错误。
  * 有些变量暂时用不到
    ```go
    package main

    import (
        "fmt"
        "io"
        "log"
        "os"
    )

    var _ = fmt.Printf // For debugging; delete when done.
    var _ io.Reader    // For debugging; delete when done.

    func main() {
        fd, err := os.Open("test.go")
        if err != nil {
            log.Fatal(err)
        }
        // TODO: use fd.
        _ = fd
    }
    ```
  * 有些import包，只是想要它的副作用，而不会真正使用它
    ```go
    import _ "net/http/pprof"
    ```
  * 有时候想在编译期检查类型转换是否ok
    ```go
    var _ json.Marshaler = (*RawMessage)(nil)
    ```
  * 判断value是否实现了该接口
    ```go
    if _, ok := val.(json.Marshaler); ok {
      fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
    }
    ```

