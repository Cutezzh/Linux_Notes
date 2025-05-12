### 标准I/O库简介

1. 标准 I/O 库函数是构建于文件 I/O这些系统调用之上的

2. 标准 I/O 和文件 I/O 的区别如下：  

   标准 I/O 是标准 C 库函数，而文件 I/O 则是Linux系统调用
   标准 I/O 是由文件 I/O 封装而来，标准 I/O 内部实际上是调用文件 I/O 来完成实际操作
   标准 I/O 相比于文件 I/O具有更好的可移植性，通常对于不同的操作系统
   标准 I/O是带有缓存的，而文件I/O 在用户空间是不带有缓存的，所以在性能、效率上，标准 I/O 要优于文件 I/O

3. 使用标准 I/O 库函数打开或创建一个文件时，会返回一个指向 FILE 类型对象的指针

4. 在标准 I/O 中，可以使用 stdin、 stdout、 stderr 来表示标准输入、标准输出和标准错误  

### 打开与关闭文件

```c
#include <stdio.h>
FILE *fopen(const char *path, const char *mode);
//path： 参数 path 指向文件路径，可以是绝对路径、也可以是相对路径。
//mode： 参数 mode 指定了对该文件的读写权限，是一个字符串。
//返回值： 调用成功返回一个指向FILE 类型对象的指针（FILE *），该指针与打开或创建的文件相关联，后续的标准 I/O 操作将围绕 FILE 指针进行。如果失败则返回 NULL，并设置 errno 以指示错误原因。

#include <stdio.h>
int fclose(FILE *stream);
//stream 为 FILE 类型指针，调用成功返回 0；失败将返回 EOF（也就是-1），并且会设置 errno 来指示错误原因
```

mode字符串类型，如下

| mode | 说明                                                         | 对应open的flags                 |
| ---- | ------------------------------------------------------------ | ------------------------------- |
| r    | 只读                                                         | O_RDONLY                        |
| r+   | 可读、可写                                                   | O_RDWR                          |
| w    | 只写;path 指定的文件存在，将文件长度截断为 0；如果指定文件不存在则创建该文件 | O_WRONLY \| O_CREAT \| O_TRUNC  |
| w+   | 可读、可写;path 指定的文件存在，将文件长度截断为 0；如果指定文件不存在则创建该文件 | O_RDWR\| O_CREAT \| O_TRUNC     |
| a    | 只写;打开以进行追加内容（在文件末尾写入），如果文件不存在则创建该文件 | O_WRONLY \| O_CREAT \| O_APPEND |
| a+   | 可读、可写;打开以进行追加内容（在文件末尾写入），如果文件不存在则创建该文件 | O_RDWR \| O_CREAT \| O_APPEND   |

调用 fopen()函数新建文件时无法手动指定文件的权限，有一个默认值  
```c
S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH (0666)
```

### 读文件和写文件

```c
#include <stdio.h>
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
//ptr：fread()将读取到的数据存放在参数 ptr 指向的缓冲区中
//size：fread()从文件读取 nmemb 个数据项，每一个数据项的大小为 size 个字节，所以总共读取的数据大小为 nmemb * size 个字节
//nmemb：指定了读取数据项的个数
//stream：FILE 指针
//返回值：调用成功时返回读取到的数据项的数目（数据项数目并不等于实际读取的字节数，除非参数size 等于 1）；如果发生错误或到达文件末尾，则 fread()返回的值将小于参数 nmemb，可以使用 ferror()或 feof()函数来判断

#include <stdio.h>
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
//ptr：将参数 ptr 指向的缓冲区中的数据写入到文件中。
//size：参数 size 指定了每个数据项的字节大小，与 fread()函数的 size 参数意义相同。
//nmemb：参数 nmemb 指定了写入的数据项个数，与 fread()函数的 nmemb 参数意义相同。
//stream：FILE 指针。
//返回值： 调用成功时返回写入的数据项的数目（数据项数目并不等于实际写入的字节数，除非参数 size等于 1）；如果发生错误，则 fwrite()返回的值将小于参数 nmemb（或者等于 0）
```

### fseek定位

```c
#include <stdio.h>
int fseek(FILE *stream, long offset, int whence);
//stream： FILE 指针。
//offset： 与 lseek()函数的 offset 参数意义相同。
//whence： 与 lseek()函数的 whence 参数意义相同。
//返回值： 成功返回 0；发生错误将返回-1，并且会设置 errno 以指示错误原因；与 lseek()函数的返回值意义不同！
```

```c
#include <stdio.h>
long ftell(FILE *stream);
```

