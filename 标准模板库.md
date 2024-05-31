# C++标准模板库

## 一. 标准容器

### 1. 顺序容器

#### vector

vector的底层是一个**动态开辟**的数组，在g++编译器中是以2倍的方式进行扩容，在msvc编译器中是以1.5倍的方式进行扩容。

**添加元素的方法：**

1. push_back(elem);
2. emplace_back();
3. insert(iterator, pos);

添加元素可能会导致容器的扩容。

**删除元素的方法：**

1. pop_back();
2. erase(iterator);

**查询：**

1. operator[];
2. iterator;
3. at();
4. find(), for_each();

```c++
#include <iostream>

using namespace std;

int main() {
    vector<int> vec {1, 2, 3, 4, 5, 6, 7, 8, 9};
    for (int i = 0; i < 10; ++i) {
        // 通过push_back()在vector尾部添加元素
        vec.push_back(i);
    }
    
    // 将vector中的所有偶数删除
    for (auto it = vec.begin(); it != vec.end(); ) {
		if (*it % 2 == 0) {
            it = vec.erase(it);
        } else { 
            ++it;
        }   
    }
    // 给vector中所有的奇数前都添加一个比该奇数小一的偶数
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        if (*it % 2 == 1) {
            it = vec.insert(it, *it - 1);
            ++it;
        }
    }
    return 0;
}
```

**对容器进行连续的插入或删除操作，一定要更新迭代器，否则第一次的插入或删除完成后，迭代器就失效了**

#### deque

deque的底层数据结构为一个动态开辟的二维数组，一维数组从2开始，以二倍的方式进行扩容。每次扩容以后，原来的二维数组，从新的一维数组下标 oldsize / 2位置开始存放，上下都预留相同的空行，方便支持deque的收尾元素添加。

```c++
#include <deque>

// deque的定义类似如下结构
template <typename T>
struct Deque {
    T* first;
    T* cur;
    T* last;
    T** m_data;
};
// 其中m_data是一个动态扩容的数组，数组中保存的元素是一个指向固定大小的T类型的数组。
// 所以deque中每一个固定大小的数组都是之间都是不连续的。
```



**添加元素**：

push_back();

push_front();

inserte(iterator, pos);

**删除元素：**

pop_back();

pop_front();

erase(iterator);

#### list

list的底层数据结构是一个双向的循环链表。

**添加元素**：

push_back();

**push_front();** // 相较于vector增加的方法

inserte(iterator, pos);

**删除元素：**

pop_back();

**pop_front();** // 相较于vector增加的方法

erase(iterator);

#### 各顺序容器之间的区别

**vector和deque之间的区别：**

1. 底层的数据结构不同，vector是动态开辟的一维数组，deque是动态开辟的二维数组，其中第一个维度和第二个维度都是动态开辟。
2. 前中后插入元素的时间复杂度区别：在**末尾插入元素**的时间复杂度为0(1)，deque在**最前面插入元素**的时间复杂度为O(1),但是vector在最前面插入元素的复杂度为0(n),在**中间插入元素**的时间复杂度为O(n)。
3. 对于内存的使用效率，vector需要分配一大段连续的内存，而deque需要分配若干连续的内存，因而deque对内存的利用效率更高。
4. 如果非要说vector和deque在中间插入元素谁的效率更高一点，vector的效率更高点，deque因为其内存不连续因而效率低一些。

**vector和list之间的区别：**

1. 底层的数据结构不同，vector是动态开辟的一维数组，list是双向循环链表。
2. vector支持随机访问，而list不支持随机访问。
3. vector在最前端插入元素复杂度为O(n),而list在最前端插入元素的复杂度为O(1)
4. vector插入或删除元素会导致后面元素的迭代器失效，而list不会。

### 2. 容器适配器

**容器适配器具有以下特性：**

1. 容器适配器底层**没有自己的数据结构**，它是另外一个容器的**封装**，它的全部方法都是由底层依赖的容器进行实现的。
2. 没有实现自己的迭代器。

#### stack

```c++
#include <stack>
#include <iostream>

using namespace std;

int main() {
    stack<int> s;
    for (int i = 0; i < 10; ++i) {
        s.push(i); // 入栈
    }
    // 栈大小
    cout << s.size() << endl;
    // 栈判空
    while (!s.empty()) {
        // 获取栈顶元素
        cout << s.top() << " ";
        // 出栈
        s.pop();
    }
    return 0;
}
```

#### queue

```c++
#include <iostream>
#include <queue>

using namespace std;

int main() {
    queue<int> q;
    for (int i = 0; i < 10; ++i) {
        q.push(i);
    }
    cout << q.size() << endl;
    while (!q.empty()) {
        cout << q.front() << " ";
        q.pop();
    }
    return 0;
}
```

#### priority_queue

```c++
#include <iostream>
#include <queue>

using namespace std;

int main() {
    // 创建一个大根堆
    priority_queue<int> pq;
    // 如果要创建一个小根堆，需要采用如下方法,其中模板第二个参数为底层容器，第三个参数表示数据的比较方法
    priority_queue<int, vector<int>, greater<int>> pq2;
    for (int i = 0; i < 10; ++i) {
        pq.push(i);
    }
    cout << pq.size() << endl;
    while (!pq.empty()) {
        cout << pq.top() << " ";
        pq.pop();
    }
}
```



