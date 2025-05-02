# go的字符串处理

1. go的strconv.Atoi(str)的使用是
   
   ```go
    int, err := strconv.Atoi("123")
    // int 为 123
   ```

2. go实现Str转换为ASCII,可以这样实现
   
   ```go
    ascii := 0
    for _ , char := range str {
   
        ascii = ascii + int(rune(str))
   
    }
   ```

3. string 、int、 rune的转换
   
   ```go
   // str转int、rune
    runes := []rune(str)
    ascii := 0
    for _ , char := range str {
   
        ascii = ascii + int(rune(str))
   
    }
    // rune 转str
    str := string([]rune)
    str := string(rune)
    // rune 转int
    value := int(rune)
   ```

---

# go的copy

go的copy和C++很像，如果是指针，那就是深copy，如果是值传递，那就是浅拷贝

---

# go的反射

go的reflect 参考：[Go的反射](https://www.cnblogs.com/jiujuan/p/17142703.html)

---

# go的匿名函数

go的[匿名函数](https://gopl-zh.codeyu.com/ch5/ch5-06.html)

---

# go的数学库

go没有定义类似C 中MAX_INT这样的常量，如果要使用最大值，需要使用math数学库。

```go
math.MaxInt32 
math.MaxInt 
```

同时go中没有幂运算符号，`^`运算符位按位异或运算符号，如果要计算幂次

```go
int(math.Pow(2,31))
```

借助`^`可以这样定义int的最大值

```go
const MAX_INT = int(^uint(0) >> 1)
```

```shell
uint(0) = 00000000...00000000 (全 0)
^uint(0) = 11111111...11111111 (全 1)
^uint(0) >> 1  = 01111111...11111111 (最高位变为 0)
```

---
