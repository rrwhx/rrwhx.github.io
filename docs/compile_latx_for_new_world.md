# Compile LATX for new world

## error: `__NR_futex` undeclared

```sh
In file included from ../util/qemu-thread-posix.c:346:
/home/lxy/lat/include/qemu/futex.h: In function ‘qemu_futex_wake’:
/home/lxy/lat/include/qemu/futex.h:20:46: error: ‘__NR_futex’ undeclared (first use in this function)
   20 | #define qemu_futex(...)              syscall(__NR_futex, __VA_ARGS__)
      |                                              ^~~~~~~~~~
/home/lxy/lat/include/qemu/futex.h:24:5: note: in expansion of macro ‘qemu_futex’
   24 |     qemu_futex(f, FUTEX_WAKE, n, NULL, NULL, 0);
      |     ^~~~~~~~~~
/home/lxy/lat/include/qemu/futex.h:20:46: note: each undeclared identifier is reported only once for each function it appears in
   20 | #define qemu_futex(...)              syscall(__NR_futex, __VA_ARGS__)
      |                                              ^~~~~~~~~~
/home/lxy/lat/include/qemu/futex.h:24:5: note: in expansion of macro ‘qemu_futex’
   24 |     qemu_futex(f, FUTEX_WAKE, n, NULL, NULL, 0);
      |     ^~~~~~~~~~
/home/lxy/lat/include/qemu/futex.h: In function ‘qemu_futex_wait’:
/home/lxy/lat/include/qemu/futex.h:20:46: error: ‘__NR_futex’ undeclared (first use in this function)
   20 | #define qemu_futex(...)              syscall(__NR_futex, __VA_ARGS__)
      |                                              ^~~~~~~~~~
/home/lxy/lat/include/qemu/futex.h:29:12: note: in expansion of macro ‘qemu_futex’
   29 |     while (qemu_futex(f, FUTEX_WAIT, (int) val, NULL, NULL, 0)) {
      |            ^~~~~~~~~~
```

### compile command

```sh
cc -Ilibqemuutil.a.p -I. -I.. -Iqapi -Itrace -Iui/shader -I/usr/include/glib-2.0 -I/usr/lib/glib-2.0/include -I/usr/include/sysprof-4 \
    -fdiagnostics-color=auto -Wall -Winvalid-pch -std=gnu99 -O2 -isystem /home/lxy/lat/linux-headers -isystem linux-headers -iquote . -iquote /home/lxy/    lat -iquote /home/lxy/lat/include -iquote /home/lxy/lat/disas/libvixl -iquote /home/lxy/lat/tcg/loongarch \
    -pthread -ffunction-sections -fdata-sections -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE \
    -Wstrict-prototypes -Wredundant-decls -Wundef -Wwrite-strings -Wmissing-prototypes -fno-strict-aliasing -fno-common -fwrapv \
    -Wold-style-declaration -Wold-style-definition -Wtype-limits -Wformat-security -Wformat-y2k -Winit-self -Wignored-qualifiers -Wempty-body   -Wnested-externs -Wendif-labels -Wexpansion-to-defined -Wimplicit-fallthrough=2 -Wno-missing-include-dirs -Wno-shift-negative-value -Wno-psabi \
    -fstack-protector-strong -DNDEBUG -fPIE -MD -MQ \
    libqemuutil.a.p/util_qemu-thread-posix.c.o -MF libqemuutil.a.p/util_qemu-thread-posix.c.o.d -o libqemuutil.a.p/util_qemu-thread-posix.c.o \
    -c ../util/qemu-thread-posix.c
```

### source code

```c
#include <sys/syscall.h>
#include <linux/futex.h>

#define qemu_futex(...)              syscall(__NR_futex, __VA_ARGS__)

static inline void qemu_futex_wake(void *f, int n)
{
    qemu_futex(f, FUTEX_WAKE, n, NULL, NULL, 0);
}
```

### /usr/include/asm-generic/unistd.h

```c
#if defined(__ARCH_WANT_TIME32_SYSCALLS) || __BITS_PER_LONG != 32
#define __NR_futex 98
__SC_3264(__NR_futex, sys_futex_time32, sys_futex)
#endif
```

### /usr/include/asm-generic/bitsperlong.h

```c
#if defined(__CHAR_BIT__) && defined(__SIZEOF_LONG__)
#define __BITS_PER_LONG (__CHAR_BIT__ * __SIZEOF_LONG__)
#else
/*
 * There seems to be no way of detecting this automatically from user
 * space, so 64 bit architectures should override this in their
 * bitsperlong.h. In particular, an architecture that supports
 * both 32 and 64 bit user space must not rely on CONFIG_64BIT
 * to decide it, but rather check a compiler provided macro.
 */
#define __BITS_PER_LONG 32
#endif
```

