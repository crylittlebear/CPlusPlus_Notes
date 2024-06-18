# GDB调试常见命令

## 1.GDB调试基本命令

**list命令：**

list命令用于查看源码，且只能显示10行，如果想要继续查看，输入l显示下一个十行代码。

**b命令:**

使用b main表示在main函数的第一行设置一个断点， b main.cpp:8表示在main.cpp源码的第8行设置一个断点。

**r命令或run命令：**

r命令用于表示运行程序。当运行程序后，程序会在设置的第一个断点处停下来。

**p指令**

p指令表示print表示答应某个变量的值，如p a表示打印a的值。

**n指令或next**

n表示执行当前行并转入到下一行。如果当前语句中有函数调用也不会进入函数中，直接执行这一句并转到下一句。

**s指令或step**

s指令表示跳转到第一行，如果当前中有函数调用，则会转到被调用函数的第一句。

**quit, detach, q指令**

这三个指令都是退出gdb的命令。 q和quit是直接结束gdb程序，detach是将程序与gdb分离，但是不会结束gdb调试。

**finish指令**

finish指令会结束当前函数的执行，并返回到调用该函数的语句的下一行继续执行。

**info locals(i locals)指令**

表示打印出所有的局部变量的值。

**c指令：**

表示继续执行，而不是移动到下一行，c指令会继续执行代码直到碰到下一个断点才会停下来。

## 2. 启动调试并传入参数

**gdb --args <exe> <args>**

在启动程序的时候使用gdb --args main 1 2 "3 4"，这样就给main程序传入了1， 2， "3 4"三个参数。

**set args <args>**

在执行了gdb指令以后，在命令中输入set args <args> 也可以对参数进行设置。

**r <args>**

在执行了gdb指令以后，在命令中输入r <args> 也可以对参数进行设置。

## 3. 对已执行的进程进行调试

**gdb attach <pid>**

可以使用ps -aux | grep <ext> 命令来查看要查询的应用程序的进程号pid

**gdb --pid <pid>**

## 4. 断点

**b <func> 指令**

使用b func这个指令会在每个func函数处设置断点，也就是说如果程序有若干个全局的重载函数，只要函数名相同，不管其参数和返回值是否相同，都会为每个函数设置一个断点。

**rb <some>**

使用rb指令可以为所有带有some的函数都设置断点。

**条件断点： b main.cpp:6 if <表达式>**

```c++
#include <iostream>

int main() {
    int sum = 0;
    for (int i = 0; i < 100; ++i) {
        sum += i;
	}
    return 0;
}
```

上面这个指令的意思是在main.cpp中的第6行设置一个断点，并且当满足一个条件，比如说慢许i==90的时候，停下来。我们就可以使用b main.cpp:6 if i == 90

**临时断点： tb main.cpp:6**

临时断点的意思是设置的断点只会被命中一次，然后就不会再命中了。

**查看断点： info breakpoints(i b)**

查看所有的断点

**删除断点: delete <断点号 num>**

如果在delete后面不跟要删除的断点号，那么会删除所有的断点。

**禁用断点和开启断点（disable <num> 或 enable <num>）**

## 5. 查看/修改变量

**info args（i args）**

显示函数中的所有参数

**set print null-stop**

这个指令的意思是让字符串碰到'\0'就停下来。

```c++
#include <iostream>

int main() {
    char name[100] = {0};
    strcpy(name, "simplesoft");
    return 0;
}
```

这样的数，如果我们在调试的时候用p name这个指令来查看name的内容的时候，会显示"simplesoft", "\0000" <repeats 89 times>,而我们一般只想看到"simplesoft",这时我们就需要输入上面的代码来精简字符串的输出。

**set print pretty**

这个指令会让我们查看某些变量的时候更好看，比如说某个结构体有好几个成员，这个时候如果直接使用p指令查看的话，现实的格式不是那么直观，因此我们需要再p指令前输入上面的指令让结构体以叫好看的方式显示。

**set print array on**

使数组的显示方式更直观。我们还可以使用p sizeof(int)来查看int的大小。

## 6. 查看/修改内存

**x /10d <address>**

上面这个命令的意思是查看address内存开始的10个字节，并以整数的方式输出每个字节。假设我们有一个变量a，我们可以通过x &a查看a变量的地址和地址内保存的内容。

**x /10d <adress>**

这个指令的意思是可以查看地址address开始的10个字节，并以字符char的方式输出每一个字节。

## 7. 查看/修改寄存器