通过fseek到达文件末尾，在使用ftell获取当前的偏移量，就能得到整个文件的大小

### 检查或复位状态  

```c
#include <stdio.h>
int feof(FILE *stream);

//示例
if (feof(file)) {
/* 到达文件末尾 */
}
else {
/* 未到达文件末尾 */
}
```

库函数 feof()用于测试参数 stream 所指文件的 end-of-file 标志，如果 end-of-file 标志被设置了，则调用feof()函数将返回一个非零值，如果 end-of-file 标志没有被设置，则返回 0  

```c
#include <stdio.h>
int ferror(FILE *stream);

//示例
if (ferror(file)) {
/* 发生错误 */
}
else {
/* 未发生错误 */
}
```

当对文件的 I/O 操作发生错误时，错误标志将会被设置  

```c
#include <stdio.h>
void clearerr(FILE *stream);
//此函数没有返回值，调用将总是会成功！
```

库函数 clearerr()用于清除 end-of-file 标志和错误标志，当调用 feof()或 ferror()校验这些标志后，通常需要清除这些标志，避免下次校验时使用到的是上一次设置的值，此时可以手动调用 clearerr()函数清除标志  

对于 end-of-file 标志，除了使用 clearerr()显式清除之外，当调用 fseek()成功时也会清除文件的 end-offile 标志  

### 格式化I/O

**格式化输出**

```c
#include <stdio.h>
int printf(const char *format, ...);//函数调用成功返回打印输出的字符数；失败将返回一个负值！
int fprintf(FILE *stream, const char *format, ...);//函数调用成功返回写入到文件中的字符数；失败将返回一个负值！
int dprintf(int fd, const char *format, ...);//函数调用成功返回写入到文件中的字符数；失败将返回一个负值！
int sprintf(char *buf, const char *format, ...);//函数调用成功返回写入到 buf 中的字节数；失败将返回一个负值！
int snprintf(char *buf, size_t size, const char *format, ...);//函数调用成功返回写入到 buf 中的字节数；失败将返回一个负值！
```

printf()函数用于将格式化数据写入到标准输出； 

dprintf()和 fprintf()函数用于将格式化数据写入到指定的文件中，两者不同之处在于， fprintf()使用 FILE 指针指定对应的文件、而 dprintf()则使用文件描述符 fd 指定对应的文件； 

sprintf()、 snprintf()函数可将格式化的数据存储在用户指定的缓冲区 buf 中,在字符串末尾自动添加终止字符'\0'; snprintf()函数解决了可能缓冲区溢出的问题。  

**格式化输入**

```c
#include <stdio.h>
int scanf(const char *format, ...);
int fscanf(FILE *stream, const char *format, ...);
int sscanf(const char *str, const char *format, ...);
```

scanf()函数可将用户输入（标准输入）的数据进行格式化转换； 

fscanf()函数从 FILE 指针指定文件中读取数据，并将数据进行格式化转换； 

sscanf()函数从参数 str 所指向的字符串中读取数据，并将数据进行格式化转换。  

### I/O缓冲

**内核缓冲**

1. read()和 write()系统调用在进行文件读写操作的时候并不会直接访问磁盘设备，而是仅仅在用户空间缓冲区和内核缓冲区之间复制数据，拷贝完成之后函数就返回了，在后面的某个时刻，内核会将其缓冲区中的数据写入（刷新）到磁盘设备中  

2.   系统调用 write()与磁盘操作并不是同步的，如果期间read()要读取这个文件的这几个字节，内核就会从缓冲区读取这几个字节返回

3. 内核缓冲区就称为文件 I/O 的内核缓冲。这样的设计，目的是为了提高文件 I/O 的速度和效率，使得系统调用 read()、 write()的操作更为快速，不需要等待磁盘操作

4. 内核会分配尽可能多的内核来作为文件 I/O 的内核缓冲区，但受限于物理内存的总量

5. 控制文件 I/O 内核缓冲的系统调用
   ```c  
   #include <unistd.h>
   int fsync(int fd);
   ```

   系统调用 fsync()将参数 fd 所指文件的内容数据和元数据写入磁盘，只有在对磁盘设备的写入操作完成之后， fsync()函数才会返回  

   ```c
   #include <unistd.h>
   int fdatasync(int fd);
   ```

   系统调用 fdatasync()与 fsync()类似，不同之处在于 fdatasync()仅将参数 fd 所指文件的内容数据写入磁盘，并不包括文件的元数据  

   ```c
   #include <unistd.h>
   void sync(void);
   ```

   系统调用 sync()会将所有文件 I/O 内核缓冲区中的文件内容数据和元数据全部更新到磁盘设备中，该函数没有参数、也无返回值，意味着它不是对某一个指定的文件进行数据更新，而是刷新所有文件 I/O 内核缓冲区  

   在程序中频繁调用 fsync()、 fdatasync()、 sync()（或者调用 open 时指定 O_DSYNC 或 O_SYNC 标志）对性能的影响极大  

