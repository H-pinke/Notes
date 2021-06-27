## 前沿

上回我们讨论了PHP的生成器，今天我们来看看PHP的迭代器。



## 区别

生成器和迭代器有什么区别呢？

**一句话：生成器只是简单的迭代器。**

与标准的PHP迭代器不同，PHP生成器不要求类实现`Iterator`接口，从而减轻了类的负担。生成器会根据需求计算并产出要迭代的值。这对应用的性能有重大影响。试想一下，假如标准的PHP迭代器经常在内存中执行迭代操作，这要预先计算出数据集，性能低下；如果要使用特定的方式计算大量数据，对性能的影响更甚。此时我们可以使用生成器，即使计算并产出后续值，不占用宝贵的内存资源。



## 作用

迭代器有什么作用呢？

**让对象变得可迭代并表现得像对象集合。**

我们来看一道**腾讯面试题**

> 使对象可以像数组一样进行foreach循环，要求属性必须是私有。

这题其实就是考察到了迭代器了

我们先看看迭代器的接口

```
Iterator extends Traversable {
/* 方法 */
  abstract public current ( ) : mixed
  abstract public key ( ) : scalar
  abstract public next ( ) : void
  abstract public rewind ( ) : void
  abstract public valid ( ) : bool
}
```

里面定义了5个抽象方法

来看看最后的答案

```
class sample implements Iterator
{
    private $_items = array(1,2,3,4,5,6,7);
    public function __construct() {
    }
    //重置到到第一个元素
    public function rewind() { reset($this->_items); }
    //返回当前元素
    public function current() { return current($this->_items); }
    //返回当前元素到键 
    public function key() { return key($this->_items); }
    //返回下一个元素
    public function next() { return next($this->_items); }
    //检查当前元素是否有效
    public function valid() { return ( $this->current() !== false ); }
}
$sa = new sample();
foreach($sa as $key => $val){
    print $key . "=>" . $val;
}
//输出0=>1 1=>2 2=>3 3=>4 4=>5 5=>6 6=>7
```

我们通过foreach 遍历对象的时候，内部会依次调用

- rewind() 重置到第一个元素
- valid() 检查当前位置是否有效
- current() 返回当前元素
- key() 返回当前元素的键
- next() 指向下一个元素

## 致谢

感谢你看完这篇文章，有什么不对的地方欢迎指出，谢谢🙏