查看或修改寄存器尤其在调试发布出去的代码尤其有用，因为发布出去的代码没有调试符号，因此需要查看寄存器的内容。

### 查看寄存器

**info registers <register symbol>可以简写为i r <ragister symbol>**

如果函数的参数小于等于6个的话，会把参数放到寄存器当中，其中

| 寄存器 |  函数参数  |
| :----: | :--------: |
|  rdi   | 第一个参数 |
|  rsi   | 第二个参数 |
|  rdx   | 第三个参数 |
|  rcx   | 第四个参数 |
|   r8   | 第五个参数 |
|   r9   | 第六个参数 |

如果函数参数多于6个，就会放在函数栈中

```assembly
Breakpoint 1, testFunc (name=0x555555556013 "simplesoft", age=20, gender=109) at main.cpp:40
warning: Source file is more recent than executable.
40          memset(&test, 0, sizeof(TestStruct));
(gdb) i args
name = 0x555555556013 "simplesoft"
age = 20
gender = 109
(gdb) p test
$1 = {name = '\000' <repeats 11 times>, gender = 0 '\000', age = 0}
(gdb) i r rdi
rdi            0x555555556013      93824992239635
(gdb) i r rdx
rdx            0x6d                109
(gdb) i r rsi
rsi            0x14                20
(gdb) list
35          return 0;
36      }
37
38      int testFunc(const char* name, int age, int gender) {
39          TestStruct test;
40          memset(&test, 0, sizeof(TestStruct));
41          strcpy(test.name, name);
42          test.age = age;
43          test.gender = gender;
44          return 0;
(gdb) x /s Quit
(gdb) x /s 0x555555556013
0x555555556013: "simplesoft"
(gdb) p (char*)0x555555556013
$2 = 0x555555556013 "simplesoft"
(gdb) c
Continuing.
[Inferior 1 (process 231046) exited normally]
(gdb)
```

### 修改寄存器

**pc/rip**寄存器中保存程序要执行的下一条指令，通过修改pc寄存器来修改程序执行的流程

具体的指令为：
**set var ${pc} = xxx 或 p ${rip} = xxx**

```assembly
Breakpoint 1, testFunc (name=0x555555556013 "simplesoft", age=20, gender=109) at main.cpp:40
warning: Source file is more recent than executable.
40          memset(&test, 0, sizeof(TestStruct));
(gdb) n
41          strcpy(test.name, name);
(gdb) n
42          test.age = age;
(gdb) i args
name = 0x555555556013 "simplesoft"
age = 20
gender = 109
(gdb) p test
$1 = {name = "simplesoft\000", gender = 0 '\000', age = 0}
(gdb) i r rip
rip            0x55555555528f      0x55555555528f <testFunc(char const*, int, int)+47>
(gdb) list
37
38      int testFunc(const char* name, int age, int gender) {
39          TestStruct test;
40          memset(&test, 0, sizeof(TestStruct));
41          strcpy(test.name, name);
42          test.age = age;
43          test.gender = gender;
44          return 0;
45      }
46
(gdb) info line 43
Line 43 of "main.cpp"
   starts at address 0x555555555295 <_Z8testFuncPKcii+53>
   and ends at 0x555555555298 <_Z8testFuncPKcii+56>.
(gdb) p $rip=0x555555555295
$2 = (void (*)(void)) 0x555555555295 <testFunc(char const*, int, int)+53>
(gdb) n
44          return 0;
(gdb) p test
$3 = {name = "simplesoft\000", gender = 109 'm', age = 0}
(gdb) disassemble
Dump of assembler code for function _Z8testFuncPKcii:
   0x0000555555555260 <+0>:     push   %rbp
   0x0000555555555261 <+1>:     mov    %rsp,%rbp
   0x0000555555555264 <+4>:     sub    $0x30,%rsp
   0x0000555555555268 <+8>:     mov    %rdi,-0x8(%rbp)
   0x000055555555526c <+12>:    mov    %esi,-0xc(%rbp)
   0x000055555555526f <+15>:    mov    %edx,-0x10(%rbp)
   0x0000555555555272 <+18>:    lea    -0x28(%rbp),%rdi
   0x0000555555555276 <+22>:    xor    %esi,%esi
   0x0000555555555278 <+24>:    mov    $0x14,%edx
   0x000055555555527d <+29>:    call   0x555555555030 <memset@plt>
   0x0000555555555282 <+34>:    lea    -0x28(%rbp),%rdi
   0x0000555555555286 <+38>:    mov    -0x8(%rbp),%rsi
   0x000055555555528a <+42>:    call   0x555555555050 <strcpy@plt>
   0x000055555555528f <+47>:    mov    -0xc(%rbp),%eax
   0x0000555555555292 <+50>:    mov    %eax,-0x18(%rbp)
   0x0000555555555295 <+53>:    mov    -0x10(%rbp),%eax
   0x0000555555555298 <+56>:    mov    %al,-0x1c(%rbp)
=> 0x000055555555529b <+59>:    xor    %eax,%eax
   0x000055555555529d <+61>:    add    $0x30,%rsp
   0x00005555555552a1 <+65>:    pop    %rbp
   0x00005555555552a2 <+66>:    ret
End of assembler dump.
(gdb) 
```

