# 数据

- [数据](#数据)
  - [基础类型](#基础类型)
    - [字符串](#字符串)
  - [引用类型](#引用类型)
    - [数组](#数组)
    - [切片](#切片)
    - [哈希表](#哈希表)
    - [结构体](#结构体)

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

### 哈希表

Go语言中，哈希表是一个无序的key/value对的集合，其中所有的key都是不同的，然后通过给定的key可以在常数时间复杂度内检索、更新或删除对应的value。其中key必须是支持==比较运算符的数据类型。

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

### 结构体

Go语言中，结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。其中，每个值称为结构体的成员。
