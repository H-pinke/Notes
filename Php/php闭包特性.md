## 前沿

今天我们来聊聊PHP5.3引入的闭包特性



## 定义

闭包是指在创建时封装**周围的状态**的函数。即便闭包所在的环境不存在了，闭包中封装的状态依然存在。

**有人可能会问闭包和匿名函数有什么区别。**

其实理论上讲，闭包和匿名函数是不同概念。不过PHP将其视为相同的概念。所以，提到闭包时，指的也是匿名函数。

闭包和匿名函数其实是伪装成函数的**对象**，如果审查PHP闭包或匿名函数，会发现它们是**Closure**类的实例。

## 

## 创建闭包

来看个例子

```
//创建闭包
$test = function($name) {
    return "hello $name\n";
};

echo $test('hpinke');//输出 hello hpinke
```

简单的来说就是创建了一个闭包对象，然后将其赋值给$test变量。闭包和普通的PHP函数很像：使用的句法相同，也接受参数，而且能返回值。

我们通常把PHP 闭包当做函数和方法的回调使用。很多PHP函数都会用到回调函数 比如array_map

```
$resultArr = array_map(function($num) {
    return $num*3;
}, [1,2,3,4]);

print_r($resultArr);//输入 3 6 9 12		
```

在没有引入闭包之前，只能单独创建具名函数，然后使用名称引入那个函数。



## 附加状态

在PHP中，必须手动调用闭包对象的bindTo()方法或者使用use关键字，把状态附加到PHP闭包上。

一、使用use关键字

```
function testPerson($name) {
    return function ($domain) use ($name) {
        echo "$domain --- $name";
    };
}

$name = testPerson('hpinke');

echo $name('hello world');
//输出 hello world --- hpinke

```

函数testPerson() 有个名为`$name`的参数，这个函数返回一个闭包对象，而且这个闭包封装了`$name` 参数，即便返回的闭包对象跳出了testPerson() 函数的作用域，它也会记住`$name` 参数的值，因为`$name`变量仍在闭包中。



二、使用binTo关键字

**PHP 闭包是对象**。与任何其他PHP对象类似，每个闭包实例都可以使用 $this 关键字获取闭包的内部状态。

我们可以使用binTo这个方法把 **Closure**对象的内部状态绑定到其他对象上。

```
//动态的给一个对象 添加方法

class Test {
    private $num = 1;

}

$f = function() {
    return $this->num + 1;
};

$test1 = $f->bindTo(new Test, 'Test');
echo $test1(); //输出 2
```

我们访问的`$num`是私有变量，所以作用域指定对象，或者类名。



## 致谢

感谢你看完这篇文章，有什么不对的地方欢迎指出，谢谢🙏