要记住**info line <line no>**这个指令，这个指令是查看某一行代码反汇编后的地址，由于有些行对应的汇编代码不止一行，因此会输出汇编指令的开始和结束的地址，我们就可以通过设置**rip**指向这个开始来跳转到某一行来执行。

具体的设置rip的方式为: **p $rip = <address>** 或 **set var $rip = <address>**

## 8. 源代码查看与管理

具体用到的命令有：

|       命令名称       |          命令           |
| :------------------: | :---------------------: |
|      显示源代码      | **list/l,默认显示10行** |
|  设置每次显示的行数  | **set listsize <size>** |
|  查看指定函数的代码  |  **list <func name>**   |
| 查看某文件指定行代码 |  **list main.cpp:15**   |

在使用list查看代码的时候，继续输入list或l则会显示接下来10行的代码，如果我们想看上面10行的代码，可以使用**list -**命令。如果存在类的继承，也即基类和派生类中都存在某个函数，可以使用作用域来查看某个特定的函数，比如list base::func可以查看base类中func函数的代码。

## 9. 搜索源代码

|      命令名称      |         命令          |
| :----------------: | :-------------------: |
|        搜索        | **search 正则表达式** |
|    从前往后搜索    |  **forword-search**   |
|    从后往前搜索    |  **reverse search**   |
| 设置源代码搜索目录 | **list main.cpp:15**  |

## 10. 函数调用栈管理

|    命令名称    |       命令       |
| :------------: | :--------------: |
| 查看栈回溯信息 | **backtrace/bt** |
|    切换栈帧    |   **frame n**    |
|  查看栈帧信息  |   **info f n**   |

## 11. 观察点

|       命令名称        |           命令            |
| :-------------------: | :-----------------------: |
|       写观察点        |         **watch**         |
|       读观察点        |        **rwatch**         |
|      读写观察点       |        **awatch**         |
|      查看观察点       |      **info watch**       |
| 删除/禁用/启用 观察点 | **delete/disable/enable** |

```c++
#include <chrono>
#include <thread>
#include <iostream>

int gdata = 0;
int gdata2 = 0;

void threadFunc(void* arg) {
    int* temp = (int*)arg;
    std::this_thread::sleep_for(std::chrono::seconds(*temp));
    gdata = *temp;
    gdata2 = 2 * (*temp);
    std::cout << "thread data: " << gdata << std::endl;
    std::cout << "test thread exited" << std::endl;
}

int main(int argc, char* argv[]) {
    int num1 = 3;
    std::thread t1(threadFunc, (void*)&num1);
    int num2 = 5;
    std::thread t2(threadFunc, (void*)&num2);
    t1.join();
    t2.join();
    std::cout << "main thread exit\n";
}
```



```assembly
Reading symbols from watch...
(gdb) l
4
5       int gdata = 0;
6       int gdata2 = 0;
7
8       void threadFunc(void* arg) {
9           int* temp = (int*)arg;
10          std::this_thread::sleep_for(std::chrono::seconds(*temp));
11          gdata = *temp;
12          gdata2 = 2 * (*temp);
13          std::cout << "thread data: " << gdata << std::endl;
(gdb) watch gdata
Hardware watchpoint 1: gdata
(gdb) r
Starting program: /home/heng/Code/linuxGdb/watch
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[New Thread 0x7ffff77ff640 (LWP 257666)]
[New Thread 0x7ffff6ffe640 (LWP 257667)]
[Switching to Thread 0x7ffff77ff640 (LWP 257666)]

Thread 2 "watch" hit Hardware watchpoint 1: gdata

Old value = 0
New value = 3
threadFunc (arg=0x7fffffffe22c) at watch.cpp:12
12          gdata2 = 2 * (*temp);
(gdb) c
Continuing.
thread data: 3
test thread exited
[Thread 0x7ffff77ff640 (LWP 257666) exited]
[Switching to Thread 0x7ffff6ffe640 (LWP 257667)]

Thread 3 "watch" hit Hardware watchpoint 1: gdata

Old value = 3
New value = 5
threadFunc (arg=0x7fffffffe214) at watch.cpp:12
12          gdata2 = 2 * (*temp);
(gdb)
```

