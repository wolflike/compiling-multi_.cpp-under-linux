构建项目
实际开发过程中当然不可能只有一个cpp这么简单，有时候会有非常多的.h和.cpp文件相互配合，那么上面直接通过g++编译可执行文件就没那么简单了。我们需要借助Make这个强大的项目构建工具，帮助我们构建和组织项目代码。

假设现在有如下3个文件：hw2.cpp、solution.h和solution.cpp

1 /* solution.h */
2 class Solution {
3 public:
4     void Say();
5 };
1 /* solution.cpp */
2 #include <iostream>
3 #include "solution.h"
4 void Solution::Say(){
5    std::cout << "HI!" << std::endl;
6 }
复制代码
复制代码
1 /* hw2.cpp */
2 #include "solution.h"
3 int main () {
4     Solution sln;
5     sln.Say();
6     return 0;
7 }
复制代码
复制代码
 

可以看到这个简单例子包括头文件引用、定义和实现分离等情况，如果直接g++ -o hw2.out hw2.cpp将会报未定义引用的错误：

[xxx@xxx ~]$ g++ -o hw2.out hw2.cpp 
/tmp/ccIMYTxf.o：在函数‘main’中：
hw2.cpp:(.text+0x10)：对‘Solution::Say()’未定义的引用
collect2: 错误：ld 返回 1

 

这时Make就该大显身手了。

首先我们还需要了解一下makefile。

在项目的根目录下创建一个makefile文件，以告诉Make如何编译和链接程序。

复制代码
复制代码
1 build : hw2.o solution.o
2     g++ -o build hw2.o solution.o #注意前面必须是tab，不能是空格
3 hw2.o : hw2.cpp solution.h
4     g++ -g -c hw2.cpp
5 solution.o : solution.h solution.cpp
6     g++ -g -c solution.cpp
7 clean :
8     rm hw2.o solution.o build
复制代码
复制代码
 

先来解释一下makefile的基本语法规则：

target ... : prerequisites ...
　　command    #注意前面是tab
target是一个目标文件，可以是Object File，也可以是执行文件，还可以是一个标签；

prerequisites是要生成那个target所需要的文件或是目标；

command是make需要执行的命令（任意的Shell命令）。

说白了就是target这一个或多个目标，依赖于prerequisites列表中的文件，其执行规则定义在command里。如果prerequisites列表中文件比target要新，就会执行command，否则就跳过。这就是整个make过程的基本原理。

 

那么，我们回头看看上面定义的makefile文件，我们解释一下每两行的作用

1 build : hw2.o solution.o
2     g++ -o build hw2.o solution.o
target是build，依赖于hw2.o 和 solution.o，执行的命令是 g++ -o build hw2.o solution.o

意思是通过g++链接hw2.o和solution.o，生成可执行文件build，prerequisites有两个.o文件，是因为代码里hw2引用了solution.h。

 

3 hw2.o : hw2.cpp solution.h
4      g++ -g -c hw2.cpp
target是hw2.o，依赖于hw2.cpp和solution.h，执行命令是g++ -g -c hw2.cpp

意思是通过g++编译hw2.cpp文件，生成hw2.o文件，g++命令中 -g 表示生成的文件是可调试的，如果没有-g，调试时无法命中断点。

 

5 solution.o : solution.h solution.cpp
6     g++ -g -c solution.cpp
同上，编译solution.cpp文件，生成solution.o文件。

 

7 clean :
8     rm hw2.o solution.o build
这里clean不是一个可执行文件，也不是一个.o文件，它只不过是一个动作名字，类似于label的作用，make不会去找冒号后的依赖关系，也不会自动执行命令。如果要执行该命令，必须在make后显示指出整个动作的名字，如make clean。

 

好了，接下来说一下make的工作原理。在默认的方式下，我们只需输入make，则发生了以下行为：

a. make在当前目录下找名为makefile或Makefile的文件；

b. 如果找到，它会找文件中的第一个target，如上述文件中的build，并作为终极目标文件;

c. 如果第一个target的文件不存在，或其依赖的.o 文件修改时间要比target这个文件新，则会执行紧接着的command来生成这个target文件;

d. 如果第一个target所依赖的.o文件不存在，则会在makefile文件中找target为.o的依赖，如果找到则执行command，.o的依赖必是.h或.cpp，于是make可以生成 .o 文件了

e. 回溯到b步执行最终目标

 

看一下执行结果

复制代码
复制代码
[xxx@xxx ~]$ make
g++ -g -c hw2.cpp
g++ -g -c solution.cpp
g++ -o build hw2.o solution.o #注意前面必须是tab，不能是空格
[xxx@xxx ~]$ ./build 
HI!
[xxx@xxx ~]$
复制代码
复制代码
 

由于makefile文件中加了-g这一选项，于是可以通过gdb进行调试，并且会命中断点，这里感兴趣可以再了解一下gdb的使用。

接下来我们要说到如何通过VS Code进行调试。
