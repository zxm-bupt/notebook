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

# 值接收者和指针接收者的问题

首先，方法的调用者有值和指针两种，然后接收者也有这两种，因此，有四种情况。

|         | 值接收者                                                     | 指针接收者                                                     |
| ------- | -------------------------------------------------------- | --------------------------------------------------------- |
| 值类型调用者  | 方法会使用调用者的一个副本，类似于“传值”                                    | 使用值的引用来调用方法，上例中，`qcrao.growUp()` 实际上是 `(&qcrao).growUp()` |
| 指针类型调用者 | 指针被解引用为值，上例中，`stefno.howOld()` 实际上是 `(*stefno).howOld()` | 实际上也是“传值”，方法里的操作会影响到调用者，类似于指针传参，拷贝了一份指针                   |

**值接收者与值调用者的情况如下图所示**

```go
type Person struct {
    age int
}

func (p Person) howOld() int {
    return p.age
}

func (p Person) growUp() {
    p.age += 1
}

func main() {
    p := Person{age: 10}
    println(p.howOld()) // 10
    p.growUp()
    println(p.howOld()) // 10
}
```

其输出结果为

```shell
10
10
```

只会进行值传递，因此不会修改调用者的值。

**指针接收者和值调用者的情况下所示**

```go
package main

type Person struct {
    age int
}

func (p Person) howOld() int {
    return p.age
}

func (p *Person) growUp() {
    p.age += 1
}

func main() {
    p := Person{age: 10}
    println(p.howOld()) // 10
    p.growUp()
    println(p.howOld()) // 11
}

```

输出为

```shell
10
11
```

对于方法growUp而言，p是一个值调用者，方法的接收者是一个指针，实际上这是一个语法糖，编译器使用引用来对growUp()进行了调用，即`(&p).growUp()`

**指针调用者和值接收者**

```go
type Person struct {
    age int
}

func (p Person) howOld() int {
    return p.age
}

func (p Person) growUp() {
    p.age += 1
}

func main() {
    p := &Person{age: 10}
    println(p.howOld()) // 10
    p.growUp()
    println(p.howOld()) // 10
}
```

输出为

```shell
10
10
```

该操作相当于解引用，即`(*p).howOld`和`(*p).growUp`

**指针调用者和指针接收者**

```go
type Person struct {
    age int
}

func (p *Person) howOld() int {
    return p.age
}

func (p *Person) growUp() {
    p.age += 1
}

func main() {
    p := &Person{age: 10}
    println(p.howOld()) // 10
    p.growUp()
    println(p.howOld()) // 11
}

```

结果为

```shell
10
11
```

相当传递将地址作为值传递。

总结来看，在使用方法的时候，调用者是什么对结果其实没有区别。主要在接收者上，值接收者不会改变内容，指针接收者可以。

---

# 接口实现和值接收者与指针接收者

对于接口的实现上，值接收者的方法和指针接收者的方法会对调用者有影响。

**指针接收者实现的方法**

```go
package main

type Human interface {
    howOld() int
    growUp()
}

type Person struct {
    age int
}

func (p *Person) howOld() int {
    return p.age
}

func (p *Person) growUp() {
    p.age += 1
}

func main() {
    var p Human = &Person{age: 10}
    println(p.howOld()) // 10
    p.growUp()
    println(p.howOld()) // 11
}
```

对于指针类型变量，显然实现了接口。

```go
package main

type Human interface {
    howOld() int
    growUp()
}

type Person struct {
    age int
}

func (p *Person) howOld() int {
    return p.age
}

func (p *Person) growUp() {
    p.age += 1
}

func main() {
    var p Human = Person{age: 10}
    println(p.howOld()) // 10
    p.growUp()
    println(p.howOld()) // 11
}
```

对于值类型的变量，并未实现对应的接口

结果为

```shell
# command-line-arguments
./main.go:21:16: cannot use Person{…} (value of type Person) as Human value in variable declaration: Person does not implement Human (method growUp has pointer receiver)
```

**值类型接收者实现的方法**

```go
package main

type Human interface {
    howOld() int
    growUp()
}

type Person struct {
    age int
}

func (p Person) howOld() int {
    return p.age
}

func (p Person) growUp() {
    p.age += 1
}

func main() {
    var p Human = Person{age: 10}
    println(p.howOld()) // 10
    p.growUp()
    println(p.howOld()) // 10
}
```

对于值类型的变量，显然实现了接口

```go
package main

type Human interface {
    howOld() int
    growUp()
}

type Person struct {
    age int
}

func (p Person) howOld() int {
    return p.age
}

func (p Person) growUp() {
    p.age += 1
}

func main() {
    var p Human = &Person{age: 10}
    println(p.howOld()) // 10
    p.growUp()
    println(p.howOld()) // 10
}
```

对于指针类型的变量同样实现了接口。

**总结**

接口的实现，值类型接收者和指针类型接收者不一样： 

- 以值类型接收者实现接口，类型本身和该类型的指针类型，都实现了该接口 

- 以指针类型接收者实现接口，只有对应的指针类型才被认为实现了接口

另外，这里用的所有都是变量没有实现接口，而不是说，该变量不能调用方法，因为无论这个变量是值还是指针，方法的接收者是指针还是值，都可以调用，而不出现编译问题。

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


