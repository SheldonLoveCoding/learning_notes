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

