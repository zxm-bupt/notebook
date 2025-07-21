# 1. 概述

Go中的string底层是一个结构体，包含两个字段：

```go
type StringHeader struct {
    Data uintptr // 指向底层字节数组的指针
    Len  int     // 字符串长度
}
```

## 核心特点

### 1. 不可变性 (Immutable)

```go
s := "hello"
//s[0] = 'H'//编译错误：cannot assign to s[0]
```

go的string支持+操作，但是因为不可变性，所以以下是创建一个新的字符串

```go
s := "hello"
s = s[0:1] + s[2:]
```

### 2. 零拷贝切片

```go
package main

import "fmt"

func main() {
    s := "hello world"
    sub := s[0:5] // "hello" - 不会复制数据，只是创建新的string header
    
    fmt.Printf("原字符串地址: %p\n", &s)
    fmt.Printf("子字符串地址: %p\n", &sub)
    // 底层数据指针相同
}
```

# 2. 字符串优化

## 2.1 与[]byte的转换

```go
package main

import (
    "unsafe"
    "reflect"
)

// string to []byte (会复制数据)
func stringToBytes(s string) []byte {
    return []byte(s)
}

// 零拷贝转换 (不安全，仅用于理解原理)
func stringToBytesUnsafe(s string) []byte {
    sh := (*reflect.StringHeader)(unsafe.Pointer(&s))
    bh := reflect.SliceHeader{
        Data: sh.Data,
        Len:  sh.Len,
        Cap:  sh.Len,
    }
    return *(*[]byte)(unsafe.Pointer(&bh))
}
```

## 2.2 字符串拼接优化

```go
package main

import (
    "strings"
    "fmt"
)

func inefficientConcat() string {
    result := ""
    for i := 0; i < 1000; i++ {
        result += "a" // 每次都会创建新字符串
    }
    return result
}

func efficientConcat() string {
    var builder strings.Builder
    builder.Grow(1000) // 预分配容量
    for i := 0; i < 1000; i++ {
        builder.WriteString("a")
    }
    return builder.String()
}
```

## 3. 字符串字面量存储

```go
package main

import "fmt"

func main() {
    // 字符串字面量存储在只读数据段
    s1 := "hello"
    s2 := "hello"
    
    // 相同的字符串字面量共享内存
    fmt.Printf("s1 data pointer: %p\n", &s1)
    fmt.Printf("s2 data pointer: %p\n", &s2)
    // 底层Data指针相同
}
```

