已在Ubuntu 20.04下测试通过，可以直接下载本代码，并参考“处理apue.h”拷贝指定文件到/usr/include目录下

1. 依赖安装`sudo apt-get install libbsd-dev`

2. 解压src.3e.tar.gz `tar -zxvf src.3e.tar.gz`

3. 编译

   ```shell
   cd apue.3e/
   sudo make 
   ```

可能遇到的问题：

**undefined reference to `major'**

```
devrdev.c: In function ‘main’:
devrdev.c:19:25: warning: implicit declaration of function ‘major’ [-Wimplicit-function-declaration]
   19 |   printf("dev = %d/%d", major(buf.st_dev),  minor(buf.st_dev));
      |                         ^~~~~
devrdev.c:19:45: warning: implicit declaration of function ‘minor’ [-Wimplicit-function-declaration]
   19 |   printf("dev = %d/%d", major(buf.st_dev),  minor(buf.st_dev));
      |                                             ^~~~~
/usr/bin/ld: /tmp/cc4fVQ7j.o: in function `main':
devrdev.c:(.text+0xc5): undefined reference to `minor'
/usr/bin/ld: devrdev.c:(.text+0xdb): undefined reference to `major'
/usr/bin/ld: devrdev.c:(.text+0x128): undefined reference to `minor'
/usr/bin/ld: devrdev.c:(.text+0x13e): undefined reference to `major'
collect2: error: ld returned 1 exit status
make[1]: *** [Makefile:18：devrdev] 错误 1
make[1]: 离开目录“/home/leetcode/CLionProjects/src.3e/apue.3e/filedir”
make: *** [Makefile:6：all] 错误 1
```

找到报错devrdev.c文件，在#include "apue.h"下方添加`#include <sys/sysmacros.h>`

**‘FILE’ {aka ‘struct _IO_FILE’} has no member named ‘__pad’**

```
make[1]: 进入目录“/home/leetcode/CLionProjects/apue.3e/stdio”
gcc -ansi -I../include -Wall -DLINUX -D_GNU_SOURCE  buf.c -o buf  -L../lib -lapue 
buf.c: In function ‘is_unbuffered’:
buf.c:90:15: error: ‘FILE’ {aka ‘struct _IO_FILE’} has no member named ‘__pad’; did you mean ‘__pad5’?
   90 | #define _flag __pad[4]
      |               ^~~~~
buf.c:98:13: note: in expansion of macro ‘_flag’
```

修改 ./stdio/buf.c 文件

将return(fp->flag & IONBF);  修改为   return(fp->flags & IONBF);
return(fp->base - fp->ptr);     修改为   return(fp->IO_buf_end - fp->IO_buf_base);

**处理apue.h**

书中大量案例都使用到了apue.h头文件，但是这个头文件是作者自封的，不是C/C++官方的头文件。因此gcc/g++无法在默认搜索路径下找到这个文件。一劳永逸的方法如下

```
cd apue.3e/
sudo cp include/apue.h /usr/include/
sudo cp lib/error.c /usr/include/ 
sudo cp lib/libapue.a /usr/lib
```

如果编译书中案例时（如第一个ls测试程序），报错

```
test.c:(.text+0x26): undefined reference to `err_quit'
/usr/bin/ld: test.c:(.text+0x63): undefined reference to `err_sys'
```

在/usr/include/apue.h文件末尾的#endif /* _APUE_H */前面添加代码#include "error.c"