使用awatch的方法如下：

```assembly
Reading symbols from watch...
(gdb) awatch gdata # 观察gdata的读写事件
Hardware access (read/write) watchpoint 1: gdata
(gdb) r
Starting program: /home/heng/Code/linuxGdb/watch
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[New Thread 0x7ffff77ff640 (LWP 258963)]
[New Thread 0x7ffff6ffe640 (LWP 258965)]
[Switching to Thread 0x7ffff77ff640 (LWP 258963)]

Thread 2 "watch" hit Hardware access (read/write) watchpoint 1: gdata

Old value = 0
New value = 3
threadFunc (arg=0x7fffffffe22c) at watch.cpp:12
12          gdata2 = 2 * (*temp);
(gdb) c
Continuing.

Thread 2 "watch" hit Hardware access (read/write) watchpoint 1: gdata

Old value = 3
New value = 5
0x00005555555552c1 in threadFunc (arg=0x7fffffffe22c) at watch.cpp:13
13          std::cout << "thread data: " << gdata << std::endl;
(gdb) c
Continuing.
thread data: 3
test thread exited
[Thread 0x7ffff77ff640 (LWP 258963) exited]
[Switching to Thread 0x7ffff6ffe640 (LWP 258965)]

Thread 3 "watch" hit Hardware access (read/write) watchpoint 1: gdata

Value = 5
threadFunc (arg=0x7fffffffe214) at watch.cpp:12
12          gdata2 = 2 * (*temp);
(gdb) c
Continuing.

Thread 3 "watch" hit Hardware access (read/write) watchpoint 1: gdata

Value = 5
0x00005555555552c1 in threadFunc (arg=0x7fffffffe214) at watch.cpp:13
13          std::cout << "thread data: " << gdata << std::endl;
(gdb) c
Continuing.
thread data: 5
test thread exited
main thread exit
[Thread 0x7ffff6ffe640 (LWP 258965) exited]
[Inferior 1 (process 258959) exited normally]
(gdb)
```

也可以通过指令:**watch gdata thread 2**指定对某个线程设置观察点。

```assembly
(gdb) r
Starting program: /home/heng/Code/linuxGdb/watch
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[New Thread 0x7ffff77ff640 (LWP 261458)]
[New Thread 0x7ffff6ffe640 (LWP 261459)]

Thread 1 "watch" hit Breakpoint 1, main (argc=1, argv=0x7fffffffe358) at watch.cpp:22
22          t1.join();
(gdb) info threads
  Id   Target Id                                  Frame
* 1    Thread 0x7ffff7ea0740 (LWP 261455) "watch" main (argc=1,
    argv=0x7fffffffe358) at watch.cpp:22
  2    Thread 0x7ffff77ff640 (LWP 261458) "watch" 0x00007ffff78e57f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0,
    req=0x7ffff77fecf8, rem=0x7ffff77fecf8)
    at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
  3    Thread 0x7ffff6ffe640 (LWP 261459) "watch" 0x00007ffff78e57f8 in __GI___clock_nanosleep (clock_id=clock_id@entry=0, flags=flags@entry=0,
    req=0x7ffff6ffdcf8, rem=0x7ffff6ffdcf8)
    at ../sysdeps/unix/sysv/linux/clock_nanosleep.c:78
(gdb) watch gdata thread 2
Hardware watchpoint 2: gdata
(gdb) c
Continuing.
[Switching to Thread 0x7ffff77ff640 (LWP 261458)]

Thread 2 "watch" hit Hardware watchpoint 2: gdata

Old value = 0
New value = 5
threadFunc (arg=0x7fffffffe22c) at watch.cpp:12
12          gdata2 = 2 * (*temp);
(gdb) c
Continuing.
thread data: 5
test thread exited
[Thread 0x7ffff77ff640 (LWP 261458) exited]
thread data: 5
test thread exited
Thread-specific breakpoint 2 deleted - thread 2 no longer in the thread list.
main thread exit
[Thread 0x7ffff6ffe640 (LWP 261459) exited]
[Inferior 1 (process 261455) exited normally]
(gdb)
```

也可以使用指令**watch gdata + gdata2 > 10**这个指令，当满足这个条件的时候，就会命中该观察点。

