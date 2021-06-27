##### 包

包名与导入路径的最后一个元素一致。

举例：

​	`import ("math/rand")`   使用： rand.Intn(10)

##### 注意点

- 字符串用双引号
- 参数定义 a(x, y int)      类型写后面
- := 简洁赋值语句 可以在类型明确的情况下替代 var 声明
- 没有明确初始化的变量声明会被赋予它们的零值 如：数值类型为0
- 类型的转换是显示转换。如：i := 42    f := float64(i)
- 类型推导：在声明一个变量而不指定其类型时（即使用不带类型的:= 语法或var = 表达式语法），变量的类型由右键推导得出  %T 打印查看
- 常量的声明与变量类似，只不过是使用 const 关键字，常量可以是字符，字符串，布尔值或数值。常量不能用:=语法声明 举例 const Pi  float64= 3.14
- 数值常量是高精度的**值**。一个未指定类型的常量由上下文来决定其类型



##### 基本类型

```
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名

rune // int32 的别名
    // 表示一个 Unicode 码点

float32 float64

complex64 complex128
```



##### go 流程控制

- Switch 的case 无需为常量，且取值不必为整数。

```
switch i {
	case 0:
	case f():
}
在 i == 0的时候 f不会被调用
```

- defer 语句会将函数推迟到外层函数返回之后执行。理解为栈就可以了

##### 结构体

- 结构体文法通过直接列出字段的值来新分配一个结构体。使用Name：语法可以仅列出部分字段（字段名的顺序无关）

```
type Vertex struct {
	X, Y int
}

var (
	v1 = Vertex{1, 2}  // 创建一个 Vertex 类型的结构体 {1, 2}
	v2 = Vertex{X: 1}  // Y:0 被隐式地赋予//{1 0}
	v3 = Vertex{}      // X:0 Y:0//{0 0}
	p  = &Vertex{1, 2} // 创建一个 *Vertex 类型的结构体（指针）//&{1 2}
)

func main() {
	fmt.Println(v1, p, v2, v3)
}
```

- `var a [10]int`  会将变量a声明为拥有10个整数的数值
- ==go中字符串不要用单引号啦！！==

- 切片：a[1:4] 表达式创建了一个切片，它包含a中下标从 **1到3**的元素

##### 切片就像数组的引用

- 切片并不存储任何数据，它只是描述了底层数组中的一段
- 更改切片的元素会修改其底层数组中对应的元素
- 与它共享底层数组的切片都会观测到这些修改
- 切片的默认行为：切片下界的默认值为0，上界则是该切片的长度
- 切片的长度 len(s)  切片的容量 cap(s) :从第一个元素开始数，到其底层数组元素末尾的个数
- 切片的零值是 nil, 切片的长度和容量为0且没有底层数组
- 使用make 创建切片 `a := make([]int, 0, 5)` 长度为0 容量为5
- 切片可以包含任何类型，甚至包括其他的切片

##### 二维数组切片

```
s := []struct {
		i int
		b bool
	}{
		{2, true},
		{3, false},
		{5, true},
		{7, true},
		{11, false},
		{13, true},
	}
	fmt.Println(s)
	//[{2 true} {3 false} {5 true} {7 true} {11 false} {13 true}]
	
	
	// 创建一个井字板（经典游戏）
	board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// 两个玩家轮流打上 X 和 O
	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"
X _ X
O _ X
_ _ O

```

数组的初始化

```
//两种方式
 var arr = []int{}
 arr := []int{}
```

Printf 打印说明 %s 字符串 %d 整型 %v 值 %T 类型 %p打印地址 %q打印字符

##### range

Range for循环的range形式可遍历切片或映射

```
pow := make([]int, 10)
for i := range pow {
		pow[i] = 1 << uint(i)
}
```

##### 映射

映射将健映射到值。

映射的零值为nil。 nil 映射既没有健，也不能添加健

