## 前沿

今天我们来看看PHP5.3引入的生成器（`generator`），我们常常忽略了这个功能，其实这是非常有用的功能。



## 创建生成器

生成器的创建方式很简单，因为生成器就是PHP函数，只不过要在函数中一次或多次使用 `yield` 关键字。

与普通的PHP函数不同的是，**生成器从不返回值，只产出值**。

```
//一个简单的生成器
function myGenerator() {
        yield 'value1';
        yield 'value2';
        yield 'value3';
}
```

上面就是一个简单的生成器，调用生成器函数时，PHP会返回一个属于 `Generator` 类的对象。这个对象可以使用 `foreach()` 函数来迭代。每次迭代，PHP会要求 `Generator` 实例计算并提供下一个要迭代的值。

```
//使用foreach() 来迭代
foreach (myGenerator() as $yieldValue) {
    echo $yieldValue, PHP_EOL;
}
//输出
value1
value2
value3
```

如上：每一次产出一个值之后， 生成器的内部状态都会停顿；向生成器请求下一个值时，内部状态又会恢复。**生成器的内部状态会一直在停顿和恢复之间切换**，直到抵达函数定义体的末位或遇到空的`return`;语句为止。



## 使用生成器

下面我们以一个简单的函数，用于生成一个范围内的数值，以此说明PHP 生成器是**如何节省内存的**。

```
<?php
function makeRangeYield($length) {
    //$dataset = [];
    for ($i = 0; $i < $length; $i++) {
        yield $i;
       // $dataset[] = $i;
    }
    //return $dataset;
}
//使用生成器
$startMemory = memory_get_usage();
$t1 = microtime(true);
$customRangeYield = makeRangeYield(1000000);
//结束
$t2 = microtime(true);
$endMemory = memory_get_usage();
foreach ($customRangeYield as $i) {
    echo $i . PHP_EOL;
}
//输出
echo sprintf("内存使用: %f kb\n", ($endMemory - $startMemory) / 1024);
echo sprintf("耗时： %f秒\n", round($t2-$t1,3));
```

我们做个对照比较，大家可以把makeRangeYield() 函数里面的注释打开，和没打开做个对比

```
// 使用yield的情况
内存使用: 0.562500 kb
耗时： 0.000000秒
// 没有使用yield的情况
内存使用: 32772.078125 kb
耗时： 0.205000秒
```

因为makeRangeYield()函数要为预先创建的一个由一百万个整数组成的数组分配内存。PHP生成器能实现相同的操作，**不过一次只会为一个整数分配内存**。



## 实战

生成器这么强，平常工作中用在哪里呢？我们来看个读取大文件的例子

```
<?php
header("content-type:text/html;charset=utf-8");
function readTxt()
{
    # code...
    $handle = fopen("./test.txt", 'rb');

    while (feof($handle)===false) {
        # code...
        yield fgets($handle);
    }

    fclose($handle);
}

foreach (readTxt() as $key => $value) {
    # code...
    echo $value . PHP_EOL;
}
```

假设我们的`test.txt`文件有4G，我们敢直接把它读入内存吗？肯定是不合适的，我们因该使用生成器一次只为`test.txt`文件中一行分配内存，而不是把整个4G的文件读取到内存。

## 致谢

感谢你看完这篇文章，有什么不对的地方欢迎指出，谢谢🙏