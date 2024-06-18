# const用法

## C和C++中const的区别

```c++
void main() {
    const int a; // 在c中合法，在c++中不合法。
    
    const int b = 20;	// 在c++中以立即数初始化的b表示一个常量，在编译阶段会把所有b出现的地方都替换为20
    int c = 40;
    const int d = c; // 如果用一个变量初始化const int，那么d就退化为和c中一样的常变量。 此时的d不能作为数组的大小，
    int arr[d]; // 错误，此时的d为常变量。
    return 0;
}
```

c中const主要修饰为**常变量**，主要侧重在变量级别，因此在c中将const修饰的int值作为数组的大小不合法，但是在c++中合法。

c++中const修饰的常量经常出现的错误：

1. 常量不能再作为左值（直接修改常量的值）
2. 不能把常量的地址泄露给一个普通的指针或引用（间接修改常量的值）

## const和一级指针的结合

```c++
// const和一级指针的结合方式
const int* p; // 离const最近的类型是int，因此const修饰的是int，这种类型指针可以指向别处，但是不能修改指针指向的内容
int const* p; // 同上
int* const p; // 离const最近的类型是int*，因此const修饰的是int*，也即指针是常量，不能指向别处，但是能通过指针修改指向的内容
const int* const p; // 既不能指向别处，也不能修改指向的内容 
```

const如果右边没有头*的时候，const是不参与类型的。

```c++
// 如果用typeid()函数来查看类型的话
int a = 10;
const int b = 20;
int* const pa = &a;
const int* pb = &b;

typeid(pa).name() // --> int*
typeid(pb).name() // --> const int*
```

## const和二级(多级)指针的结合

总结下来，转换情况如下：

```c++
/*
	int* <---- const int* 错误
	const int* <---- int* 正确
	
	const int** <---- int ** 错误
	int** <---- const int**  错误
	
	int* const* <---- int** 正确
	int** <---- int* const* 错误
*/
```

