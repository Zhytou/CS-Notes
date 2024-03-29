# 数据

- [数据](#数据)
  - [基础类型](#基础类型)
    - [字符串](#字符串)
  - [引用类型](#引用类型)
    - [数组](#数组)
    - [切片](#切片)
    - [哈希表](#哈希表)
    - [结构体](#结构体)
  - [其他类型](#其他类型)
    - [优先队列](#优先队列)
    - [栈](#栈)
    - [队列](#队列)

Go语言将数据类型分为四类：

- 基础类型：数字、字符串和布尔型；
- 复合类型：数组和结构体；
- 引用类型：切片、字典、函数和通道；
- 接口类型：一组方法的集合。

## 基础类型

### 字符串

Go语言中，一个字符串是一个不可改变的字节序列。

**不可变性**：

字符串的值是不可变的：一个字符串包含的字节序列永远不会被改变，当然我们也可以给一个字符串变量分配一个新字符串值。除此之外，追加一个字符串也是允许的。

不变性意味如果两个字符串共享相同的底层数据的话也是安全的，这使得复制任何长度的字符串代价是低廉的。

**byte vs rune**：

byte是Go语言中的内置数据类型，它是一个8位无符号整数类型，可以用来表示ASCII编码字符集中的字符。

rune是Go语言中的另一个内置数据类型，它是一个32位有符号整数类型，可以用来表示Unicode编码字符集中的字符。

**字符串遍历**：

值得注意的是，使用索引循环遍历和关键字`range`遍历访问到的的值类型是不同的，前者是`byte`后者是`rune`。

``` golang
for i := 0; i < len(nums); i++ {
    // type of nums[i] is byte
    var num byte = nums[i]
}

for _, ch := range(nums) {
    // type of ch is rune
}
```

**字符串 & byte切片 & rune切片**：

由于Go语言字符是不可变的，当遇到需要操作字符串时，可以使用byte切片或rune切片代替，只需要在最后将其转换为字符串类型即可。

例如，在N皇后问题中需要操作字符串切片，我们可以将其替换成rune切片。

``` go
func main() {
    n := 4
    board := make([][]rune, n)
    for i := range board {
        board[i] = make([]rune, n)
        for j := range board[i] {
            board[i][j] = '.'
        }
    }

    var solutions [][]string
    backtrack(&solutions, board, 0)

    fmt.Println(solutions)
}

func backtrack(solutions *[][]string, board [][]rune, row int) {
    if row == len(board) {
        var solution []string
        for _, row := range board {
            solution = append(solution, string(row))
        }
        *solutions = append(*solutions, solution)
        return
    }

    for col := 0; col < len(board); col++ {
        if isValid(board, row, col) {
            board[row][col] = 'Q'
            backtrack(solutions, board, row+1)
            board[row][col] = '.'
        }
    }
}

func isValid(board [][]rune, row, col int) bool {
    n := len(board)

    // 检查列
    for i := 0; i < row; i++ {
        if board[i][col] == 'Q' {
            return false
        }
    }

    // 检查左上角到右下角的对角线
    for i, j := row-1, col-1; i >= 0 && j >= 0; i, j = i-1, j-1 {
        if board[i][j] == 'Q' {
            return false
        }
    }

    // 检查右上角到左下角的对角线
    for i, j := row-1, col+1; i >= 0 && j < n; i, j = i-1, j+1 {
        if board[i][j] == 'Q' {
            return false
        }
    }

    return true
}
```

**字符串操作**：

Go语言主要提供了`strings`和`strconv`两个包用于操作字符串。其中，`strings`包适用于字符串的基本操作，而`strconv` 包适用于将字符串转换为数值类型和反之的操作。

此外，我们还可能会用到如：`bytes`包：提供用于操作字节切片的函数和方法，可以用于处理二进制数据、大型文本文件等；`fmt`包：提供了格式化输出的功能，可以将数据格式化为字符串并输出到控制台或文件。

简单介绍一下常用函数：

``` golang
// 返回子字符串 substr 在字符串 s 中第一次出现的索引。
// 如果 substr 未找到，则返回 -1。
strings.Index(s, substr string) int


// 返回一个新的字符串，其中前 n 个旧字符串 old 被替换为新字符串。
// 参数 s 是原始字符串，参数 old 是要被替换的子串，参数 new 是替换后的新子串，参数 n 是指定替换的次数。如果 n 的值为 -1，则会替换所有出现的子串。
strings.Replace(s, old, new string, n int) string

// 将字符串切片中的所有字符串连接起来，生成一个新的字符串，并返回这个新字符串。
// 参数 a 是要连接的字符串切片，参数 sep 是分隔符。
strings.Join(a []string, sep string) string

// 将 int 类型的值转换为字符串类型
strconv.Itoa(a int) string

// 将字符串类型的整数转换为 int 类型的值
strconv.Atoi(s string) int
```

下面举例说明常见的字符串操作：

``` golang
// 切片转字符串
nums := []int{1, 2, 3, 4, 5}
ss := make([]string, len(nums))
for i, num := range nums {
    ss[i] = strconv.Itoa(num)
}
str := strings.Join(ss, "")

// 切片转字符串
nums := []int{1, 2, 3, 4, 5}
str = fmt.Sprint(nums)
```

## 引用类型

**声明 & 初始化**：

对于除数组之外的引用类型，如切片、字典等，使用`var`声明了变量之后，仍需要使用`make`为其分配内存空间，否则不能执行插入和访问等操作。

**new vs make**：

使用new函数仅仅是分配了内存空间，并没有对该内存空间进行初始化

在使用make函数创建map变量时，可以指定map的初始容量，即预分配的map元素数量。

### 数组

Go语言中，数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。

**定义**:

- 默认情况下，数组的每个元素都被初始化为元素类型对应的零值，对于数字类型来说就是0。
- 如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算

``` golang
// 声明一个长度为3的整形数字数组 
var a [3]int
// 定义一个长度为3的形数字数组，其内容为1，2，3
a := [...]int{1, 2, 3}
// 定义了一个长度为100，序号为99位置为-1的整形数字数组
a := [...]int{99: -1}
```

**遍历**：

值得注意的是，在 Go 语言中，使用 range 遍历切片时，每次迭代返回的是当前元素的值的一个副本，而不是它的引用。

``` golang
// 使用索引遍历
for i := 0; i < len(a); i++ {
  //
}
// 使用range关键字遍历
for i, v := range a {
  //
}
```

### 切片

在Go语言中，切片代表变长的序列，序列中每个元素都有相同的类型。

一个切片由三个部分构成：

- 指针：指针指向第一个slice元素对应的底层数组元素的地址；
- 长度：长度对应slice中元素的数目；
- 容量：容量一般是从slice的开始位置到底层数据的结尾位置。

因此，若向函数传递slice将允许在函数内部修改底层数组的元素。换句话说，复制一个slice只是对底层的数组创建了一个新的slice别名。此外，多个切片之间可以共享底层的数据，并且引用的数组部分区间可能重叠。

和数组不同的是，slice之间不能比较，因此我们不能使用`==`操作符来判断两个slice是否含有全部相等元素。

**定义**：

``` golang
// 定义并初始化一个长度为 3 的 int 类型切片
a := []int{0, 1, 2, 3, 4, 5}
// 定义一个初始长度为 0，容量为 5 的 int 类型切片
slice := make([]int, 0, 5)
```

**插入 & 删除**：

Go语言内置的`append`函数用于向切片追加元素，如：`arr = append(arr, num)`。

事实上，每次调用append函数都会检测切片底层数组是否有足够容量来保存新添加的元素。

此外，切片的删除元素也是依靠`append`函数来实现的，例如：

``` go
// 声明一个切片
s := []int{1, 2, 3, 4, 5}

// 删除索引为2的元素
index := 2
s = append(s[:index], s[index+1:]...)
```

*注意*：由于`append`函数可能会导致底层指针发生变化，因此，尽管切片是一个引用类型，但当前做参数传入某函数，且该函数内调用了`append`函数，应当将传入的切片修改为指针类型。对于其他引用数据类型，如：哈希表也是同理。

**排序**：

Go语言提供了常见切片类型的排序方法，包括：

- []int 切片的排序方法：`sort.Ints`
- []float64 切片的排序方法：`sort.Float64s`
- []string 切片的排序方法：`sort.Strings`

``` golang
// 定义一个 []int 切片
nums := []int{3, 5, 2, 8, 1, 9}

// 使用 sort 包对切片进行排序
sort.Ints(nums)
```

此外，Go语言还定义了[]interface{}切片的排序方法`sort.Slice`。使用`sort.Slice`函数可以向其传入自定义的比较函数。

``` golang
// 定义一个 []interface{} 切片
values := []interface{}{3, 1, 4, 1, 5}

// 使用 sort 包和自定义比较函数对切片进行排序
sort.Slice(values, func(i, j int) bool {
    return values[i].(int) < values[j].(int)
})
```

最后，针对一个自定义类型的切片进行排序，只需要使用`sort.Sort`函数并实现该类型的`sort.Interface`接口方法即可。其中`sort.Interface`如下：

``` golang
type Interface interface {
    Len() int           // 获取数据的长度
    Less(i, j int) bool // 对数据进行比较
    Swap(i, j int)      // 交换数据
}
```

``` golang
// 定义Person类型
type Person struct {
    name string
    age  int
}

type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].age < a[j].age }

// 定义一个 []Person 切片
people := []Person{
    {"Alice", 25},
    {"Bob", 30},
    {"Charlie", 20},
    {"David", 35},
}

// 使用 sort 包对切片进行排序
sort.Sort(ByAge(people))
```

### 哈希表

Go语言中，哈希表是一个无序的key/value对的集合，其中所有的key都是不同的，然后通过给定的key可以在常数时间复杂度内检索、更新或删除对应的value。其中key必须是支持==比较运算符的数据类型。

*值得注意的是*map的每次遍历顺序都是不同的。此外，切片和数组均不能作为哈希表键值。

**定义**：

``` golang
// 定义一个空的 map，键和值的类型都是 string
var m map[string]string

// 定义并初始化一个 map，键和值的类型都是 string
m := map[string]string{
    "key1": "value1",
    "key2": "value2",
}
// 使用 make() 函数定义一个空的 map，键和值的类型都是 string
m := make(map[string]string)
```

**哈希set**：

Go语言中没有定义想C++中`set`的数据结构，而是使用`map[type]bool`来实现该结构。

**查找 & 访问**：

要查找一个哈希表中是否存在某个键，可以使用`访问操作符[]`来访问其中的元素，如果该键不存在，则返回该键类型的零值。

``` go
// 定义一个哈希表
var m map[rune]bool
// 判断键值是否存在
if val, ok := m[key]; ok {
  // ... 
}
```

**插入 & 删除**：

此外，在分配内存之后，哈希表还可以使用`访问操作符[]`来插入新键值。

至于，删除键值，则需要使用内置函数`delete`。

``` go
// 插入键值对（“abc",1)
m["abc"] = 1

// 删除键值对（“abc",1)
delete(m, "abc")
```

**自定义类型作为键**：

在 Go 语言中，自定义类型可以作为 map 的键类型，这需要满足以下条件：

- 自定义类型必须支持相等运算符 ==，否则无法判断两个键是否相等。
- 自定义类型不能是函数类型、映射类型或切片类型，否则会导致编译错误。

在满足上述条件的情况下，只需要实现以下两个函数即可。

``` golang
func (t T) Equals(other T) bool

func (t T) HashCode() int
```

### 结构体

Go语言中，结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。其中，每个值称为结构体的成员。

**成员访问**：

结构体变量的成员可以通过点操作符访问。

此外，点操作符也可以和指向结构体的指针一起工作。这一点与C++中使用`->`箭头符非常不一样需要注意。

**导出字段**：

如果结构体成员名字是以大写字母开头的，那么该成员就是导出的；这是Go语言导出规则决
定的。一个结构体可能同时包含导出和未导出的成员。

**递归包含**：

一个结构体不能包含自身，因为一个聚合的值不能包含它自身。

但是某结构体可以包含该指结构体的针类型的成员，这可以允许程序员创建递归的数据结构，比如链表和树结构等。

**结构体面值**：

结构体值也可以用结构体面值表示，结构体面值可以指定每个成员的值。

第一种写法，要求以结构体成员定义的顺序为
每个结构体成员指定一个面值。

``` golang
// 第一种面值写法
type Point struct{ X, Y int }
p := Point{1, 2}
```

其实更常用的是第二种写法，以成员名字和相应的值来初始化，可以包含部分或全部的成员。

``` golang
// 第二种面值写法
type Point struct{ X, Y int }
p := Point{X: 1, Y: 2}
```

此外，需要注意的是：如果一个结构体的字段是未导出的，那么它的值不能使用面值赋值。

**JSON**：

在Go语言中，可以使用struct的标签（tag）来指定JSON序列化和反序列化时的字段名称、类型和其他配置选项。标签是struct字段后跟一个tag字符串，该字符串被包含在反引号中。例如：

``` golang
type Person struct {
    Name   string `json:"name"`
    Age    int    `json:"age"`
}
```

## 其他类型

Go语言中并没有直接提供如优先队列、栈和队列等常见的数据结构，但我们可以在切片的基础上实现这些结构。

### 优先队列

``` golang
type Item struct {
    value    interface{} // 值
    priority int         // 优先级
    index    int         // 在堆中的索引
}

type PriorityQueue []*Item

func (pq PriorityQueue) Len() int { return len(pq) }

func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].priority < pq[j].priority
}

func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i
    pq[j].index = j
}

func (pq *PriorityQueue) Push(x interface{}) {
    n := len(*pq)
    item := x.(*Item)
    item.index = n
    *pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    item.index = -1 // for safety
    *pq = old[0 : n-1]
    return item
}

func (pq *PriorityQueue) update(item *Item, value interface{}, priority int) {
    item.value = value
    item.priority = priority
    heap.Fix(pq, item.index)
}

pq := make(PriorityQueue, 0)
heap.Init(&pq)

// 添加元素
heap.Push(&pq, &Item{value: "foo", priority: 1})
heap.Push(&pq, &Item{value: "bar", priority: 2})

// 弹出元素
item1 := heap.Pop(&pq).(*Item)
fmt.Println(item1.value) // 输出 "bar"

// 更新元素
item2 := &pq[0] // 获取第一个元素
pq.update(item2, item2.value, 3)
```

### 栈

``` golang
type Stack []interfice {}

func (stack Stack) Len() int {
    return len(stack)
}

func (stack Stack) Cap() int {
    return cap(stack)
}

func (stack *Stack) Push(value interface{})  {
    *stack = append(*stack, value)
}

func (stack *Stack) Pop() (interface{}, error)  {
    theStack := *stack
    if len(theStack) == 0 {
        return nil, errors.New("Out of index, len is 0")
    }
    value := theStack[len(theStack) - 1]
    *stack = theStack[:len(theStack) - 1]
    return value, nil
}

func (stack Stack) Top() (interface{}, error)  {
    if len(stack) == 0 {
        return nil, errors.New("Out of index, len is 0")
    }
    return stack[len(stack) - 1], nil
}
```

### 队列

``` golang
type Queue []interfice {}

func (queue Queue) Len() int {
    return len(queue)
}

func (queue Queue) Cap() int {
    return cap(queue)
}

func (queue *Queue) Push(value interface{})  {
    *queue = append(*queue, value)
}

func (queue *Queue) Pop() (interface{}, error)  {
    theQueue := *queue
    if len(theQueue) == 0 {
        return nil, errors.New("Out of index, len is 0")
    }
    value := theQueue[0]
    *queue = theQueue[1:]
    return value, nil
}

func (queue Queue) Front() (interface{}, error)  {
    if len(stack) == 0 {
        return nil, errors.New("Out of index, len is 0")
    }
    return queue[0], nil
}
```