**直接 I/O：绕过内核缓冲  **  

使用直接 I/O 可能会大大降低性能，这是因为为了提高 I/O 性能，内核针对文件 I/O 内核缓冲区做了不少的优化  

直接 I/O 只在一些特定的需求场合，譬如磁盘速率测试工具、数据库系统等  

\_GNU\_SOURCE 宏可用于开启/禁用 Linux 系统调用和 glibc 库函数的一些功能、特性，要打开这些特性，需要在应用程序中定义该宏，定义该宏之后意味着用户应用程序打开了所有的特性  

gcc 的-D 选项可用于定义一个宏，并且该宏定义在整个源码工程中都是生效的，是一个全局宏定义  

**stdio缓冲**

标准 I/O 所维护的 stdio 缓冲是用户空间的缓冲区，当应用程序中通过标准 I/O 操作磁盘文件时，为了减少调用系统调用的次数，标准 I/O 函数会将用户写入或读取文件的数据缓存在 stdio 缓冲区，然后再一次性将 stdio 缓冲区中缓存的数据通过调用系统调用 I/O（文件 I/O）写入到文件 I/O 内核缓冲区或者拷贝到应用程序的 buf 中  

```c
#include <stdio.h>
int setvbuf(FILE *stream, char *buf, int mode, size_t size);
//stream： FILE 指针，用于指定对应的文件，每一个文件都可以设置它对应的 stdio 缓冲区。
//buf： 如果参数 buf 不为 NULL，那么 buf 指向 size 大小的内存区域将作为该文件的 stdio 缓冲区，所以应该以动态或静态的方式在堆中为该缓冲区分配一块空间，而不是分配在栈上的函数内的自动变量（局部变量）。
/*
mode： 参数 mode 用于指定缓冲区的缓冲类型，可取值如下：
_IONBF： 不对 I/O 进行缓冲（无缓冲）。
_IOLBF： 采用行缓冲 I/O。当在输入或输出中遇到换行符"\n"时，标准 I/O 才会执行文件 I/O 操作。
_IOFBF： 采用全缓冲 I/O。在这种情况下，在填满 stdio 缓冲区后才进行文件 I/O 操作（read、write）。*/
//size： 指定缓冲区的大小。
//返回值： 成功返回 0，失败将返回一个非 0 值，并且会设置 errno 来指示错误原因
```

```c
#include <stdio.h>
void setbuf(FILE *stream, char *buf);
//要么将 buf 设置为 NULL 以表示无缓冲，要么指向由调用者分配的 BUFSIZ 个字节大小的缓冲区
//BUFSIZ 定义于头文件<stdio.h>中，该值通常为 8192
```

```c
#include <stdio.h>
void setbuffer(FILE *stream, char *buf, size_t size);
//setbuffer()函数类似于 setbuf()，但允许调用者指定 buf 缓冲区的大小
```

标准输出默认是行缓冲模式，只有输出了换行符时，才会将换行符这一行字符进行输出显示（也就是刷入到内核缓冲区），在没有输出换行符之前，会将数据缓存在 stdio缓冲区中  

```c
#include <stdio.h>
int fflush(FILE *stream);
//参数 stream 指定需要进行强制刷新的文件，如果该参数设置为 NULL，则表示刷新所有的 stdio 缓冲区。
//函数调用成功返回 0，否则将返回-1，并设置 errno 以指示错误原因
```

当程序退出时，使用 exit()、return 或不显式调用相关函数会自动刷新 stdio 缓冲区;使用\_exit 或\_Exit()终止程序则不会刷新  

调用 fflush()库函数可强制刷新指定文件的 stdio 缓冲区；

调用 fclose()关闭文件时会自动刷新文件的 stdio 缓冲区；

程序退出时会自动刷新 stdio 缓冲区（注意区分不同的情况）

### 文件描述符与 FILE 指针互转  

```c
#include <stdio.h>
int fileno(FILE *stream);
//根据传入的 FILE 指针得到整数文件描述符，通过返回值得到文件描述符
//转换错误将返回-1，并且会设置 errno 来指示错误原因
FILE *fdopen(int fd, const char *mode);
//给定一个文件描述符，得到该文件对应的 FILE 指针
//mode参数与文件描述符 fd 的访问模式不一致，则会导致调用 fdopen()失败
```

