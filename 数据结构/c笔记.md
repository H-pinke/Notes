##### c 语言终端运行

```
 vim abc.c 编写
 cc -c abc.c //编译记得安装 gcc
 cc abc.o //链接
 ./a.out. //运行
```

##### 动态分配内存

```
一般我们动态分配用 malloc
这个是返回数组的首地址
例如
int * pArr = (int *)malloc(sizeof(int) * len);
*pArr = 4 // 类似于 a[0] = 4
pArr[1] = 10;//类似于 a[1] = 10;
```

###### 注意：

Malloc 函数必须自己通过free()释放内存。要不然就的等待整个程序终止才会释放

##### typedef

```
typedef int ZHAG //为int再重新多取一个名字，ZHAG 等价于int
typedef struct Student 
{
		int sid;
		int name[100];
		char sex;
}* PST,ST;//等价于ST 代表struct Student, PST 代表了struct Student *
```

##### malloc	calloc

c语言的标准内存分配函数

- malloc	于 calloc的区别为1块与n块的区别
- malloc 调用形式为 （类型*)malloc(size):在内存的动态存储区中分配一块长度为size字节的连续区域，返回该区域的首地址
- calloc 调用形式为 (类型*)calloc(n,size): 在内存的动态存储区中分配n块长度为size字节的连续区域，返回首地址
- realloc 调用形式为(类型*)realloc(ptr, size):将ptr内存大小增加到size
- free的调用形式为free(void ptr):释放ptr所指向的一块内存空间

