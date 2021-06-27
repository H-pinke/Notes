##### 字符串SDS定义

```
struct sdshdr {
	int free; //数组中未使用字节的数量
	int len; // 所保存字符串的长度
	char buf[]; // 字节数组，用于保存字符串	
}
```

