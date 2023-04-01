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

如何定位共享库文件呢？ 当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道绝对路径。此时就需要系统的动态载入器来获取该绝对路径。对于elf格式的可执行程序，是由`ld-linux-so`来完成的，它先后搜索elf文件的 DT_RPATE段—＞==环境变量 ID_IIBRARY_BATH==-> /etc/ld.so.cache文件列表一> /lib/，/user/lib 目录找到库文件后将其载入内存。

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

## GDB调试

GDB是调试程序的工具，与GCC搭配形成一整套开发环境。

GDB 主要帮助你完成下面四个方面的功能： 

1. 启动程序，可以按照自定义的要求随心所欲的运行程序 
2. 可让被调试的程序在所指定的调置的断点处停住（断点可以是条件表达式） 
3. 当程序被停住时，可以检查此时程序中所发生的事 
4. 可以改变程序，将一个BUG 产生的影响修正从而测试其他 BUG

在为调试而编译程序时，需要：关掉`-O`；打开调试选项`-g`；打开所有Warning`-Wall`；

```gcc
gcc -g -Wall main.c -o app
```

==app 和 main.c 需要在同一个文件夹下。==

### GDB命令

| commend                   | function               |
| ------------------------- | ---------------------- |
| gdb                       | 打开gdb                |
| quit                      | 退出                   |
| set args 10 20            | 给程序设置参数         |
| show args                 | 显示程序已设置的参数   |
| help                      | 帮助                   |
| list or l                 | 查看当前文件代码       |
| list or l 行号            | 从当前文件指定行显示   |
| list or l 函数名          | 从当前文件指定函数显示 |
| list or l 文件名:行号     | 从指定文件指定行显示   |
| list or l 文件名:函数名   | 从指定文件指定函数显示 |
| set list or listsize 行数 | 设置显示行数           |
| show list orlistsize      | 显示设置的显示行数     |

### GDB断点操作

| commend              | function                      |
| -------------------- | ----------------------------- |
| b/break 行号         | 在 行号 处打断点              |
| info break           | 查看断点信息                  |
| b/break 函数名       | 在 函数第一行 处打断点        |
| d/delete 断点编号    | 删除断点                      |
| dis/disable 断点编号 | 使断点无效                    |
| ena/enable 断点编号  | 使断点生效                    |
| b/break 行号 if 条件 | break 10 if i==3 一般用于循环 |

### GDB调试操作

| commend        | function                         |
| -------------- | -------------------------------- |
| start          | 程序停在第一行                   |
| run            | 遇到断点才停                     |
| c/continue     | 遇到下一个断点才停               |
| n/next         | 执行下一行；不会进入函数体       |
| s/step         | 向下单步调试；遇到函数进入函数体 |
| finish         | 跳出函数体                       |
| until          | 跳出循环                         |
| p/print 变量名 | 打印变量值                       |
| ptype 变量名   | 打印变量类型                     |
|                |                                  |

## 文件IO

![文件IO](.\asset\文件IO.png)

## 虚拟地址空间

![虚拟地址空间](.\asset\虚拟地址空间.png)

虚拟地址空间是不存在的，在一个可执行的文件运行时（一个进程在运行），系统会给改进程分配地址空间。

上图是以32位的机器为例的虚拟地址空间。虚拟地址空间会通过MMU映射到真实物理地址的映射。

## 文件描述符

![文件描述符](.\asset\文件描述符.png)

==文件描述符表的默认大小是1024==，每一个进程都包含一个文件描述符表，所以每一个进程最多可以打开1024个文件。

Linux中一切皆文件，是指所有的设备在Linux中都被虚拟成一个“文件”，通过“文件”来管理设备。

==同一个文件可以被多次打开，而被多次打开的文件描述符（`FILE`的结构体中的文件描述符）是不一样的。==

## Linux系统IO函数

Linux系统的函数API手册 在第二章；标准C库的函数手册 在第三章；

```cpp
int open (const char *pathname, int flags); // man 2 open
/*
pathname：要打开的文件路径
flags：对文件的权限设置（O_RDONLY, O_WRONLY, O_RDWR）
返回文件描述符，文件不存在返回-1；
*/
int open (const char *pathname, int flags, mode_t mode);
/*
pathname：要打开的文件路径
flags：程序运行时 对文件的操作权限设置（必选项：O_RDONLY, O_WRONLY, O_RDWR， 可选项若干）
mode：八进制的数，表示文件rwe权限，文件本身的权限 (read, write, and execute permission)  当前用户、当前用户所在组、其他组
文件最终权限是 mode & ~umask（umask是抹去文件的一些权限）
返回文件描述符，文件不存在返回-1；
*/
int close (int fd); 

ssize_t read(int fd, void *buf, size_t count); 
/*
fd: 文件描述符
buf: 要读取的数据存放的缓冲区，数组的地址
count: buf数组的大小
返回值是读取的数据的大小（字节数，大于零）或 0（文件读取完成了），如果出错返回-1
*/
ssize_t write (int fd, const void *buf, size_t count);
/*
fd: 文件描述符
buf: 要写入的数据的缓冲区，数组的地址
count: 要写的数据的实际大小
返回值是写入的数据的大小（字节数，大于零）或 0（没有任何数据需要写入），如果出错返回-1
*/
off_t lseek (int fd, off_t offset, int whence);
/*
lseek()  repositions  the  file  offset of the open file description associated with the file
       descriptor fd to the argument offset according to the directive whence as follows:

       SEEK_SET
              The file offset is set to offset bytes.

       SEEK_CUR
              The file offset is set to its current location plus offset bytes.

       SEEK_END
              The file offset is set to the size of the file plus offset bytes.
fd: 文件描述符
offset: 偏移量
whence: SEEK_SET   SEEK_CUR   SEEK_END
返回文件指针位置

lseek(fd, 0, SEEK_SET); //移动文件指针到文件头
lseek(fd, 0, SEEK_CUR); //获取当前文件指针的位置
lseek(fd, 0, SEEK_END); //获取文件长度
lseek(fd, 100, SEEK_END); //扩展文件长度 增加100字节 注意最后要 write(fd, " ", 1); 才会真正完成扩展
*/
int stat(const char *pathname, struct stat *statbuf);
/*
pathname: 文件路径
statbuf: 传出参数，保存文件的信息
返回值：0-成功  -1 失败
*/
```

![stat](.\asset\stat.png)

![st_mode](.\asset\st_mode.png)



## Linux文件属性操作函数

