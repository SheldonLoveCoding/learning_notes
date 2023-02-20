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
重定向文件描述符；用newfd去指向oldfd描述的文件。
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



