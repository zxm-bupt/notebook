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

---

# go的泛型(模板)

> 如果你经常要分别为不同的类型写完全相同逻辑的代码，那么使用泛型将是最合适的选择



## 1. 类型形参、类型实参、类型约束和泛型类型

```go
type Slice[T int|float32|float64 ] []T
```

不同于一般的类型定义，这里类型名称 Slice 后带了中括号，对各个部分做一个解说就是：

* T 就是上面介绍过的**类型形参(Type parameter)**，在定义Slice类型的时候 T 代表的具体类型并不确定，类似一个占位符

* int|float32|float64 这部分被称为**类型约束(Type constraint)**，中间的 | 的意思是告诉编译器，类型形参 T 只可以接收 int 或 float32 或 float64 这三种类型的实参

* 中括号里的 T int|float32|float64 这一整串因为定义了所有的类型形参(在这个例子里只有一个类型形参T），所以我们称其为**类型形参列表(type parameter list)**

* 这里新定义的类型名称叫 Slice[T]

泛型类型不能直接拿来使用，必须传入**类型实参(Type argument)** 将其确定为具体的类型之后才可使用。而传入类型实参确定具体类型的操作被称为 **实例化(Instantiations)** :

```go
// 这里传入了类型实参int，泛型类型Slice[T]被实例化为具体的类型 Slice[int]
var a Slice[int] = []int{1, 2, 3}  
fmt.Printf("Type Name: %T",a)  //输出：Type Name: Slice[int]

// 传入类型实参float32, 将泛型类型Slice[T]实例化为具体的类型 Slice[string]
var b Slice[float32] = []float32{1.0, 2.0, 3.0} 
fmt.Printf("Type Name: %T",b)  //输出：Type Name: Slice[float32]

// ✗ 错误。因为变量a的类型为Slice[int]，b的类型为Slice[float32]，两者类型不同
a = b  

// ✗ 错误。string不在类型约束 int|float32|float64 中，不能用来实例化泛型类型
var c Slice[string] = []string{"Hello", "World"} 

// ✗ 错误。Slice[T]是泛型类型，不可直接使用必须实例化为具体的类型
var x Slice[T] = []int{1, 2, 3} 
```

对于上面的例子，我们先给泛型类型 `Slice[T]` 传入了类型实参 `int` ，这样泛型类型就被实例化为了具体类型 `Slice[int]`

上面只是个最简单的例子，实际上类型形参的数量可以远远不止一个，如下：

```go
// MyMap类型定义了两个类型形参 KEY 和 VALUE。分别为两个形参指定了不同的类型约束
// 这个泛型类型的名字叫： MyMap[KEY, VALUE]
type MyMap[KEY int | string, VALUE float32 | float64] map[KEY]VALUE  

// 用类型实参 string 和 flaot64 替换了类型形参 KEY 、 VALUE，泛型类型被实例化为具体的类型：MyMap[string, float64]
var a MyMap[string, float64] = map[string]float64{
    "jack_score": 9.6,
    "bob_score":  8.4,
}
```

关于类型形参、类型约束、类型形参列表、类型实参可以参照下图的概括：

![](http://123.57.190.49:12121/api/image/2J2J86J0.jpg)

关于泛型的基本概念还有以下一些在语法上需要注意的地方：

* 所有类型定义都可使用类型形参，所以下面这种结构体以及接口的定义也可以使用类型形参：
  
  ```go
  // 一个泛型类型的结构体。可用 int 或 sring 类型实例化
  type MyStruct[T int | string] struct {  
      Name string
      Data T
  }
  
  // 一个泛型接口(关于泛型接口在后半部分会详细讲解）
  type IPrintData[T int | float32 | string] interface {
      Print(data T)
  }
  
  // 一个泛型通道，可用类型实参 int 或 string 实例化
  type MyChan[T int | string] chan T
  ```

* 类型形参是可以互相套用的，如下
  
  ```go
  type WowStruct[T int | float32, S []T] struct {
      Data     S
      MaxValue T
      MinValue T
  }
  ```
  
  这个例子看起来有点复杂且难以理解，但实际上只要记住一点：任何泛型类型都必须传入类型实参实例化才可以使用。
  因为 S 的定义是 []T ，所以 T 一定决定了的话 S 的实参就不能随便乱传了，下面这样的代码是错误的：
  
  ```go
  // 错误。S的定义是[]T，这里T传入了实参int, 所以S的实参应当为 []int 而不能是 []float32
  ws := WowStruct[int, []float32]{
          Data:     []float32{1.0, 2.0, 3.0},
          MaxValue: 3,
          MinValue: 1,
      }
  ```

* 定义泛型类型的时候，**基础类型不能只有类型形参**，如下：
  
  ```go
  type MyType [T int|float32] T
  ```
  
  这种方式就是错误的

* 当类型约束的一些写法会被编译器误认为是表达式时会报错。如下：
  
  ```go
  
  //✗ 错误。T *int会被编译器误认为是表达式 T乘以int，而不是int指针
  type NewType[T *int] []T
  // 上面代码再编译器眼中：它认为你要定义一个存放切片的数组，数组长度由 T 乘以 int 计算得到
  type NewType [T * int][]T 
  
  //✗ 错误。和上面一样，这里不光*被会认为是乘号，| 还会被认为是按位或操作
  type NewType2[T *int|*float64] []T 
  
  //✗ 错误
  type NewType2 [T (int)] []T 
  ```
  
  那么为了解决这个问题，可以使用interface{}来消除歧义。如下:
  
  ```go
  type NewType[T interface{*int}] []T
  type NewType2[T interface{*int|*float64}] []T 
  // 如果类型约束中只有一个类型，可以添加个逗号消除歧义
  type NewType3[T *int,] []T
  //✗ 错误。如果类型约束不止一个类型，加逗号是不行
  type NewType4[T *int|*float32,] []T 
  ```

* 泛型和普通的类型一样，可以互相嵌套定义出更加复杂的新类型，如下：
  
  ```go
  
  // 先定义个泛型类型 Slice[T]
  type Slice[T int|string|float32|float64] []T
  
  // ✗ 错误。泛型类型Slice[T]的类型约束中不包含uint, uint8
  type UintSlice[T uint|uint8] Slice[T]  
  
  // ✓ 正确。基于泛型类型Slice[T]定义了新的泛型类型 FloatSlice[T] 。FloatSlice[T]只接受float32和float64两种类型
  type FloatSlice[T float32|float64] Slice[T] 
  
  // ✓ 正确。基于泛型类型Slice[T]定义的新泛型类型 IntAndStringSlice[T]
  type IntAndStringSlice[T int|string] Slice[T]  
  // ✓ 正确 基于IntAndStringSlice[T]套娃定义出的新泛型类型
  type IntSlice[T int] IntAndStringSlice[T] 
  
  // 在map中套一个泛型类型Slice[T]
  type WowMap[T int|string] map[string]Slice[T]
  // 在map中套Slice[T]的另一种写法
  type WowMap2[T Slice[int] | Slice[string]] map[string]T
  ```

* go的匿名结构体不支持泛型
  
  ```go
  //✗ 错误。匿名结构体不支持泛型
  testCase := struct[T int|string] {
          caseName string
          got      T
          want     T
      }[int]{
          caseName: "test OK",
          got:      100,
          want:     100,
      }
  ```

## 2. 泛型receiver

定义了新的普通类型之后可以给类型添加方法。那么可以给泛型类型添加方法吗？答案自然是可以的，如下：

```go
type MySlice[T int | float32] []T

func (s MySlice[T]) Sum() T {
    var sum T
    for _, value := range s {
        sum += value
    }
    return sum
}
```

那现在，我们可以这样使用了

```go
var a MySlice[int] = []int{1,2,3}
print(a.Sum())// 6
var b MySlice[float32] = []float32{1.0,2.0,3.0}
print(b.Sum())// 6.0
```

再有了泛型后，我们就能实现一些通用的数据结构了，例如堆，栈，队列等，这些go的需要import库来实现的类型。当然也可以用接口+反射实现，但是我想没人愿意放着优雅的泛型不用，去用又恶心又性能低下的反射。

下面是使用泛型使用的堆

```go
type Ordered interface {
    ~int | ~float64 | ~string // 允许底层类型为 int、float64 或 string 的类型
}

type Heap[T Ordered] struct {
    H     []T
    isMax bool
}

// 传入一个参数,来确定时大顶堆还是小顶堆
func Construct[T Ordered](nums []T, isMax ...bool) *Heap[T] {
    heap := &Heap[T]{H: nums, isMax: true}
    if len(isMax) > 0 {
        heap.isMax = isMax[0]
    }
    heap.Heapify()
    return heap
}

func (h *Heap[T]) siftDown(i int) {
    // Implementation of siftDown
    location := i
    for location <= len(h.H)/2-1 {
        left := 2*location + 1
        right := 2*location + 2
        //这里的判断条件需要 将大顶堆和小顶堆合并，用一套逻辑
        canChangeL := left < len(h.H) && (h.isMax && h.H[left] > h.H[location] || !h.isMax && h.H[left] < h.H[location])
        canChangeR := right < len(h.H) && (h.isMax && h.H[right] > h.H[location] || !h.isMax && h.H[right] < h.H[location])
        if canChangeL && canChangeR {
            selectL := h.isMax && h.H[left] > h.H[right] || !h.isMax && h.H[left] < h.H[right]
            if selectL {
                canChangeR = false
            } else {
                canChangeL = false
            }
        }
        if canChangeL && !canChangeR {
            h.H[location], h.H[left] = h.H[left], h.H[location]
            location = left
            continue
        }
        if canChangeR && !canChangeL {
            h.H[location], h.H[right] = h.H[right], h.H[location]
            location = right
            continue
        }
        break
    }
}

func (h *Heap[T]) Heapify() {
    n := len(h.H)
    for i := n/2 - 1; i >= 0; i-- {
        h.siftDown(i)
    }
}

func (h *Heap[T]) Insert(value T) {
    h.H = append(h.H, value)
    index := len(h.H) - 1
    for index > 0 {
        parent := (index - 1) / 2
        if h.isMax {
            if h.H[parent] < h.H[index] {
                h.H[parent], h.H[index] = h.H[index], h.H[parent]
                index = parent
            } else {
                break
            }
        } else {
            if h.H[parent] > h.H[index] {
                h.H[parent], h.H[index] = h.H[index], h.H[parent]
                index = parent
            } else {
                break
            }
        }
    }
}

func (h *Heap[T]) ExtractMax() T {
    if len(h.H) == 0 {
        var zero T
        return zero // or some error value
    }
    max := h.H[0]
    h.H[0] = h.H[len(h.H)-1]
    h.H = h.H[:len(h.H)-1]
    h.siftDown(0)
    return max
}
```

使用接口的时候经常会用到类型断言或 type swith 来确定接口具体的类型，然后对不同类型做出不同的处理，如：

```go
var i interface{} = 123
i.(int) // 类型断言

// type switch
switch i.(type) {
    case int:
        // do something
    case string:
        // do something
    default:
        // do something
    }
}
```

那么你一定会想到，对于 `valut T` 这样通过类型形参定义的变量，我们能不能判断具体类型然后对不同类型做出不同处理呢？答案是不允许的，如下：

```go
func (q *Queue[T]) Put(value T) {
    value.(int) // 错误。泛型类型定义的变量不能使用类型断言

    // 错误。不允许使用type switch 来判断 value 的具体类型
    switch value.(type) {
    case int:
        // do something
    case string:
        // do something
    default:
        // do something
    }

    // ...
}
```

虽然type switch和类型断言不能用，但我们可通过反射机制达到目的：

```go
func (receiver Queue[T]) Put(value T) {
    // Printf() 可输出变量value的类型(底层就是通过反射实现的)
    fmt.Printf("%T", value) 

    // 通过反射可以动态获得变量value的类型从而分情况处理
    v := reflect.ValueOf(value)

    switch v.Kind() {
    case reflect.Int:
        // do something
    case reflect.String:
        // do something
    }

    // ...
}
```

这看起来达到了我们的目的，可是当你写出上面这样的代码时候就出现了一个问题：

你为了避免使用反射而选择了泛型，结果到头来又为了一些功能在在泛型中使用反射

当出现这种情况的时候你可能需要重新思考一下，自己的需求是不是真的需要用泛型（毕竟泛型机制本身就很复杂了，再加上反射的复杂度，增加的复杂度并不一定值得）。

当然，这一切选择权都在你自己的手里，根据具体情况斟酌。

## 3. 泛型函数

在介绍完泛型类型和泛型receiver之后，我们来介绍最后一个可以使用泛型的地方——泛型函数。有了上面的知识，写泛型函数也十分简单。假设我们想要写一个计算两个数之和的函数：

```go
func Add[T int|float32|float64|string](a, b T) T{
    return a + b
}
```

上面就是泛型函数的定义。

> 这种带类型形参的函数被称为**泛型函数**

它和普通函数的点不同在于函数名之后带了类型形参。这里的类型形参的意义、写法和用法因为与泛型类型是一模一样的，就不再赘述了。

和泛型类型一样，泛型函数也是不能直接调用的，要使用泛型函数的话必须传入类型实参之后才能调用。

```go
Add[int](1,2) // 传入类型实参int，计算结果为 3
Add[float32](1.0, 2.0) // 传入类型实参float32, 计算结果为 3.0
Add[string]("hello", "world") // 错误。因为泛型函数Add的类型约束中并不包含string
```

Go同时支持类型实参的自动推导：

```go
Add(1, 2)  // 1，2是int类型，编译请自动推导出类型实参T是int
Add(1.0, 2.0) // 1.0, 2.0 是浮点，编译请自动推导出类型实参T是float32
```

我们想到，在匿名的结构体中不支持泛型，那匿名函数是不是也不支持呢？恭喜你猜对了，匿名函数也不支持泛型:

```go
// 错误，匿名函数不能自己定义类型实参
fnGeneric := func[T int | float32](a, b T) T {
        return a + b
} 

fmt.Println(fnGeneric(1, 2))
```

但是匿名函数可以使用别处定义好的类型实参，如：

```go
func MyFunc[T int | float32 | float64](a, b T) {

    // 匿名函数可使用已经定义好的类型形参
    fn2 := func(i T, j T) T {
        return i*2 - j*2
    }

    fn2(a, b)
}
```

那么支不支持泛型方法呢？很不幸，目前Go的方法并不支持泛型，如下：

```go
type A struct {
}

// 不支持泛型方法
func (receiver A) Add[T int | float32 | float64](a T, b T) T {
    return a + b
}
```

但是因为receiver支持泛型， 所以如果想在方法中使用泛型的话，目前唯一的办法就是曲线救国，迂回地通过receiver使用类型形参：

```go
type A[T int | float32 | float64] struct {
}

// 方法可以使用类型定义中的形参 T 
func (receiver A[T]) Add(a T, b T) T {
    return a + b
}

// 用法：
var a A[int]
a.Add(1, 2)

var aa A[float32]
aa.Add(1.0, 2.0)
```

这很有go的特点，既然receiver支持泛型了，那方法就不支持了，less is more。

## 4. 泛型中的interface{}

### 4.1 interface{}样式的类型约束

go的泛型约束有时会写的很长，这显然不方便阅读(其实方便写，因为AI会自动补全)。因此，可以将类型约束封装成一个接口。其实我觉的go的interface{}不应该称之为接口，更应该是一种类型。他和其他语言的接口定义显然不太相似，或者说他的定义更像是java接口的超集。例如下面常见的go的接口

```go
type MyInterface interface{
    Work()
    Relax()
}
```

这就可以理解成，MyInterface是一种类型，一种实现了Work()和Relax()方法的类型，这种理解方式，要比单纯的理解成接口要好的多。也更符合less is more的设计理念。

回到泛型上来，对于如下的类型约束，可以这样定义：

```go
// 一个可以容纳所有int,uint以及浮点类型的泛型切片
type Slice[T int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64 | float32 | float64] []T
```

```go
type IntUintFloat interface {
    int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64 | float32 | float64
}

type Slice[T IntUintFloat] []T
```

不过这样的代码依旧不好维护，而接口和接口、接口和普通类型之间也是可以通过 `|` 进行组合：

```go
type Int interface {
    int | int8 | int16 | int32 | int64
}

type Uint interface {
    uint | uint8 | uint16 | uint32
}

type Float interface {
    float32 | float64
}

type Slice[T Int | Uint | Float] []T  // 使用 '|' 将多个接口类型组合
```

上面的代码中，我们分别定义了 Int, Uint, Float 三个接口类型，并最终在 Slice[T] 的类型约束中通过使用 `|` 将它们组合到一起。

同时，在接口里也能直接组合其他接口，所以还可以像下面这样：

```go
type SliceElement interface {
    Int | Uint | Float | string // 组合了三个接口类型并额外增加了一个 string 类型
}

type Slice[T SliceElement] []T 
```

得益于go自身的缺点，我们还需要解决如下一个问题：

```go
var s1 Slice[int] // 正确 

type MyInt int
var s2 Slice[MyInt] // ✗ 错误。MyInt类型底层类型是int但并不是int类型，不符合 Slice[T] 的类型约束
```

显然MyInt是int类型，但是MyInt却不能作为Slice的类型实参。这说明，go的类型系统比较抽象。为了解决这个问题，go引入`~`，上面的问题我们可以这样解决：

```go
type Int interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64
}

type Uint interface {
    ~uint | ~uint8 | ~uint16 | ~uint32
}
type Float interface {
    ~float32 | ~float64
}

type Slice[T Int | Uint | Float] []T 

var s Slice[int] // 正确

type MyInt int
var s2 Slice[MyInt]  // MyInt底层类型是int，所以可以用于实例化

type MyMyInt MyInt
var s3 Slice[MyMyInt]  // 正确。MyMyInt 虽然基于 MyInt ，但底层类型也是int，所以也能用于实例化

type MyFloat32 float32  // 正确
var s4 Slice[MyFloat32]
```

`~`表示就代表着不光是 int ，所有以 int 为底层类型的类型也都可用于实例化。但是`~`也有一些限制：

* ~后面的类型不能为接口

* ~后面的类型必须为基本类型

嗯，这太符合go了。

go提供了两个内置的类型约束接口comparable

comparable表示支持`!=`和`==`操作，例如：

```go
// 错误。因为 map 中键的类型必须是可进行 != 和 == 比较的类型
type MyMap[KEY any, VALUE any] map[KEY]VALUE
type MyMap[KEY comparable, VALUE any] map[KEY]VALUE // 正确 
```

但是comparable不代表支持`> < >= <= `，这几个运算符表述为orderd，但是orderd不在go内置的，因此如果要用，需要自己实现一个，例如：

```go
// Ordered 代表所有可比大小排序的类型
type Ordered interface {
    Integer | Float | ~string
}

type Integer interface {
    Signed | Unsigned
}

type Signed interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64
}

type Unsigned interface {
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr
}

type Float interface {
    ~float32 | ~float64
}
```

### 4.2 Interface的理解

> 其实我觉的go的interface{}不应该称之为接口，更应该是一种类型。他和其他语言的接口定义显然不太相似，或者说他的定义更像是java接口的超集。例如下面常见的go的接口

现在我们再来看这个定义。在go1.18版本之后，go的interface{}的定义发生了如下的变化：

go version < 1.18

> An interface type specifies a **method set** called its interface

go version ≥ 1.18

> An interface type defines a **type set**

go的interface从方法集到类型集。

既然接口定义发生了变化，那么从Go1.18开始 `接口实现(implement)` 的定义自然也发生了变化：

当满足以下条件时，我们可以说 **类型 T 实现了接口 I ( type T implements interface I)**：

* T 不是接口时：类型 T 是接口 I 代表的类型集中的一个成员 (T is an element of the type set of I)
* T 是接口时： T 接口代表的类型集是 I 代表的类型集的子集(Type set of T is a subset of the type set of I)

既然是interface{}是集合，那么就有元素的交和并，例如：

```go
type AllInt interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 | ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint32
}

type Uint interface {
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
}

type A interface { // 接口A代表的类型集是 AllInt 和 Uint 的交集
    AllInt
    Uint
}

type B interface { // 接口B代表的类型集是 AllInt 和 ~int 的交集
    AllInt
    ~int
}type AllInt interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 | ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint32
}

type Uint interface {
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
}

type A interface { // 接口A代表的类型集是 AllInt 和 Uint 的交集
    AllInt
    Uint
}

type B interface { // 接口B代表的类型集是 AllInt 和 ~int 的交集
    AllInt
    ~int
}
```

上面这个例子中

* 接口 A 代表的是 AllInt 与 Uint 的 交集，即 ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64

* 接口 B 代表的则是 AllInt 和 ~int 的交集，即 ~int

既然interface{}代表了类型集合，那么interface{}是否代表空集合呢，显然不是的，interface{}我们知道是代表全部类型集合，嗯，这显然是因为interface{}是1.18版本之前就存在的，但是修改了interface的定义后，interface{}出现了歧义，那么解决方案是什么呢，一个新的关键字any代替了了interface{}，any成为了interface{}的alias，any就没有歧义了。太棒了，真是一个工工又程程的修改方式。 

### 4.3 接口的两种类型

看下面这个例子

```go
type ReadWriter interface {
    ~string | ~[]rune

    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
// 类型 StringReadWriter 实现了接口 Readwriter
type StringReadWriter string 

func (s StringReadWriter) Read(p []byte) (n int, err error) {
    // ...
}

func (s StringReadWriter) Write(p []byte) (n int, err error) {
 // ...
}

//  类型BytesReadWriter 没有实现接口 Readwriter
type BytesReadWriter []byte 

func (s BytesReadWriter) Read(p []byte) (n int, err error) {
 ...
}

func (s BytesReadWriter) Write(p []byte) (n int, err error) {
 ...
}
```

只有既满足类型约束还要实现对应的方法的才是实现了接口。为了解决这个问题也为了保持Go语言的兼容性，Go1.18开始将接口分为了两种类型

* 基本接口

* 一般接口

接口定义中如果只有方法的话，那么这种接口被称为**基本接口(Basic interface)**。这种接口就是Go1.18之前的接口，用法也基本和Go1.18之前保持一致。基本接口大致可以用于如下几个地方：

```go
type MyError interface { // 接口中只有方法，所以是基本接口
    Error() string
}

// 用法和 Go1.18之前保持一致
var err MyError = fmt.Errorf("hello world")
// io.Reader 和 io.Writer 都是基本接口，也可以用在类型约束中
type MySlice[T io.Reader | io.Writer]  []Slice
```

* 最常用的，定义接口变量并赋值
* 基本接口因为也代表了一个类型集，所以也可用在类型约束中

如果接口内不光只有方法，还有类型的话，这种接口被称为 **一般接口(General interface)** ，如下例子都是一般接口：

```go
type Uint interface { // 接口 Uint 中有类型，所以是一般接口
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
}

type ReadWriter interface {  // ReadWriter 接口既有方法也有类型，所以是一般接口
    ~string | ~[]rune

    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```

一般接口类型不能用来定义变量，只能用于泛型的类型约束中。所以以下的用法是错误的：

```go
type Uint interface {
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64
}

var uintInf Uint // 错误。Uint是一般接口，只能用于类型约束，不得用于变量定义
// 当然也不支持作为函数形参
type MyInterface interface {
    int | string
    Add()
    Print()
}

func Hello(a MyInterface) {
    a.Add()
    a.Print()
}

```

这一限制保证了一般接口的使用被限定在了泛型之中，不会影响到Go1.18之前的代码，同时也极大减少了书写代码时的心智负担，太抽象了这个解决方案。

### 4.4 接口定义中的泛型

所有类型的定义中都可以使用类型形参，所以接口定义自然也可以使用类型形参，观察下面这两个例子：

```go
type DataProcessor[T any] interface {
    Process(oriData T) (newData T)
    Save(data T) error
}

type DataProcessor2[T any] interface {
    int | ~struct{ Data interface{} }

    Process(data T) (newData T)
    Save(data T) error
}
```

因为引入了类型形参，所以这两个接口是泛型类型。而泛型类型要使用的话必须传入类型实参实例化才有意义。所以我们来尝试实例化一下这两个接口。因为 T 的类型约束是 any，所以可以随便挑一个类型来当实参(比如string)：

```go
DataProcessor[string]

// 实例化之后的接口定义相当于如下所示：
type DataProcessor[string] interface {
    Process(oriData string) (newData string)
    Save(data string) error
}
```

这是一个基本类型接口

```go
DataProcessor2[string]

// 实例化后的接口定义可视为
type DataProcessor2[T string] interface {
    int | ~struct{ Data interface{} }

    Process(data string) (newData string)
    Save(data string) error
}
```

这是一个一般类型接口，所以实例化之后的这个接口代表的意思是：

1. 只有实现了 `Process(string) string` 和 `Save(string) error` 这两个方法，并且以 `int` 或 `struct{ Data interface{} }` 为底层类型的类型才算实现了这个接口
2. **一般接口(General interface)** 不能用于变量定义只能用于类型约束，所以接口 `DataProcessor2[string]` 只是定义了一个用于类型约束的类型集

```go
// XMLProcessor 虽然实现了接口 DataProcessor2[string] 的两个方法，但是因为它的底层类型是 []byte，所以依旧是未实现 DataProcessor2[string]
type XMLProcessor []byte

func (c XMLProcessor) Process(oriData string) (newData string) {

}

func (c XMLProcessor) Save(oriData string) error {

}

// JsonProcessor 实现了接口 DataProcessor2[string] 的两个方法，同时底层类型是 struct{ Data interface{} }。所以实现了接口 DataProcessor2[string]
type JsonProcessor struct {
    Data interface{}
}

func (c JsonProcessor) Process(oriData string) (newData string) {

}

func (c JsonProcessor) Save(oriData string) error {

}

// 错误。DataProcessor2[string]是一般接口不能用于创建变量
var processor DataProcessor2[string]

// 正确，实例化之后的 DataProcessor2[string] 可用于泛型的类型约束
type ProcessorList[T DataProcessor2[string]] []T

// 正确，接口可以并入其他接口
type StringProcessor interface {
    DataProcessor2[string]

    PrintString()
}

// 错误，带方法的一般接口不能作为类型并集的成员(参考4.5接口定义的种种限制规则
type StringProcessor interface {
    DataProcessor2[string] | DataProcessor2[[]byte]

    PrintString()
}
```

### 4.5  接口的种种限制

1. 用 `|` 连接多个类型的时候，类型之间不能有相交的部分(即必须是不交集):
   
   ```go
    type MyInt int
    // 错误，MyInt的底层类型是int,和 ~int 有相交的部分
    type _ interface {
      ~int | MyInt
    }
   ```
   
   但是相交的类型中是接口的话，则不受这一限制：
   
   ```go
   type MyInt int
   
   type _ interface {
       ~int | interface{ MyInt }  // 正确
   }
   
   type _ interface {
       interface{ ~int } | MyInt // 也正确
   }
   
   type _ interface {
       interface{ ~int } | interface{ MyInt }  // 也正确
   }
   ```

2. 类型的并集中不能有类型形参
   
   ```go
   type MyInf[T ~int | ~string] interface {
    ~float32 | T  // 错误。T是类型形参
    }
   
    type MyInf2[T ~int | ~string] interface {
        T  // 错误
    }
   ```

3. 接口不能直接或间接地并入自己
   
   ```go
   type Bad interface {
       Bad // 错误，接口不能直接并入自己
   }
   
   type Bad2 interface {
       Bad1
   }
   type Bad1 interface {
       Bad2 // 错误，接口Bad1通过Bad2间接并入了自己
   }
   
   type Bad3 interface {
       ~int | ~string | Bad3 // 错误，通过类型的并集并入了自己
   }
   ```

4. 接口的并集成员个数大于一的时候不能直接或间接并入 `comparable` 接口，不知道为什么，很天才的想法，我猜是因为comparable是给编译器看的，如果支持的话，编译器比较难写
   
   ```go
   type OK interface {
       comparable // 正确。只有一个类型的时候可以使用 comparable
   }
   
   type Bad1 interface {
       []int | comparable // 错误，类型并集不能直接并入 comparable 接口
   }
   
   type CmpInf interface {
       comparable
   }
   type Bad2 interface {
       chan int | CmpInf  // 错误，类型并集通过 CmpInf 间接并入了comparable
   }
   type Bad3 interface {
       chan int | interface{comparable}  // 理所当然，这样也是不行的
   }
   ```

5. 带方法的接口(无论是基本接口还是一般接口)，都不能写入接口的并集中：带方法的接口(无论是基本接口还是一般接口)，都不能写入接口的并集中：
   
   ```go
   type _ interface {
       ~int | ~string | error // 错误，error是带方法的接口(一般接口) 不能写入并集中
   }
   
   type DataProcessor[T any] interface {
       ~string | ~[]byte
   
       Process(data T) (newData T)
       Save(data T) error
   }
   
   // 错误，实例化之后的 DataProcessor[string] 是带方法的一般接口，不能写入类型并集
   type _ interface {
       ~int | ~string | DataProcessor[string] 
   }
   
   type Bad[T any] interface {
       ~int | ~string | DataProcessor[T]  // 也不行
   }
   
   
   // 错误，间接引入也不可以
   type MyInterface interface {
   	int | string
   	Add()
   	Print()
   }
   
   type shell interface {
   	MyInterface
   }
   
   type MyInt interface {
   	~int | shell
   }
   ```