```c
int access(const char *pathname, int mode);
/*
access()  checks  whether the calling process can access the file pathname.
pathname: 文件路径
mode: F_OK tests for the existence of the file.  R_OK, W_OK, and X_OK test whether the file exists and  grants read, write, and execute permissions, respectively.
返回值: 0-success  -1 -failed
*/

int chmod(const char *pathname, mode_t mode);
/*
改变文件权限 
mode_t (0665 0777)
返回值: 0-success  -1 -failed
*/

int chown(const char *pathname, uid_t owner, gid_t group);
/*
改变拥有者
owner: 用户ID
group: 组ID
返回值: 0-success  -1 -failed
*/

int truncate(const char *path, off_t length);
/*
缩减或者扩展文件到指定大小
path: 文件路径
length: 文件最终的大小  off_t--long int
返回值: 0-success  -1 -failed
int res1 = truncate(filename, 20); 将file的文件大小改成20K。
*/
```

## Linux目录操作函数

```c
int mkdir(const char *pathname, mode_t mode);

int rmdir(const char *pathname);

int rename(const char *oldpath, const char *newpath);

int chdir(const char *path);
/*
修改进程工作目录
*/
char *getcwd(char *buf, size_t size);
/*
返回当前进程工作的绝对路径
buf: 传出参数，存储的是路径
size: buf的大小
返回值: 成功则 指向buf的地址；失败则NULL。
*/

DIR *opendir(const char *name);
    
struct dirent *readdir(DIR *dirp);
    
int closedir(DIR *dirp);
    
```



## 一些函数

```c
int dup(int oldfd);
/*
复制文件描述符
The  dup() system call creates a copy of the file descriptor oldfd, using the lowest-numbered
       unused file descriptor for the new descriptor.
返回值为新的文件描述符，指向同一个文件
*/

int dup2(int oldfd, int newfd);
/*
重定向文件描述符；用newfd去指向oldfd描述的文件。将newfd重定向到oldfd

The dup2() system call performs the same task as dup(), but instead of using the  lowest-num‐
       bered  unused file descriptor, it uses the file descriptor number specified in newfd.  If the
       file descriptor newfd was previously open, it is silently closed before being reused.
返回值为新的文件描述符，即newfd
*/

int fcntl(int fd, int cmd, ... /* arg */ );
/*
fcntl()  performs  one of the operations described below on the open file descriptor fd.  The
       operation is determined by cmd.

复制文件描述符 cmd-F_DUPFD
获取或者修改指定的文件描述符的状态。cmd-F_GETFL  cmd-F_SETFL
设置文件描述符为非阻塞:
int flags = fcntl(fd, F_GETFL);
flags |= O_NONBLOCK;
fcntl(fd, F_SETFL, flags);
*/
```



# Linux多进程开发



## 子进程创建

```c
pid_t fork(void);

/*
创建子进程，返回值会返回两次，具体内容如下。
On  success,  the  PID  of the child process is returned in the parent, and 0 is returned in the child.  On failure, -1 is returned in the parent, no child process is created, and errno is set appropriately.
*/

#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main(){
    //创建子进程
    pid_t pid = fork(); // 程序再次分叉
    if(pid > 0){
        //父进程只执行这里面的代码
        printf("fork 函数返回值： %d \n", pid);
        printf("pid: %d, ppid: %d \n", getpid(), getppid());
        printf("这是父进程\n");
    }else if(pid == 0){
        //子进程只执行这里面的代码
        printf("fork 函数返回值： %d \n", pid);
        printf("pid: %d, ppid: %d \n", getpid(), getppid());
        printf("这是子进程\n");
    }
    //父子进程共享的代码，父子进程交替运行
    for(int i = 0; i < 3333;i++){
        printf("i = %d, pid = %d\n", i, getpid());
    }

    return 0;
}
```

创建子进程之后，子进程会将父进程的 ==用户区== 的虚拟地址空间clone，==内核区除了pid==其余部分也会被clone。但是只是初始的值相同，改变各自进程里的变量的值，并不会影响另一个进程。

此外，fork函数实现的方法是 **读时共享，写时拷贝**，并不是完全复制一份完整的父进程的虚拟地址空间，只是

函数的返回值是在栈空间里的。

## 多进程GDB调试

GDB 默认跟踪父线程，当断点在父线程时，父线程阻塞，子线程正常运行。

两条命令来修改这一属性：

`set follow-fork-mode [parent | child]`  切换跟踪属性，运行到 哪个进程 的断点处阻塞

`set detach-on-fork [on | off]` 调试当前进程时，其他进程是否要等待，off为等待，on不等待。gdb-8.0+ 多线程调试有一点问题

`info inferiors` 查看调试的进程



## exec函数族

exec函数族的作用是==根据指定的文件名找到可执行文件==，并用它来取代调用进程的exec内容，换句话说，就是在调用进程内部执行一个可执行文件。
==exec函数族的函数执行成功后不会返回==，因为==调用进程的实体，包括代码段，数据段和堆栈等都已经被新的内容取代==，只留下PID等一些表面上的信息仍保持原样。颇有些神似“三十六计”中的“金蝉脱壳”。看上去还是旧的躯壳，却已经注入了新的灵魂。只有调用失败了，它们才会返 -1，从原程序的调用点接着往下执行

```cpp
#include <unistd.h>

int execl(const char *path, const char *arg, ...
                       /* (char  *) NULL */);
/*
- path:需要指定的执行的文件的 -路径- 或者 -名称-
	a.out /home/nowcoder/a.out 推荐使用绝对路径./a.out hello world
- arg:是执行可执行文件所需要的参数列表第一个参数一般没有什么作用，为了方便，一般写的是执行的程序的名称。自从第二个参数开始往后，就是程序执行所需要的的参数列表。参数最后需要以NULL结束(哨兵)
返回值:
只有当调用失败，才会有返回值，返回-1，并且设置errno如果调用成功，没有返回值。
*/

int execlp(const char *path, const char *arg, ...
                       /* (char  *) NULL */);
/*
- path:需要指定的执行的文件的 -名称-( 从环境变量里面去寻找)
	 比如 "ps"
- arg:是执行可执行文件所需要的参数列表第一个参数一般没有什么作用，为了方便，一般写的是执行的程序的名称。自从第二个参数开始往后，就是程序执行所需要的的参数列表。参数最后需要以NULL结束(哨兵)
返回值:
只有当调用失败，才会有返回值，返回-1，并且设置errno如果调用成功，没有返回值。
*/

int execle(const char *path, const char *arg, ...
                       /*, (char *) NULL, char * const envp[] */);
int execv(const char *path, char *const argv[]);
```