make 函数会返回给定类型的映射，并将其初始化备用 `make(map[string]int)`  ==string:代表参数类型(key)==，int 代表返回类型(value)

```
type Vertex struct {
		Lat, Long float64
}
var m map[string]Vertex
m = make(map[string]Vertex)

```

- 映射的文法与结构体相似，不过必须有健名
- 若顶级类型只是一个类型名，你可以在文法的元素中省略它

修改映射

```
m := make(map[string]int)
m["hu"] = 23
delete(m, "hu")  // 0//删除键值
v, ok := m["hu"] // 0， false
若key在m中，ok为true；否则为false
若key不在映射中，那么v是该映射元素类型的零值
同样的，当从映射中读取某个不存在的健时，结果是映射的元素类型的零值
```

##### 函数值

- 函数也是值。它们可以像其他一样传递

- 函数值可以用作**函数的参数或返回值**

    ```
    func compute(fn func(float64, float64) float64) float64 {
    		return fn(3,4)
    }
    ```

##### 函数的闭包

Go函数可以是一个闭包。闭包是一个函数值，它引用了其函数体之外的变量。该函数可以访问并赋予其引用的变量的值，换句话说，该函数被这些变量“绑定”在一起

```
func adder() func(int) int {
		sum := 0;
		return func(x int) int {
				sum += x
				return sum 
		}
}
```



## 一些疑问

1. 如果make创建切片容量很大，长度为0呢？

```
pow := make([]int, 0, 10)
//pow[2] = 1   这样会报错，长度为0的时候数组为[]空数组
fmt.Println(pow) 
```





##### 方法与指针重定向

==记住一个点，如果参数没有返回值要修改参数，必须是指针为接收者方法==

==有返回值函数还是方法都的加返回类型，没有返回值可以不用加==

- 指针参数的函数必须接受一个指针

    ```
    var v *Vertex
    ScaleFunc(v, 5)  // 编译错误！
    ScaleFunc(&v, 5) // OK
    ```

    

- 而以指针为接收者的方法被调用时，接收者即能为值、又能为指针

```
var v *Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK

//Go 会将语句 v.Scale(5) 解释为 (&v).Scale(5)。
但是你不能 p.Scale(10) 在写成 (&p).Scale(10) 了这样会报错，显示调用了，不用在加了，这里将p.Scale(10) 会被解释为 (*p).Scale(10) 因为这时的(*p)就代表着地址
```

##### 方法

- go 没有类。不过你可以为结构体类型定义方法。
- 方法就是一类带特殊的----**接收者**----参数的**函数**。
- 方法即函数

##### 选择值或指针作为接收者

- 首先，方法能够修改其他接收者指向的值
- 其次，这样可以避免在每次调用方法时复制改值。若值的类型为大型结构体时，这样做会更加高效

##### 接口

- **接口类型** 是由一组方法签名定义的集合

##### 接口与隐式实现

- 类型通过实现一个接口的所有方法来实现该接口

    ```
    type I interface {
    		M()
    }
    type T struct {
    		S string
    }
    //此方法表示类型T 实现了接口I。但我们无需显示声明此事
    func (t T) M() {
    		fmt.Println(t.s)
    }
    ```

    

### type用法

1. 类型等价定义，相当于类型重命名

    ```
    （举例 type name string   ---> name类型与string等价）
    var myname name = "taoz"
    ```

2. 结构体内嵌匿名成员

    ```
    //结构体内嵌匿名成员定义
    type person struct {
    
    string  //直接写类型，匿名
    
    age int
    }
    func main() {
    
    //结构体匿名成员初始化
    
    p := person{string: "taozs", age: 18}//可以省略部分字段，如：person{string: "taozs"}。也可以这样省略字段名：person{“taozs”, 18}，但必须写全了，不可以省略部分字段
    
    //结构体匿名成员访问
    
    fmt.Println(p.string) //注意不能用强制类型转换（类型断言）：p.(string)
    
    }
    
    ```

    