# 基础

### 定义变量

```go
package main

import (
	"errors"
	"fmt"
)

// 定义变量
// 单个变量  // var variableName type
//var v1 string

//定义并初始化
var v1 string = "v1_value"

// 多个变量 // var vname1, vname2, vname3 type
var name, age, sex string = "zhao", "20", "boy"

// 定义常量
const v2 = 100
const v3 float32 = 3.1415926

// bool
var isActive bool

// 字符串 Go中的字符串都是采用UTF-8字符集编码。字符串是用一对双引号（""）或反引号（` `）括起来定义
var frenchHello string

func main() {
	// 简短声明 只能在函数中使用
	date, month, year := 14, 9, 2020
	// _ 特殊变量，任何赋值都会被丢弃
	_, b := 100, 1000
	// bool
	isActive = true
	// string
	no, yes, maybe := "no", "yes", "maybe" // 简短声明，同时声明多个变量
	japaneseHello := "Konichiwa"           // 同上
	frenchHello = "Bonjour"                // 常规赋值
	// 字符串默认不可变，改变方法
	s := "hello"
	c := []byte(s)
	c[0] = 'c'
	s2 := string(c)
	fmt.Printf("s2:%v\n", s2)
	// 方法2
	s3 := "hello"
	s4 := "c" + s3[1:] // 字符串虽不能更改，但可进行切片操作
	fmt.Printf("%s\n", s4)
	// 错误类型
	err := errors.New("emit macho dwarf: elf header corrupted")
	if err != nil {
		fmt.Print(err)
	}
	fmt.Printf("v1 %v\n", v1)
	fmt.Printf("name: %v; age: %v ;sex: %v \n", name, age, sex)
	fmt.Printf("%v%v%v \n", year, month, date)
	fmt.Printf("%v\n", b)
	fmt.Printf("%v\n", v3)
	fmt.Printf("%v", isActive)

	fmt.Printf("%v%v%v \n", no, yes, maybe)
	fmt.Printf("%v\n", japaneseHello)
	fmt.Printf("%v\n", frenchHello)

	// iota 枚举
	const (
		x = iota // x == 0
		y = iota // y == 1
		z = iota // z == 2
		w        // 常量声明省略值时，默认和之前一个值的字面相同。这里隐式地说w = iota，因此w == 3。其实上面y和z可同样不用"= iota"
	)
	fmt.Print(x, y, z, w)
}

```

### Go程序设计的一些规则

#### Go之所以会那么简洁，是因为它有一些默认的行为：

* **大写字母开头的变量是可导出的，也就是其它包可以读取的，是公有变量；小写字母开头的就是不可导出的，是私有变量。**
* **大写字母开头的函数也是一样，相当于`class`中的带`public`关键词的公有函数；小写字母开头的就是有`private`关键词的私有函数。**

### array、slice、map

```go
package main

import (
	"fmt"
)

// array 数组
var arr [10]int

func main() {
	fmt.Printf("hello world")
	arr[0] = 100
	arr[1] = 101
	fmt.Printf("%d\n", arr)

	a := [3]int{1, 2, 3}   // 声明了一个长度为3的int数组
	b := [10]int{1, 2, 3}  // 声明了一个长度为10的int数组，其中前三个元素初始化为1、2、3，其它默认为0
	c := [...]int{4, 5, 6} // 可以省略长度而采用`...`的方式，Go会自动根据元素个数来计算长度

	fmt.Printf("a:%v\n,b:%b\n,c:%v\n", a, b, c)

	// 声明了一个二维数组，该数组以两个数组作为元素，其中每个数组中又有4个int类型的元素
	doubleArray := [2][4]int{[4]int{1, 2, 3, 4}, [4]int{5, 6, 7, 8}}
	// 上面的声明可以简化，直接忽略内部的类型
	easyArray := [2][4]int{{1, 2, 3, 4}, {5, 6, 7, 8}}
	fmt.Printf("%v%v", doubleArray, easyArray)

	// slice
	// 声明一个数组
	// var array = [10]byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
	// 声明两个slice
	//var aSlice, bSlice []byte

	// // 演示一些简便操作
	// aSlice = array[:3] // 等价于aSlice = array[0:3] aSlice包含元素: a,b,c
	// aSlice = array[5:] // 等价于aSlice = array[5:10] aSlice包含元素: f,g,h,i,j
	// aSlice = array[:]  // 等价于aSlice = array[0:10] 这样aSlice包含了全部的元素

	// // 从slice中获取slice
	// aSlice = array[3:7]  // aSlice包含元素: d,e,f,g，len=4，cap=7
	// bSlice = aSlice[1:3] // bSlice 包含aSlice[1], aSlice[2] 也就是含有: e,f
	// bSlice = aSlice[:3]  // bSlice 包含 aSlice[0], aSlice[1], aSlice[2] 也就是含有: d,e,f
	// bSlice = aSlice[0:5] // 对slice的slice可以在cap范围内扩展，此时bSlice包含：d,e,f,g,h
	// bSlice = aSlice[:]   // bSlice包含所有aSlice的元素: d,e,f,g

	//slice是引用类型，所以当引用改变其中元素的值时，其它的所有引用都会改变该值，例如上面的aSlice和bSlice，如果修改了aSlice中元素的值，那么bSlice相对应的值也会改变。

	// map
	var numbers map[string]int

	numbers = make(map[string]int)
	numbers["one"] = 1  //赋值
	numbers["ten"] = 10 //赋值
	numbers["three"] = 3

	fmt.Println("第三个数字是: ", numbers["three"])
}

```

#### 使用map过程中需要注意的几点：

* `map`是无序的，每次打印出来的`map`都会不一样，它不能通过`index`获取，而必须通过`key`获取
* `map`的长度是不固定的，也就是和`slice`一样，也是一种引用类型
* 内置的`len`函数同样适用于`map`，返回`map`拥有的`key`的数量
* `map`的值可以很方便的修改，通过`numbers["one"]=11`可以很容易的把key为`one`的字典值改为`11`
* `map`和其他基本型别不同，它不是thread-safe，在多个go-routine存取时，必须使用mutex lock机制

#### make & new ??????