l(list)     参数地址列表，以空指针结尾

v(vector)     存有各参数地址的指针数组的地址        argv = {"ps", "aux", NULL}

p(path)       按PATH环境变量指定的目录搜索可执行文件

e(environment)     存有环境变量字符串地址的指针数组的地址


## 进程退出

### exit() 与 _exit()

status是int类型，代表进程退出时的状态信息，父进程回收子进程资源时会收集到这个参数。

![进程退出exit](.\asset\进程退出exit.png)

### 孤儿进程

==父进程运行结束，但子进程还在运行(未运行结束)，这样的子进程就称为孤儿进程（Orphan Process）==

每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init ，而 init进程会循环地 wait() 它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束了其生命周期的时候，init 进程就会代表之前的父进程出面处理它的一切善后工作。因此孤儿进程并不会有什么危害。

### 僵尸进程

每个进程结束之后，都会==释放自己地址空间中的用户区数据==，==内核区的 PCB 没有办法自己释放掉，需要父进程去释放==。
进程终止时，父进程尚未回收，子进程残留资源 (PCB) 存放于内核中，变成僵尸(Zombie) 进程
僵尸进程不能被 ki11 -9 杀死。
这样就会导致一个问题，==如果父进程不调用 wait() 或 waitpid() 的话==，那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，此即为僵尸进程的危害，应当避免

### 进程回收

在每个进程退出的时候，内核释放该进程所有的资源、包括打开的文件、占用的内存等。但是仍然为其保留一定的信息，这些信息主要主要指进程控制块PCB的信息(包括进程号、退出状态、运行时间等)。

父进程可以通过调用wait或waitpid得到子进程的退出状态同时彻底清除掉这个进程。

wait()和 waitpid() 函数的功能一样，区别在于，wait() 函数会阻塞，waitpid()可以设置不阻塞，waitpid() 还可以指定等待哪个子进程结束。==注意：一次wait或waitpid调用只能清理一个子进程，清理多个子进程应使用循环==



```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *wstatus);
/*
*wstatus 是传出参数，返回的是子进程退出时的状态
返回值是 结束的子进程的pid 或 -1
*/
pid_t waitpid(pid_t pid, int *wstatus, int options);
/*
功能: 回收指定进程号的子进程，可以设置是否阻塞。
参数:
pid:
pid > 0 : 某个子进程的pid
pid = 0: 回收当前进程组的所有子进程
pid = -1 : 回收所有的子进程，相当于 wait()
pid < -1 : 为某个进程组的组id的绝对值的负值，功能是回收指定进程组中的子进程
options:
0 阻塞
WNOHANG  非阻塞
返回值：
结束的子进程的pid 或 0（带有子进程活着） 或 -1 （没有子进程了或错误）
*/
```

## 进程间通信（重点）

进程是一个独立的资源分配单元，不同进程 (这里所说的进程通常指的是==用户进程==)之间的资源是独立的，没有关联，不能在一个进程中直接访问另一个进程的资源但是，进程不是孤立的，不同的进程需要进行信息的交互和状态的传递等，因此需要进程间通信( IPC: Inter Processes Communication )。

进程间通信的目的：
数据传输：一个进程需要将它的数据发送给另一个进程
通知事件：一个进程需要向另一个或一组进程发送消息，通知它 (它们) 发生了某种事件(如进程终止时要通知父进程)
资源共享：多个进程之间共享同样的资源。为了做到这一点，需要内核提供互斥和同步机制。
进程控制：有些进程希望完全控制另一个进程的执行(如 Debug 进程)，此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

## 匿名管道（pipe）

匿名管道是IPC的最古老的方式。

管道的特点：

1. 管道其实是一个在内核内存中维护的缓冲器，这个缓冲器的存储能力是有限的，不同的操作系统大小不一定相同；

2. 管道拥有文件的特质：读操作、写操作。管道两端分别对应一个文件描述符。匿名管道没有文件实体，有名管道有文件实体但不存储数据。可以按照操作文件的方式对管道进行操作。

3. 一个管道是一个字节流，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少。

4. 通过管道传递的数据是==顺序==的，从管道中读取出来的字节的顺序和它们被写入管道的顺序是==完全一样==的。

5. 在管道中的数据的==传递方向是单向的==，==一端用于写入，一端用于读取，管道是半双工的==

6. 从管道读数据是一次性操作，数据一旦被读走，它就从管道中被抛弃，释放空间以便写更多的数据，在管道中无法使用 lseek() 来随机的访问数据。

7. 匿名管道只能在==具有公共祖先的进程==(父进程与子进程，或者两个兄弟进程，具有亲缘关系) 之间使用。

8. 管道的数据结构是循环队列。也就是因为这样，才对应了特点6，读完数据之后就留出了空间交给写操作了。

   <img src=".\asset\管道的数据结构.png" alt="管道的数据结构" style="zoom:50%;" />



```c
#include <unistd.h>

int pipe(int pipefd[2]);
/*
用于创建管道，进行进程间的通信。
pipe() creates a pipe, a unidirectional data channel that can be used for inter‐
       process communication.  The array pipefd is used to return two file  descriptors
       referring  to  the  ends  of  the pipe.  pipefd[0] refers to the read end of the
       pipe.  pipefd[1] refers to the write end of the pipe.  
pipefd[2]是传出参数，表示读写两端的文件描述符
返回值 成功-0 失败- -1

管道默认是阻塞的,
*/
```

管道读写的特点：使用管道时，需要注意以下几种特殊的情况(假设都是==阻塞I/O操作==)

1. 所有的==指向管道写端的文件描述符都关闭了==(管道写端引用计数为0)，==有进程从管道的读端读数据==，那么管道中==剩余的数据==被读取以后，==再次read会返回0==，就像读到文件末尾一样。
2. 如果有==指向管道写端的文件描述符没有关闭==(管道的写端引用计数大于0) ，而==持有管道写端的进程也没有往管道中写数据==，这个时候有进程从管道中读取数据，那么==管道中剩余的数据被读取之后，再次read会阻塞==，直到管道中有数据可以读了才读取数据并返回。
3. 如果所有==指向管道读端的文件描述符都关闭了==(管道的读端引用计数为0)，这个时候有进程向管道中==写数据==，那么该进程会收到一个信号SIGPIPE，通常会导致进程异常终止。
4. 如果有==指向管道读端的文件描述符没有关闭==(管道的读端引用计数大于0) ，而==持有管道读端的进程也没有从管道中读数据==，这时有进程向管道中写数据，那么==在管道被写满的时候再次write会直到管道中有空位置才能再次写入数据并返回==。

