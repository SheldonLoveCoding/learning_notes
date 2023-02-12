# linux基础

linux常用指令

mkdir ./xxx/ 生成xxx文件夹

cp 

## GCC

程序由源代码到可执行程序的步骤包括：

预处理：头文件的展开、宏定义的替换

编译：新生成汇编代码



![gcc_2](.\asset\gcc_2.png)

上述过程对应的GCC命令为：

| gcc编译选项                                 | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| `-E`                                        | ==预处理指定的源文件，不进行编译==                           |
| `-S`                                        | ==编译指定的源文件，但是不进行汇编==                         |
| `-c`                                        | ==编译、汇编指定的源文件，但是不进行链接==                   |
| `-o [file1] [file2]或者 [file2] -o [file1]` | ==将文件 file2 编译成可执行文件 file1==                      |
| `-I directory`                              | ==指定 include 包含头文件的搜索目录==                        |
| `-g`                                        | 编译的时候生成调试信息，该程序可以被调试器调试               |
| `-D`                                        | ==在程序编译的时候，指定一个宏==                             |
| `-w`                                        | 不生成任何警告信息                                           |
| `-Wall`                                     | 生成所有警告信息                                             |
| `-On`                                       | n的取值范围: 0~3。编译器的优化选项的4个级别，-O0表示没有优化，-O1为缺省值，-O3优化级别最高 |
| `-l`                                        | ==在程序编译的时候，指定使用的库==                           |
| `-L`                                        | ==指定编译的时候，搜索的库的路径==                           |
| `-fPIC/-fpic`                               | 生成与位置无关的代码                                         |
| `-shared`                                   | ==生成共享目标文件，通常用在建立共享库时==                   |
| `-std`                                      | 指定C方言，如:-std=c99，gcc默认的方言是GNU C                 |

## 库文件的制作与使用

### 静态库的制作与使用

库文件是将我们写好的代码打包好一个库，可以提供给其他用户调用或使用的一些函数和类，而且不用提供源码即可使用，但是不可以单独使用。

库文件分为静态库和动态库，静态库是程序在链接阶段直接复制到程序中使用；动态库是程序在运行过程中将相关内容加载到内存中使用。

Linux下静态库命名规则是 `libxxx.a`，前缀`lib`固定，`xxx`库的名字，`.a`固定

Windows下静态库命名`linxxx.lib`

静态库的制作流程：

1. gcc生成 .o文件 （经过预处理、编译、汇编之后的文件， gcc -c）

   `gcc -c xxx.c xx.c xxx.c`

2. 将 .o 文件打包，使用 `ar`工具生成库文件

   `ar rcs libxxxx.a xx.o xx.o xx.o`

   r - 将文件插入备存文件中；c - 建立备存文件； s - 索引

使用：

```gcc
/////////
├── include
│   └── head.h
├── lib
│   └── libcalc.a
├── main.c
└── src
    ├── add.c
    └── sub.c
/////////
gcc main.c -o app -I ./include/ -l calc -L ./lib/  生成app
/////////
├── app
├── include
│   └── head.h
├── lib
│   └── libcalc.a
├── main.c
└── src
    ├── add.c
    └── sub.c
/////////
./app   即可运行
```



### 动态库的制作与使用

Linux下静态库命名规则是 `libxxx.so`，前缀`lib`固定，`xxx`库的名字，`.so`固定

Windows下静态库命名`linxxx.dll`

静态库的制作流程：

1. gcc生成 .o文件 （经过预处理、编译、汇编之后的可执行文件， gcc -c）以及与位置无关的代码

   `gcc -c -fpic xxx.c xx.c `

   -fpic 用于编译阶段，产生的代码没有绝对地址，全部用相对地址，这正好满足了共享库的要求，共享库被加载时地址不是固定的。如果不加-fpic，那么生成的代码就会与位置有关，当进程使用该`.so`文件时都需要重定位，且会产生成该文件的副本，每个副本都不同，不同点取决于该文件代码段与数 居段所映射内存的位置。

2. 将 .o 文件打包，使用 `gcc`工具生成库文件

   `gcc -shared xx.o xx.o xx.o -o libxxxx.so`



```gcc
/////////
├── include
│   └── head.h
├── lib
│   └── libcalc.so
├── main.c
└── src
    ├── add.c
    └── sub.c
/////////
gcc main.c -o app -I ./include/ -l calc -L ./lib/
/////////
├── app
├── include
│   └── head.h
├── lib
│   └── libcalc.so
├── main.c
└── src
    ├── add.c
    └── sub.c
//////////

```

使用：

由于动态库，不会把库程序代码在链接阶段直接复制到程序中，所以不能直接`./app`运行，而动态库是在运行程序的过程中将相关内容加载到内存中使用。

如何定位共享库文件呢？ 当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道绝对路径。此时就需要系统的动态载入器来获取该绝对路径。对于e1f格式的可执行程序，是由`ld-linux-so`来完成的，它先后搜索elf文件的 DT_RPATE段—＞==环境变量 ID_IIBRARY_BATH==-> /etc/ld.so.cache文件列表一> /lib/，/user/lib 目录找到库文件后将其载入内存。