`__CHAR_BIT__` and `__SIZEOF_LONG__`

```sh
> echo | gcc -dM -E - | grep __CHAR_BIT__
#define __CHAR_BIT__ 8
> echo | gcc -dM -E - | grep __SIZEOF_LONG__
#define __SIZEOF_LONG__ 8
```

###  test_syscall.c

```c
#include <stdio.h>

#include <unistd.h>
#include <sys/syscall.h>
#include <linux/futex.h>

int main(int argc, char** argv) {


    printf("__NR_futex : %d\n", __NR_futex);
    printf("__BITS_PER_LONG : %d\n", __BITS_PER_LONG);

    return  0;
}
```

```sh
gcc test_syscall.c -o test_syscall && ./test_syscall
__NR_futex : 98
__BITS_PER_LONG : 64
```

## 控制变量

### 是不是源文件有问题？

将要编译的文件换为 `test_syscall.c`

```sh
cc -Ilibqemuutil.a.p -I. -I.. -Iqapi -Itrace -Iui/shader -I/usr/include/glib-2.0 -I/usr/lib/glib-2.0/include -I/usr/include/sysprof-4 \
    -fdiagnostics-color=auto -Wall -Winvalid-pch -std=gnu99 -O2 -isystem /home/lxy/lat/linux-headers -isystem linux-headers -iquote . -iquote /home/lxy/    lat -iquote /home/lxy/lat/include -iquote /home/lxy/lat/disas/libvixl -iquote /home/lxy/lat/tcg/loongarch \
    -pthread -ffunction-sections -fdata-sections -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE \
    -Wstrict-prototypes -Wredundant-decls -Wundef -Wwrite-strings -Wmissing-prototypes -fno-strict-aliasing -fno-common -fwrapv \
    -Wold-style-declaration -Wold-style-definition -Wtype-limits -Wformat-security -Wformat-y2k -Winit-self -Wignored-qualifiers -Wempty-body   -Wnested-externs -Wendif-labels -Wexpansion-to-defined -Wimplicit-fallthrough=2 -Wno-missing-include-dirs -Wno-shift-negative-value -Wno-psabi \
    -fstack-protector-strong -DNDEBUG -fPIE -MD -MQ \
    libqemuutil.a.p/util_qemu-thread-posix.c.o -MF libqemuutil.a.p/util_qemu-thread-posix.c.o.d -o libqemuutil.a.p/util_qemu-thread-posix.c.o \
    -c ~/test_syscall.c

error: `__NR_futex` undeclared

```

不是源文件的问题。


### 还有什么不一样的，编译选项？

`-isystem /home/lxy/lat/linux-headers` 很可疑， 删掉。

```sh
cc -Ilibqemuutil.a.p -I. -I.. -Iqapi -Itrace -Iui/shader -I/usr/include/glib-2.0 -I/usr/lib/glib-2.0/include -I/usr/include/sysprof-4 \
    -fdiagnostics-color=auto -Wall -Winvalid-pch -std=gnu99 -O2 -isystem linux-headers -iquote . -iquote /home/lxy/    lat -iquote /home/lxy/lat/include -iquote /home/lxy/lat/disas/libvixl -iquote /home/lxy/lat/tcg/loongarch \
    -pthread -ffunction-sections -fdata-sections -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE \
    -Wstrict-prototypes -Wredundant-decls -Wundef -Wwrite-strings -Wmissing-prototypes -fno-strict-aliasing -fno-common -fwrapv \
    -Wold-style-declaration -Wold-style-definition -Wtype-limits -Wformat-security -Wformat-y2k -Winit-self -Wignored-qualifiers -Wempty-body   -Wnested-externs -Wendif-labels -Wexpansion-to-defined -Wimplicit-fallthrough=2 -Wno-missing-include-dirs -Wno-shift-negative-value -Wno-psabi \
    -fstack-protector-strong -DNDEBUG -fPIE -MD -MQ \
    libqemuutil.a.p/util_qemu-thread-posix.c.o -MF libqemuutil.a.p/util_qemu-thread-posix.c.o.d -o libqemuutil.a.p/util_qemu-thread-posix.c.o \
    -c ~/test_syscall.c

```

**ok**

## 找到问题

```sh
> ls linux-headers/
COPYING    asm-arm/      asm-loongarch/  asm-riscv/  linux/
LICENSES/  asm-arm64/    asm-mips/       asm-s390/
README     asm-generic/  asm-powerpc/    asm-x86/
> cat linux-headers/README
Automatically imported Linux kernel headers.
Only use scripts/update-linux-headers.sh to update!
```

使用的 头文件部匹配，需要重新生成头文件，`scripts/update-linux-headers.sh ~/linux-6.7`解决问题