## 有名管道(FIFO)

匿名管道，由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了有名管道 (FIFO)，也叫命名管道、FIFO文件。

有名管道 (FIFO)不同于匿名管道之处在于==它提供了一个路径名与之关联，以 FIFO的文件形式存在于文件系统中==，并且其打开方式与打开一个普通文件是一样的，这样即使与 FIFO的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过 FIFO 相互通信，因此，通过 FIFO 不相关的进程也能交换数据一旦打开了 FIFO，就能在它上面使用与操作匿名管道和其他文件的系统调用一样的I/O系统调用了 (如read()、write()和close()) 。与管道样，FIFO也有一个写入端和读取端，并且==从管道中读取数据的顺序与写入的顺序是一样的==。FIFO的名称也由此而来:先入先出。

有名管道 (FIFO)和匿名管道 (pipe) 有一些特点是相同的，不一样的地方在于

1. FIFO在文件系统中作为一个特殊文件存在，但 FIFO 中的内容却存放在内存中，所以看到的FIFO文件的数据大小为0
2. 当使用 FIFO的进程退出后，FIFO 文件将继续保存在文件系统中以便以后使用
3. FIFO 有名字，不相关的进程可以通过打开有名管道进行通信



```c
int mkfifo(const char *pathname, mode_t mode);
```

有名管道注意事项：

1. 一个为只写打开的管道会阻塞，直到另一个进程 以读打开管道

2. 一个为只读打开的管道会阻塞，直到另一个进程 以写打开管道

3. 读管道：管道中有数据，read返回实际读到的字节数。

   ​               管道中无数据：管道写端被全部关闭，read返回0，（相当于读到文件末尾）；写端没有全部被关闭，read阻塞等待

4. 写管道：管道读端全部关闭，程序异常终止 SIGPIPE

   ​				管道读端没有全部关闭：管道已满，则阻塞；管道没有满，写入数据并返回写入的数据字节数。

## 内存映射

### 页表

页表是记录虚拟内存地址和物理内存地址之间的映射关系的一张表。以32位系统为例，虚拟地址空间大小为$2^{32} bytes$，即4GB，也就是需要记录$2^{32} $个映射关系。

分页即将内存划分为固定长度的单元，每个单元就是一页（通常操作系统的使用的页大小是4KB。）。对于虚拟地址空间，分页机制将地址空间分割成固定大小的单元，每个单元称为一页。对于物理地址空间，物理内存被抽象成固定大小的单元，每个单元称为页帧(frame)。==分页机制的核心就是虚拟页号（VPN）到物理页帧号（PFN）的映射==。而==VPN到PFN的映射关系是通过**页表**记录的==。MMU通过页表记录的映射关系完成VPN到PFN的转换，即找到了页表就找到了物理地址。

页表存储在物理内存中，

https://zhuanlan.zhihu.com/p/458935522



### 函数说明

```c
void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);
/*
    - 功能：将文件或设备的数据映射到虚拟内存中；
    - 参数：addr：映射的首地址，建议为 NULL（由内核指定）
            length：要映射的数据的长度，建议使用文件的长度，不能为0，
                    如果length少于分页的长度，则自动分配一个分页的大小。
            获取文件长度方法：stat lseek
            prot：内存的操作权限。
                PROT_EXEC  Pages may be executed.
                PROT_READ  Pages may be read.
                PROT_WRITE Pages may be written.
                PROT_NONE  Pages may not be accessed.
                要操作内存区，必须拥有读权限。
            flags：
                MAP_SHARED 映射区的数据会自动和磁盘文件进行同步，进程间
                通信必须设置这个权限。
                MAP_PRIVATE 私人映射，不同步，映射区数据改变，文件数据不改变。
                写时拷贝。
                MAP_ANONYMOUS 匿名映射，不需要文件示例作
            fd：需要映射的文件的文件描述符，通过open得到。
                文件大小不能为0.
                open的指定的权限不能与port有冲突。
            offset：偏移量，一般不用，必须指定的是4k的整数倍，0表示不偏移。
*/

int munmap(void *addr, size_t length);
/*
    - 功能：释放内存映射；
    - 参数：addr 指向映射区的指针
            length 要释放的数据的长度
*/
```

### 注意事项

1. 如果对mmap的返回值(ptr)做++操作(ptr++)，munmap是否能够成功？

可以进行++操作，但是在释放的时候不能正确释放了。在释放的时候需要传入首地址。

2. 如果open时O_RDONLY，mmap时prot参数指定PROT_READ|PROT_WRITE会怎样？

会产生错误，并返回 MAP_FAILED；

3. 如果文件偏移量为1000会怎样?

偏移量必须是1024的整数倍。

4. mmap什么情况下会调用失败?

- 第二个参数 len == 0;
- 第三个参数没有 PORT_READ
- 第五个参数的fd 是在 O_RDONLY 或 O_WRONLY 情况下open的

5. 可以open的时候O_CREAT一个新文件来创建映射区吗?

可以，但是新文件的大小不能为0。

6. mmap后关闭文件描述符，对mmap映射有没有影响?

没有影响

7. 对ptr越界操作会怎样?

越界操作操作的是非法内存，会产生段错误。



## 信号

信号是 Linux进程间通信的最古老的方式之一，是事件发生时对进程的通知机制，有时也称之为软件中断，它是在软件层次上对中断机制的一种模拟，是一种异步通信的方式。

信号可以导致一个正在运行的进程被另一个正在运行的异步进程中断，转而处理某一个突发事件发往进程的诸多信号，通常都是源于内核。引发内核为进程产生信号的各类事件如下:

- 对于前台进程，用户可以通过输入特殊的终端字符来给它发送信号。比如输入ctrl+C通常会给进程发送一个中断信号。
- 硬件发生异常，即硬件检测到一个错误条件并通知内核，随即再由内核发送相应信号给相关进程。比如执行一条异常的机器语言指令，诸如被 0 除，或者引用了无法访问的内存区域。
- 系统状态变化，比如 alarm 定时器到期将引起 SIGALRM 信号，进程执行的 CPU时间超限，或者该进程的某个子进程退出
- kill命令

