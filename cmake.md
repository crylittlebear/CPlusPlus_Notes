# cmake学习笔记

## 1. 基本用法

### 最小版本要求

```cmake
#最低版本信息
cmake_minimum_required(VERSION 3.25)
```

### 构建项目的名称(project)

```cmake
#构建的项目的名称
project(project_name)
```

### 添加可执行程序

```cmake
#生成可执行程序,名字之间可以通过空格或分号间隔
add_executable(可执行程序 源文件名称)
#设置可执行程序输出路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
#设置c++标准
set(CMAKE_CXX_STANDARD 11)
```

### 定义变量

```cmake
#通过set来定义变量
set(SRC_LIST add.cpp mul.cpp main.cpp)
add_executable(app ${SRC_LIST})
```

### 常用的目录路径宏

```cmake
#CMAKE_CURRENT_SOURCE_DIR是指当前处理的CMakeLists.txt文件所在的目录。
CMAKE_CURRENT_SOURCE_DIR

#PROJECT_SOURCE_DIR是指项目的顶层目录（即包含第一个调用project()命令的CMakeLists.txt文件的目录）
PROJECT_SOURCE_DIR
```

### 搜索源文件

```cmake
aux_source_directory(<dir> <variable>)
#示例
# PROJECT_SOURCE_DIR表示的是CMakeLists.txt所在的路径，将该路径下的所有原文件保存到SRC中
#第一个参数是文件的路径
#第二个参数是变量名
aux_source_directory(${PROJECT_SOURCE_DIR} SRC)

#file命令 
# 第一个参数可以为CLOB/CLOB_RECURSE，其中第一个为当前目录下搜索、第二个命令是当前目录下递归搜索
# 第二个参数是变量名
# 第三个参数是目录的路径,路径后面需要加上要搜寻文件的类型
file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp) 
```

### 指定头文件路径

```cmake
#指定头文件的路径
include_directorys(<header path>)
```

### 将源文件制作成静态库或动态库

```cmake
#制作库 
#库名称不需要加lib和so后缀，最后生成的库会自动加上
add_library(<库名称> STATIC/SHARED <源文件>)

#指定动态库或动态库的生成路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

#使用生成的库文件

#链接静态库
#指定静态库名字的时候可以掐头去尾，也即省略掉lib和.a
link_libraries(<static lib> <static lib> ...)
#指定静态库或动态库的路径
link_directorys(<lib> <path>)

#链接动态库
#target可以是一个源文件，可以是一个动态库文件，也可以是一个可执行文件
#在cmake中指定要链接的动态库的时候，应该将（target_link_libraries）这个命令放在可执行程序生成之后
target_link_libraries(<target> <PRIVAGE/PUBLIC> <item> <PRIVAGE/PUBLIC> <item> ... )
```

### 日志

```cmake
#在cmake中可以用用户显示一条消息，命令名为message
#警告级别分为
# 无： 重要消息
# STATUS: 非重要消息
# WARNING: cmake警告，会继续执行
# AUTHOR_WARNING: cmake警告(dev),会继续执行
# SEND_ERROR: cmake错误，继续执行，但会跳过生成的步骤
# FATAL_ERROR: cmake错误，终止所有处理过程
message(<警告级别> "message to display" ... )
#示例

#输出一般日志信息
message(STATUS "source path: ${PROJECT_SOURCE_DIR}")
#输出警告信息
message(WARNING "source path: ${PROJECT_SOURCE_DIR}")
#输出错误信息
message(FATAL_ERROR "source path: ${PROJECT_SOURCE_DIR}")
```

### 变量操作

```cmake
#用set进行字符串的拼接
set(<变量名> ${变量名1} ${变量名2} ...)

#使用list进行字符串的拼接
#第二个参数就是变量的名字
list(APPEND <list> [<element> ...])

#字符串移除 其中value不能单单是源文件的名称，需要加上绝对路径
list(REMOVE_ITEM <list> <value> [<value>...])
```

### 在CMAKE中进行宏定义

```cmake
#在cmake中对执行的宏进行定义的命令为add_definitions
add_definitions(-D宏名称)
```

### 多层CMakeLists嵌套

```cmake
#构建子节点
add_subdirectory(source_dir)
```