重要问题解析：

**(1). 为什么stack和queue的底层依赖于deque而不是vector？**

1. 因为vector的初始内存效率太低了，以元素为int类型数据为例，初始状态下vector是按照1->2->4->8->16->32的扩容方式来进行内存的开辟，而deque初始的第一个第二维度动态数组的大小就是4096 / sizeof(int)，也即初始状态就开辟了1024个元素空间大小。

2. 对于queue和stack来说，需要支持在尾部进行插入或删除数据的操作，vector在尾部插入的时间复杂度为O(1),而在头部进行插入或删除的时间复杂度为O(n)，相比来说deque在尾部插入的时间复杂度为O(1),在头部删除的时间复杂度也为O(1),因而deque的效率更高。

3. vector需要大片连续的内存，而deque需要分段的连续内存，因而deque对内存的利用效率更高。

**(2). 为什么priority_queue的底层依赖于vector而不是deque？**

因为priority_queue底层默认把数据组成一个大根堆或小根堆结构，而堆结构通常保存在一个内存连续的数组中，通过数组的下标来访问大根堆中的每一个节点。如果使用deque由于其内存不连续，使用数组下表访问每一个元素并不如vector来的便利，因而采用vector作为priority_queue的底层容器。

### 3. 关联容器

#### 无序关联容器

无序关联容器的底层实现方式一般为**链式哈希表**，其增、删、查的复杂度接近O(1)。无序关联容器包括unordered_set，unordered_multiset，unordered_map

，unordered_multimap

#### 有序关联容器

有序关联容器的底层实现方式一般为**红黑树**，其增、删、查的复杂度为(logn)。有序关联容器包括set，multiset，map，multimap

```c++
#include <iostream>
#include <set>
#include <unordered_set>
#include <map>
#include <unordered_map>

using namespace std;

int main() {
    // set
    set<int> st;
    for (int i = 0; i < 50; ++i) {
        st.insert(rand() % 20 + 1);
    }
    cout << "set size: " << st.size() << endl;
    cout << st.count(15) << endl;c 
    // multiset
    set<int> st2;
    for (int i = 0; i < 50; ++i) {
        st2.insert(rand() % 20 + 1);
    }
    cout << "multiset size: " << st2.size() << endl;
    cout << st2.count(15) << endl;
    return 0;
}
```

## 二. 迭代器

## 三. 函数对象

```c++
#include <iostream>

using namespace std;

// 通过函数指针解决比较方法不灵活的问题
template <typename T>
bool mygreater(const T& a, const T& b) {
    return a > b;
}

template <typename T>
bool myless(const T& a, const T& b) {
    return a < b;
}

// c++函数对象的实现方式
template <typename T>
class MyGreater {
public:
    inline bool operator()(const T& a, const T& b) {
        return a > b;
    }
};

template <typename T>
class MyLess {
public:
    inline bool operator()(const T& a, const T& b) {
        return a < b;
    }
};

template <typename T, typename Compare>
bool compare(const T& a, const T& b, Compare comp) {
    return comp(a, b);
}

int main() {
    // 通过指针调用函数是不能实现内联的，效率很低，因为有函数调用的开销
    cout << compare(10, 20, mygreater<int>) << endl;

    // 通过函数对象调用operator()，可以省略函数开销，比通过函数指针效用
    // 函数（不能够内联）效率高
    cout << compare(10, 20, MyLess<int>()) << endl;
    return 0;
}
```

## 四. 泛型算法

泛型算法的特点：

1. 泛型算法的参数接受的都是迭代器
2. 泛型算法的参数还可以接收函数对象(c函数指针)

```c++
#include <functional>
#include <iostream>
#include <vector>
#include <algorithm>
#include <string_view>

using namespace std;

int main() {
	int arr[] = {1,3,34,5,6,4,4,3,4,5,23,4,54,4,234,4,543,4,345,45,4545,65,67};
    vector<int> vec(begin(arr), end(arr));
    for (auto c : vec) {
        cout << c << " ";
    }
    cout << endl;
    sort(vec.begin(), vec.end());
    for (auto c : vec) {
        cout << c << " ";
    }
    // binary search
    cout << endl;
    if (binary_search(vec.begin(), vec.end(), 65)) {
        cout << 65 << "存在\n";
    } else {
        cout << 65 << "不存在\n";
    }
    // find 
    auto it2 = find(vec.begin(), vec.end(), 65);
    if (it2 != vec.end()) {
        cout << "找到了\n";
    } else {
        cout << "找不到\n";
    }
    // find if 
    auto it3 = find_if(vec.begin(), vec.end(), [](int val) {
        return val > 20;
    });
    vec.insert(it3, 20);
    // 使用绑定器
    auto it4 = find_if(vec.begin(), vec.end(), bind(greater<int>(), placeholders::_1, 48));
    // auto it4 = find_if(vec.begin(), vec.end(), bind1st(greater<int>(), 48));
    vec.insert(it4, 48);
    for (auto c : vec) {
        cout << c << " ";
    }
    cout << endl;
    for_each(vec.begin(), vec.end(), [](int elem){
        if (!(elem & 1)) {
            cout << elem << " ";
        }
    });
    cout << endl;
}
```