使用信号的两个主要目的：

- 让进程知道已经发生了一个特定的事情
- 强迫进程执行它自己代码中的信号处理程序

信号的特点：

- 简单
- 不能携带大量信息
- 满足某个特定条件才发送
- 优先级比较高

查询信号：`kill -l`，`man 7 signal`



### 函数说明

```c
int kill(pid_t pid, int sig);
/*
功能:给任何的进程或者进程组pid，发送任何的信号 sig
参数:
- pid :
> 0 : 将信号发送给指定的进程
= 0 : 将信号发送给当前的进程组
= -1 : 将信号发送给每一个有权限接收这个信号的进程
<-1 : 这个pid=某个进程组的ID取反 (-12345)
sig : 需要发送的信号的编号或者是宏值，0表示不发送任何信号
*/
int raise(int sig);
/*
功能:给当前进程发送信号 sig
参数:
sig : 需要发送的信号的编号或者是宏值，0表示不发送任何信号
*/
void abort(void);
/*
功能:给当前进程发送信号SIGABRT，杀死当前进程
sig : 需要发送的信号的编号或者是宏值，0表示不发送任何信号
*/
unsigned int alarm(unsigned int seconds);
/*
功能：设置长度为seconds的定时器
seconds到期之后发送SIGALRM的信号，如果seconds为0，表示定时器无效。
重复调用，前者失效。
返回值：
	-之前没有定时器，返回0
	-之前有定时器，返回之前定时器剩余时间
不阻塞
*/
int setitimer(int which, const struct itimerval *new_value,struct itimerval *old_value);
/*
功能：周期设置定时器
参数：
-which: 定时器统计的是哪种时间。
	 ITIMER_REAL 真实时间    This  timer  counts  down  in real (i.e., wall clock)
                      time.  At each expiration, a SIGALRM signal is gener‐
                      ated.

     ITIMER_VIRTUAL 用户时间 This timer counts down against the user-mode CPU time
                      consumed by the process.  (The  measurement  includes
                      CPU time consumed by all threads in the process.)  At
                      each expiration, a SIGVTALRM signal is generated.

     ITIMER_PROF  内核+用户时间  This timer counts down against the total (i.e.,  both
                      user  and  system)  CPU time consumed by the process.
                      (The measurement includes CPU time  consumed  by  all
                      threads  in the process.)  At each expiration, a SIG‐
                      PROF signal is generated.

	struct itimerval {
    	struct timeval it_interval; // Interval for periodic timer 
        struct timeval it_value;    // Time until next expiration 
    };
	每隔it_interval设置it_value长度的定时器
    struct timeval {
    	time_t      tv_sec;         // seconds 
        suseconds_t tv_usec;        // microseconds 
    };
不阻塞
*/
```

### 信号集

许多信号相关的系统调用都需要能表示一组不同的信号，多个信号可使用一个称之为**信号集**的数据结构来表示，其系统数据类型为 **sigset t**。

在 PCB 中有两个非常重要的信号集。一个称之为“阻塞信号集”，另一个称之为未决信号集”。这两个信号集都是内核使用位图机制来实现的。但操作系统不允许我们直接对这两个信号集进行位操作。而需自定义另外一个集合，借助信号集操作函数来对 PCB中的这两个信号集进行修改。

信号的“未决”是一种状态，指的是从信号的产生到信号被处理前的这一段时间

信号的“阻塞”是一个开关动作，指的是阻止信号被处理，但不是阻止信号产生信号的阻塞就是让系统暂时保留信号留待以后发送。

由于另外有办法让系统忽略信号所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感的操作。

### 利用SIGCHLD信号解决僵尸进程的问题

在正常情况下，父进程在执行过程中会调用`wait(pid)`来回收子进程的资源。但是如果子进程结束了，父进程并没有回收该子进程的资源，这样就会造成僵尸进程。但是由于子进程结束时会向父进程提供`SIGCHLD`信号，父进程默认忽略该信号，我们可以考虑捕捉该信号，在该信号的信号处理函数中去调用`wait(pid)`来回收子进程的资源。

用到的信号捕捉函数为

```c
 #include <signal.h>

int sigaction(int signum, const struct sigaction *act,
                     struct sigaction *oldact);
-参数：signum specifies the signal and can  be  any  valid  signal  except  SIGKILL  and
       SIGSTOP.
    
    act：这个结构体中会定义回调函数
    oldact：一般为NULL   

```

```c
/*
SIGCHLD产生的三个条件：
    1. 子进程结束了
    2. 子进程暂停了
    3. 子进程继续运行了
*/

#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <signal.h>
#include <sys/stat.h>
#include <sys/wait.h>

void myaction(int num){
    printf("捕捉到信号：%d, pid=%d\n", num, getpid());
    //wait(NULL);
    while(1){
        int ret = waitpid(-1, NULL, WNOHANG);
        if(ret > 0){
            printf("child process %d died\n", ret);
        }else if(ret == 0){
            //说明还有未结束的子进程
            break;
        }else if(ret == -1){
            break;
        }
    }
}

int main(){

    //创建五个子进程
    pid_t pid;
    for(int i = 0;i < 5;i++){
        pid = fork();
        if(pid == 0){
            break;
        }
    }

    if(pid > 0){
        struct sigaction act;
        act.sa_flags = 0;
        act.sa_handler = myaction;
        sigemptyset(&act.sa_mask);

        sigaction(SIGCHLD, &act, NULL);

        while(1){
            printf("parent process pid %d:\n", getpid());
            sleep(1);
        }
        
    }else if(pid == 0){
        //sleep(5);
        printf("child process pid %d:\n", getpid());
    }

    return 0;
}
```



## 共享内存

共享内存允许两个或者多个进程==共享物理内存的同一块区域==(通常被称为段)。由于个共享内存段会称为一个进程用户空间的一部分，因此这种 IPC 机制无需内核介入。所有需要做的就是让一个进程将数据复制进共享内存中，并且这部分数据会对其他所有共享同一个段的进程可用。

与管道等要求发送进程将数据从**用户空间的缓冲区复制进内核内存**和**接收进程将数据从内核内存复制进用户空间的缓冲区**的做法相比，这种 IPC 技术的速度更快。

### 函数使用

