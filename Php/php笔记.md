##### global关键字

我们经常会遇到一种情况，在函数外部声明了一个全局变量后，我们想在函数内部访问或者修改这一全局变量。

```
$var1 = 4;
function gloable_references() {
    global $var1;//等价于 $var1 =& $GLOBALS['var1']
    $var1 = 6;//等价于$GLOBALS["var1"] =6;
    echo $var1;//6
    return $var1;
}
gloable_references();
echo $var1; //6
```