使用动态库，需要告诉系统动态库的绝对路径，这里可以用的方法有：

1. 修改环境变量

   1. 临时的配置

      `ldd app`   分析动态库依赖

      `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/sheldon/test/lesson05_sharelib/library/lib` 添加到环境变量 `LD_LIBRARY_PATH`
      `./app` 运行

   2. 用户级别的配置 永久，配置`.bashrc`

      ```
      cd home/sheldon/
      vim .bashrc
      在该文件里最后一行添加 (键入o，进入插入模式)
      export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/sheldon/test/lesson05_sharelib/library/lib
      保存退出 (ecs-:wq)
      source ~/.bashrc
      ./app
      ```

   3. 系统级别配置

      ```
      sudo vim /etc/profile
      在该文件里最后一行添加 (键入o，进入插入模式)
      export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/sheldon/test/lesson05_sharelib/library/lib
      保存退出 (ecs-:wq)
      source /etc/profile
      ./app
      ```

      

2. 

3.  把动态库放入到 `/lib/，/user/lib/` 这个方法不推荐，因为这里面本身有很多库文件，有可能出现冲突。
   

### 静态库动态库的优缺点

静态库优点：

- 加载速度快
- 发布程序无需提供静态库，移植方便（因为链接时已经把程序写到了可执行文件里了）

静态库缺点：

- 消耗系统资源，浪费内存
- 更新部署发布麻烦

动态库优点：

- 可以实现不同进程之间的共享
- 更新部署发布简单
- 可以控制何时加载动态库

动态库缺点：

- 加载速度比静态库慢
- 发布程序时需要提供依赖的动态库



## Makefile

Makefile文件定义了一系列的规则来制定哪些文件需要先编译 、后编译、重新编译...... Makefile像一个shell脚本，执行操作系统命令，实现项目的自动化编译。

`make`是一个命令，解释Makefile文件中的指令的命令工具。

### Makefile命名规则

文件命名：makefile 或 Makefile

Makefile规则：

目标：最重要生成的文件

依赖：生成目标文件需要的文件或目标

命令：执行命令从依赖得到目标（主要`Tab`）

```
目标...: 依赖...
	命令（shell 命令）
```

### Makefile工作原理

首先会检查当前路径下 `该规则所需要的依赖` 是否存在，存在就执行，不存在则会往下执行其他的规则。==`Makefile`中其他规则一般都是为第一条规则服务的。==如果其他的规则对第一条规则没有贡献，那么这一条规则不会被执行。

Makefile会检测更新，比较目标和依赖文件的时间，如果依赖的时间比目标的时间玩，则需要重新生成目标。如果以来的时间比目标的时间早，则目标不需要更新，对应的规则的命令不需要被执行。

```makefile
app:add.o sub.o main.o
	gcc add.o sub.o main.o -o app

add.o:add.c
	gcc -c add.c

sub.o:sub.c
	gcc -c sub.c
	
main.o:main.c
	gcc -c main.c
```



### Makefile变量

变量只能在命令中使用。

- 自定义变量

  变量名 = 变量值 `var=hello`

- 预定义变量

  AR：归档维护程序的名称，默认为 ar

  CC：C编译器的名称，默认值 gcc

  CXX：C++编译器名称，默认为 g++

  $@：目标的完整名称

  $<：第一个依赖文件的名称

  $^：所有的依赖文件

- 获取变量的值

  $(变量名)

```makefile
#定义变量
src = add.o sub.o main.o
target = app
$(target): $(src)
	$(CC) $(src) -o $(target)
%.o:%.c
	$(CC) -c $< -o $@
```

运行结果就是

```makefile
cc -c add.c -o add.o
cc -c sub.c -o sub.o
cc -c main.c -o main.o
cc add.o sub.o main.o -o app
```

### Makefile函数

`$(wildcard PATTERN...)`

- 功能：获取指定目录下的某个类型的文件列表，PATTERN就是指某个或多个目录下对应的某个类型的文件，如果有多个目录，目录间空格隔开。
- 返回值是得到的若干个文件名称，文件名之间空格间隔。

```makefile
$(wildcard ./*.c) #注意没有逗号
#返回值 add.c sub.c main.c 
```

`$(patsubst <pattern>,<replacement>,<text>)`

- 查找`<text>`中的单词是否符合`<pattern>`，符合的话则利用`<replacement>`替换。`<pattern>`可以包括通配符`%`
- 返回值是替换后的字符串

```makefile
$(patsubst %.c %.o, add.c sub.c main.c) #注意没有逗号
#返回值 add.o sub.o main.o
```

```makefile
#定义变量
src=$(wildcard ./*.c)
obj=$(patsubst %.c, %.o, $(src))
target = app
$(target): $(obj)
	$(CC) $(obj) -o $(target)
%.o:%.c
	$(CC) -c $< -o $@

.PHONY:clean
clean:
	rm -f $(obj)
```