- 调用 shmget() 创建一个新共享内存段或取得一个既有共享内存段的标识符 (即由其他进程创建的共享内存段)。这个调用将返回后续调用中需要用到的共享内存标识符。**创建共享内存段或获取共享内存段**

  ```
  int shmget(key_t key, size_t size, int shmflg);
  功能: 创建一个新的共享内存段，或者获取一个既有的共享内存段的标识。
  新创建的内存段中的数据都会被初始化为0
  参数:
      - key : key_t类型是一个整形，通过这个找到或者创建一个共享内存。一般使用16进制表示，非0值
      - size: 共享内存的大小
      - shmflg: 共享内存的属性
      		IPC_CREAT | IPC_EXEC | 0664
  返回值:
  	-1 失败
  	>0 内存段的id
  ```

  

- 使用 shmat() 来附上共享内存段，即使该段成为调用进程的虚拟内存的一部分此刻在程序中可以像对待其他可用内存那样对待这个共享内存段。为引用这块共享内存程序需要使用由 shmat() 调用返回的 addr 值，它是一个指向进程的虚拟地址空间中该共享内存段的起点的指针。**将共享内存段与进程联系起来**

  ```
  void *shmat(int shmid, const void *shmaddr, int shmflg);
  shmid：共享内存段id
  shmaddr：链接共享内存段的起始地址，一般为NULL，由内核指定
  shmflg：
  	读：SHM_RDONLY
  	读写：0
  返回值 共享内存首地址
  ```

  

- 调用 shmdt()来分离共享内存段。在这个调用之后，进程就无法再引用这块共享内存了。这一步是可选的，并且在进程终止时会自动完成这一步。**将共享内存段与进程解除联系**

  ```
  int shmdt(const void *shmaddr);
  shmaddr：链接共享内存段的起始地址，一般为NULL，由内核指定
  返回值 0 1
  ```

  

- 用shmctl()来获取共享内存段状态。主要用于删除。只有当当前**所有附加内存段的进程都与之分离之后内存段才会销毁**。只有一个进程需要执行这一步。**删除共享内存段**

  ```
  int shmctl(int shmid, int cmd, struct shmid_ds *buf);
  shmid：共享内存段id
  cmd：IPC_STAT：获取状态，将状态写入到buf中
  	IPC_SET：将buf中的状态设置到内存段中
  	IPC_RMID：删除共享内存段
  ```




ipcs 用法

- ipcs -a    打印当前系统中所有的进程间通信方式的信息
- ipcs -m  打印出使用共享内存进行进程间通信的信息
- ipcs -q  打印出使用消息队列进行进程间通信的信息
- ipcs -s  打印出使用信号进行进程间通信的信息

ipcrm 用法

-  ipcrm -M shmkey //移除用shmkey创建的共享内存段
-  ipcrm -m shmid //移除用shmid标识的共享内存段
- ipcrm -Q msgkey // 移除用msqkey创建的消息队列
- ipcrm -q msqid // 移除用msqid标识的消息队列

- ipcrm -S semkey // 移除用semkey创建的信号
- ipcrm -s semid //移除用semid标识的信号

### 问题

1. 操作系统如何知道共享内存被多少进程关联？

   共享内存中维护了一个结构体 `struct shmid_ds`中的成员`shm_nattch`表示了被多少进程关联。

2. 可不可以多次 `shmctl`执行删除操作？

   可以，shmctl是标记删除，真正删除的时刻是连接数变为0的时候。

3. 共享内存与内存映射的区别？

   - 共享内存可以直接创建，内存映射需要依赖磁盘文件（匿名映射除外，匿名映射只能在有关系的进程通信）
   - 共享内存效率更高
   - 共享内存操作同一块物理内存，内存映射是在各自的虚拟内存对同一个文件进行映射。
   - 数据安全：
     - 进程突然退出：共享内存还存在；内存映射区消失
     - 运行进程的电脑死机，宕机了：存在在共享内存中的数据，没有了；内存映射区的数据 ，由于磁盘文件中的数据还在，所以内存映射区的数据还有。
   - 生命周期：进程退出，内存映射消失，共享内存还在，需要手动删除共享内存。如果进程退出，会自动与共享内存取消关联。

### 消息队列（补上）



## 守护进程



# 进程同步

## 锁

用于保护临界区，以保护==任何时刻只有一个线程在执行其中的代码==，其大体轮廓大体如下：

```
　　lock_the_mutex(...);

　　临界区

　　unlock_the_mutex(...);
```

下列三个函数给一个互斥锁上锁和解锁：

```
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mptr);　　//若不能立刻获得锁，将阻塞在此处

int pthread_mutex_trylock(pthread_mutex_t *mptr);　　//若不能立刻获得锁，将返回EBUSY，用户可以根据此返回值做其他操作，非阻塞模式

int pthread_mutex_unlock(pthread_mutex_t *mptr);　　//释放锁
```

互斥锁通常用于保护由多个线程或多个进程分享的共享数据(Share Data)

产生死锁的的四个条件如下：

- 互斥条件：一个资源每次只能被一个进程使用；
- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放；
- 不剥夺条件：进程已获得的资源，在没使用完之前，不能强行剥夺；
- 循环等待条件：多个进程之间形成一种互相循环等待资源的关系。

## 条件变量

```c
/*
    条件变量的类型 pthread_cond_t
    int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
    int pthread_cond_destroy(pthread_cond_t *cond);
    int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
        - 等待，调用了该函数，线程会阻塞。
    int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime);
        - 等待多长时间，调用了这个函数，线程会阻塞，直到指定的时间结束。
    int pthread_cond_signal(pthread_cond_t *cond);
        - 唤醒一个或者多个等待的线程
    int pthread_cond_broadcast(pthread_cond_t *cond);
        - 唤醒所有的等待的线程
*/
```

条件变量和互斥锁相互配合，调用`pthread_cond_wait`之后线程会在此处阻塞，并且释放对应的mutex的权限（即允许其他线程加锁并处理公共区域的数据）；当满足条件变量满足（别的线程调用了`pthread_cond_signal`），那么就该函数结束阻塞，并把对应的mutex重新加锁，继续向下运行。

## 信号量

```c
/*
    信号量的类型 sem_t
    int sem_init(sem_t *sem, int pshared, unsigned int value);
        - 初始化信号量
        - 参数：
            - sem : 信号量变量的地址
            - pshared : 0 用在线程间 ，非0 用在进程间
            - value : 信号量中的值

    int sem_destroy(sem_t *sem);
        - 释放资源

    int sem_wait(sem_t *sem);
        - 对信号量加锁，调用一次对信号量的值-1，如果值为0，就阻塞

    int sem_trywait(sem_t *sem);

    int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
    int sem_post(sem_t *sem);
        - 对信号量解锁，调用一次对信号量的值+1

    int sem_getvalue(sem_t *sem, int *sval);
*/
```

信号量的值可以简单理解为该线程可操作的空间是多大。首先对信号量加锁（调用`sem_wait`，每调用一次则信号量的值减一），如果信号量的值大于0，则不阻塞，说明该线程还可以操作该共享的空间；如果信号量的值等于0，则阻塞，直到别的线程调用了`sem_post`（每调用一次则信号量的值加一），然后唤醒正在等待该信号量值变为正数的任意线程。

# 网络编程

## IP转换函数

IP转换函数完成的是 `字符串IP -> 整数`，`网络字节序 <-> 主机字节序`的转换。

```c
#include <arpa/inet.h>
in_addr_t inet_addr (const char *cp); # 字符串IP -> 整数（in network byte order）
int inet_aton(const char *cp, struct in_addr *inp);# 字符串IP -> 整数(in network byte order)
    /*
    struct in_addr
    {
      in_addr_t s_addr;
    };
    */
char *inet_ntoa(struct in_addr in); 
```

下面这对更新的函数也能完成前面3 个函数同样的功能，并且它们同时适用IPV4 地址和 PV6 地址

```c
#include <arpa/inet.h>
int inet_pton(int af, const char *src, void *dst);
/*
af : addr family, AF_INET or AF_INET6
src : IP字符串
dst : 转换后的结果 传出参数
*/
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```



## socket函数



<img src=".\asset\tcp_socket.png" alt="tcp_socket" style="zoom: 67%;" />

```c
#include <sys/types .h>
#include <sys/socket .h>
#include <arpa/inet.h> // 包合了这个头文件，上面两个就可以省略
int socket (int domain, int type, int protocol);
功能:创建一个套接字
参数:
domain: 协议族
    AF_INET : ipv4
    AF_INET6 : ipv6
    AF_UNIX，AF_LOCAL : 本地套接字通信(进程间通信)
type: 通信过程中使用的协议类型
    SOCK_STREAM : 流式协议
    SOCK_DGRAM :报式协议
protoco1 : 具体的一个协议。一般写0
    SOCK_STREAM : 流式协议认使用 TCP
    SOCK_DGRAM :报式协议认使用 UDP

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
功能:绑定，将fd 和本地的IP + 端口进行绑定
参数:
    sockfd : 通过socket函数得到的文件描述符
    addr : 需要绑定的socket地址，这个地址封装了ip和端口号的信息
    addren : 第二个参数结构体占的内存大小
int listen(int sockfd, int backlog);// 
功能:监听这个socket上的连接
参数:
    sockfd : 通过socket函数得到的文件描述符
    backlog : 未连接的和已连接（半连接和全连接队列）的和的最大值，/proc/sys/net/core/somaxconn
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
功能:接收客户端连接，默认是一个阻塞的函数，阻塞等待客户端连接
参数:
    sockfd:用于监听的文件描述符
    addr : 传出参数，记录了连接成功后客户端的地址信息(ip，port)
    addrlen : 指定第二个参数的对应的内存大小
返回值:
    成功:用于通信的文件描述符
    -1 : 失败
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
功能：客户端连接服务器
参数:
	sockfd:用于通信的文件描述符
    addr : 客户端要连接的服务器的地址信息
    addrlen : 第二个参数的内存大小
        

ssize_t write(int fd, const void *buf, size_t count);
ssize_t read(int fd, void *buf, size_t count);
```



## tcp-client/sever的bug1

如果是下面的情况，现象不正常：客户端正常发送完毕数据之后，服务器端并没有跳出循环。

```c
//client
int count = 0;
while(count <= 2){
    count++;

    char * sendBuf = "I am Client\n";
    write(fd_sock, sendBuf, strlen(sendBuf));
    sleep(1);
    
}
char recvBuf[1024] = {};
int len_recv = read(fd_sock, recvBuf, sizeof(recvBuf));
printf("len_recv = %d, sizeof(recvBuf) == %ld\n", len_recv, sizeof(recvBuf));
if(len_recv == -1){
    perror("read");
    exit(-1);
}else if(len_recv > 0){
    printf("client recv: %s\n", recvBuf);
}else if(len_recv == 0){
    printf("sever close\n");

}
```

```c
//sever
char recvBuf[1024] = {0};
while(1){
    int len = read(cfd, recvBuf, sizeof(recvBuf));
    printf("recvBuf: %s\n", recvBuf);
    printf("len = %d, sizeof(recvBuf) == %ld\n", len, sizeof(recvBuf));
    if(len == -1){
        perror("read");
        exit(-1);
    }else if(len > 0){
        printf("sever recv: %s\n", recvBuf);

    }else if(len == 0){
        printf("client close\n");
        break;
    }
    
}

char * sendBuf = "I am Sever\n";
write(cfd, sendBuf, strlen(sendBuf));
```

原因是：read函数是默认阻塞的。客户端发送完数据之后，并没有马上结束进程，所以对这个socket仍然保留着`WR`权限，那么客户端就会在读这个socket的时候阻塞，不会正常退出。直到客户端结束进程，服务器端read返回0，代表读到EOF，服务端程序继续往下运行。

## 多进程并发服务器





## 多线程并发服务器

CPU 密集型 和 IO密集型 的区别，如何确定线程池大小？

- IO密集型：大量网络，文件操作 
- CPU 密集型：大量计算，cpu 占用越接近 100%, 耗费多个核或多台机器。单核CPU处理CPU密集型程序，就不要使用多线程了。多核CPU处理CPU密集型程序才合适，而且中间可能没有线程的上下文切换（一个核心处理一个线程）。

最佳线程数目 = （线程等待时间与线程CPU时间之比 + 1）* CPU数目

https://cloud.tencent.com/developer/article/1806245

## 端口复用

端口复用最常用的用途是:

- 防止服务器端断开又重启的时候时之前绑定的端口还未释放，而导致无法监听该端口
- 防止程序突然退出而系统没有释放端口，而导致无法监听该端口



```
#include <sys/types .h>
#include <sys/socket .h>
//设置sock属性，不一定是端口复用
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

//设置sock端口复用, 要再绑定之前设置
int reuse = 1;
int setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &reuse, sizeof(reuse));


常看网络相关信息的命令
netstat
参数
-a 所有的socket
-p 显示正在使用socket的程序的名称
-n 直接使用IP地址，而不通过域名服务器
```





## IO多路复用

### select

主旨思想：
1. 首先要构造一个关于文件描述符的列表，将要监听的文件描述符添加到该列表中。
2. 调用一个系统函数，监听该列表中的文件描述符，直到这些描述符中的一个或者多个进行I/O操作时，该函数才返回。
a.这个函数是阻塞
b.函数对文件描述符的检测的操作是由内核完成的
3. 在返回时，它会告诉进程有多少（哪些）描述符要进行I/O操作

```c
// sizeof(fd_set) = 128 1024
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds,
fd_set *exceptfds, struct timeval *timeout);
- 参数：
    - nfds : 委托内核检测的最大文件描述符的值 + 1
    - readfds : 要检测的文件描述符的读的集合，委托内核检测哪些文件描述符的读的属性
    - 一般检测读操作
    - 对应的是对方发送过来的数据，因为读是被动的接收数据，检测的就是读缓冲
    区
    - 是一个传入传出参数
    - writefds : 要检测的文件描述符的写的集合，委托内核检测哪些文件描述符的写的属性
    - 委托内核检测写缓冲区是不是还可以写数据（不满的就可以写）
    - exceptfds : 检测发生异常的文件描述符的集合
    - timeout : 设置的超时时间
        struct timeval {
        long tv_sec; /* seconds */
        long tv_usec; /* microseconds */
        };
        - NULL : 永久阻塞，直到检测到了文件描述符有变化
        - tv_sec = 0 tv_usec = 0， 不阻塞
        - tv_sec > 0 tv_usec > 0， 阻塞对应的时间
- 返回值 :
    - -1 : 失败
    - >0(n) : 检测的集合中有n个文件描述符发生了变化
// 将参数文件描述符fd对应的标志位设置为0
void FD_CLR(int fd, fd_set *set);
// 判断fd对应的标志位是0还是1， 返回值 ： fd对应的标志位的值，0，返回0， 1，返回1
int FD_ISSET(int fd, fd_set *set);
// 将参数文件描述符fd 对应的标志位，设置为1
void FD_SET(int fd, fd_set *set);
// fd_set一共有1024 bit, 全部初始化为0
void FD_ZERO(fd_set *set);
```

### poll

与select的思路一致，也需要遍历文件描述符组才可以确定是哪些文件描述符对应的缓冲区的状态发生了变化

 poll 和 select 并没有太大的本质区别，**都是使用「线性结构」存储进程关注的 Socket 集合，因此都需要遍历文件描述符集合来找到可读或可写的 Socket，时间复杂度为 O(n)，而且也需要在用户态与内核态之间拷贝文件描述符集合**

### epoll

#### 原理

*第一点*，epoll 在内核里使用**红黑树来跟踪进程所有待检测的文件描述字**，把需要监控的 socket 通过 `epoll_ctl()` 函数加入内核中的红黑树里，红黑树是个高效的数据结构，增删改一般时间复杂度是 `O(logn)`。而 select/poll 内核里没有类似 epoll 红黑树这种保存所有待检测的 socket 的数据结构，所以 select/poll 每次操作时都传入整个 socket 集合给内核，而 epoll 因为在内核维护了红黑树，可以保存所有待检测的 socket ，所以只需要传入一个待检测的 socket，减少了内核和用户空间大量的数据拷贝和内存分配。

*第二点*， epoll 使用**事件驱动**的机制，内核里**维护了一个链表来记录就绪事件**，当某个 socket 有事件发生时，通过**回调函数**内核会将其加入到这个就绪事件列表中，当用户调用 `epoll_wait()` 函数时，**只会返回有事件发生的文件描述符的个数**，不需要像 select/poll 那样轮询扫描整个 socket 集合，大大提高了检测的效率。

虽然是只是返回的文件描述符的个数n，但是内核维护的红黑树会将有事件发生的文件描述符转移到树的头部，因此用户可以访问前n个文件描述符即可。

#### epoll的边缘触发与水平触发

水平触发：LT（level - triggered）是缺省的工作方式，并且同时支持 block 和 no-block socket。在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的 fd 进行 IO 操作。如果你不作任何操作，内核还是会继续通知你的。

读缓冲区有数据 - > epoll检测到了会给用户通知

- a.用户不读数据，数据一直在缓冲区，epoll 会一直通知
- b.用户只读了一部分数据，epoll会通知
- c.缓冲区的数据读完了，不通知



边缘触发：ET（edge - triggered）是高速工作方式，只支持 no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了。但是请注意，如果一直不对这个 fd 作 IO 操作（从而导致它再次变成未就绪），内核不会发送更多的通知（only once）。
ET 模式在很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。epoll工作在 ET 模式的时候，**必须使用非阻塞套接口**，以**避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死**。（==因为ET模式下，read一个文件描述符的时候需要循环去读cfd的缓冲区，如果cfd是阻塞IO，那么会阻塞在这里？如果缓冲区有新数据到来，程序将无法及时获取到这些新数据，从而出现读取不完整的情况。==）

读缓冲区有数据 - > epoll检测到了会给用户通知

- a.用户不读数据，数据一致在缓冲区中，epoll下次检测的时候就不通知了
- b.用户只读了一部分数据，epoll不通知
- c.缓冲区的数据读完了，不通知



# 项目实战

## 阻塞+非阻塞，同步+异步

一个典型的网络IO接口调用，分为两个阶段，**分别是“数据就绪” 和 “数据读写**”，数据就绪阶段分为阻塞和非阻塞，表现得结果就是，阻塞当前线程或是直接返回。

同步表示A向B请求调用一个网络IO接口时（或者调用某个业务逻辑API接口时），数据的读写都是由请求方A自己来完成的，即应用程序自己把数据从缓冲区搬送到应用程序（不管是阻塞还是非阻塞）；

异步表示A向B请求调用一个网络IO接口时（或者调用某个业务逻辑API接口时），向B传入请求的事件以及事件发生时通知的方式，A就可以处理其它逻辑了，当B监听到事件处理完成后，会用事先约定好的通知方式，通知A处理结果。进一步来说，应用程序A并不用自己花时间来进行数据的读写，而是把这个任务交给操作系统B（告知操作系统请求的事件以及事件发生时通知的方式），操作系统完成后，会按照约定好的通知方式告知应用程序。

![阻塞、非阻塞、同步、异步](.\asset\阻塞、非阻塞、同步、异步.png)

read、recv什么时候会阻塞？见匿名管道部分