# 调试总结

## Cpython && Openssl test

#### 现象：

开启AVX指令集后，执行cpython测试错误项增加。

```
liuchaoyi@10.40.65.155
翻译器 lat-zl
cpuid有CPUID_7_0_EBX_ADX之后cpython以下测试同不过

test_asyncio
test_ftplib
test_imaplib
test_ssl
test_urllib2_localnet
test_logging
test_poplib
test_nntplib

跑的时候要注掉CPUID_EXT_PCLMULQDQ

export LATX_SOFTFPU=1 && export LATX_AVXON=1

执行测试位置/opt/latx-tests/cpython/build

跑单项的命令

./python -m test.子项
./python -m test -v test.子项 -m 子项的子项

起docker的脚本在test-cpython里
```

#### 思路：

找一个开始调，复现错误，当时选择了认为可能更为熟悉的`test_ssl`

#### 调试：

##### 复现错误：

按照同事的记录，在docker中运行相应测试，复现了错误。

```bash
./python -m test.test_ssl &> vim
grep -rni ERROR vim
```

##### 第一次精简尝试：

选择一个错误子项`test_ciphers`，此时问题已经足够小。

````bash
root@loongson-pc:/opt/latx-tests/cpython/build# ./python -m test -v test.test_ssl -m test_ciphers

test_ciphers (test.test_ssl.SimpleBackgroundTests) ...  server:  new connection from ('127.0.0.1', 33910)
 server:  bad connection attempt from ('127.0.0.1', 33910):
Traceback (most recent call last):
   File "/opt/latx-tests/cpython/Lib/test/test_ssl.py", line 2358, in wrap_conn
    self.sslconn = self.server.context.wrap_socket(
   File "/opt/latx-tests/cpython/Lib/ssl.py", line 500, in wrap_socket
    return self.sslsocket_class._create(
   File "/opt/latx-tests/cpython/Lib/ssl.py", line 1040, in _create
    self.do_handshake()
   File "/opt/latx-tests/cpython/Lib/ssl.py", line 1309, in do_handshake
    self._sslobj.do_handshake()
 ssl.SSLError: [SSL: TLSV1_ALERT_DECRYPT_ERROR] tlsv1 alert decrypt error (_ssl.c:1131)
ERROR

======================================================================
ERROR: test_ciphers (test.test_ssl.SimpleBackgroundTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/opt/latx-tests/cpython/Lib/test/test_ssl.py", line 2162, in test_ciphers
    s.connect(self.server_addr)
  File "/opt/latx-tests/cpython/Lib/ssl.py", line 1342, in connect
    self._real_connect(addr, False)
  File "/opt/latx-tests/cpython/Lib/ssl.py", line 1333, in _real_connect
    self.do_handshake()
  File "/opt/latx-tests/cpython/Lib/ssl.py", line 1309, in do_handshake
    self._sslobj.do_handshake()
ssl.SSLError: [SSL: BAD_SIGNATURE] bad signature (_ssl.c:1131)

----------------------------------------------------------------------

Ran 2 tests in 0.338s

FAILED (errors=1)
test test.test_ssl failed
test.test_ssl failed

== Tests result: FAILURE ==

1 test failed:
    test.test_ssl
````

此时环境仍在docker中，更换翻译器比较麻烦，因此想办法把测试抽离至LA真机环境。

此时会遇到点小问题，不过不展开了，我在`test_ssl.py`中强制修改了`self`中的一个存储路径的变量，使之能在外部运行。



##### 第一次调试尝试：

正确环境和错误环境同时使用 `--latx-dump  11111`选项，输出测试中全部翻译的指令。再使用正则表达式匹配出所有翻译的IR1指令，对比不同。

```
错误的log中，多翻译了4条指令
adcx,adox,mulx,xgetbv
```

此时 注条指令查看翻译函数，未发现异常。除了mulx指令，刚开始看时同事说确认过mulx在dest和src相同时的正确性。

所以怀疑是在某个pattern下出错。

同时开启lative随机指令测试，运行这些指令的随机，运行一晚上未出错。



##### 第二次精简尝试：

尝试找到上面错误信息中的底层函数，`do_handshake`，找了半天似乎是`_ssl.cpython-38-x86_64-linux-gnu.so`中的函数。

python -> C  比较难重编。

换个思路，既然是调用openssl的接口，那么，大概率Openssl的测试也无法通过。

立刻上网下了一份`openssl-3.2.0`

```
mkdir build && cd build
../config
make test -j20
```

以上在docker环境中完成，省略了对齐环境的问题。

果然发生了错误，找到第一个错误的测试开始调试。

```bash
make test/ec_internal_test
```

此时修改 Makefile  给ec_internal_test 编译时增加了--static -g等常用参数。

那么，问题聚焦在一个C语言编写的小测试了` ./ec_internal_test`，剩下只是时间问题。 



##### 第二次调试尝试：

此测试有100多个测试子项，在第162个测试子项出错，此时所有代码均翻译完成，较难调试。

重复第一次尝试的内容，依旧是那几条指令。



##### 第三次精简测试：

稍微看了下Openssl的源码，其中数据使用的数据结构为`BIGNUM`，应该是某次`BIGNUM`的计算错误。

一路给测试加log，最后定位至函数`bn_mul_mont`，此函数有多个实现，最后发现都对不上。

```bash
grep -rn  bn_mul_mont ..
vi  ./crypto/bn/x86_64-mont.s
```

在一个汇编文件里，几百行汇编。

强行构建了错误的`BIGNUM`，精简为第一次运行就会出错的代码。(`BIGNUM`是真的BIG）

```
BN_BITS2:64
top:0
dmax:16
neg:0
flags:0
0
d[0]:c4ce96095161d9d3
d[1]:683e4d64272c02a4
d[2]:34ab04146df55e8f
d[3]:8550539514c01fc8
d[4]:2433d76f905c8737
d[5]:b2b6ea37f36d3cf7
d[6]:871cb5ca006d4573
d[7]:5a2ba14c0994e981
BN_BITS2:64
top:8
dmax:8
neg:0
flags:1
5A2BA14C0994E981871CB5CA006D4573B2B6EA37F36D3CF72433D76F905C87378550539514C01FC834AB04146DF55E8F683E4D64272C02A4C4CE96095161D9D3

     BN_ULONG *d = ret->d;
     d[0] = 0xc4ce96095161d9d3;
     d[1] = 0x683e4d64272c02a4;
     d[2] = 0x34ab04146df55e8f;
     d[3] = 0x8550539514c01fc8;
     d[4] = 0x2433d76f905c8737;
     d[5] = 0xb2b6ea37f36d3cf7;
     d[6] = 0x871cb5ca006d4573;
     d[7] = 0x5a2ba14c0994e981;
     ret->top = 8;
     ret->dmax = 8;
     ret->neg = 0;
     ret->flags = 1;
     
d[0]: 28aa6056583a48f3
d[1]: 2881ff2f2d82c685
d[2]: aecda12ae6a380e6
d[3]: 7d4d9b009bc66842
d[4]: d6639cca70330871
d[5]: cb308db3b3c9d20e
d[6]: 3fd4e6ae33c9fc07
d[7]: aadd9db8dbe9c48b
BN_BITS2: 64
top: 8
dmax: 8
neg: 0
flags: 0
AADD9DB8DBE9C48B3FD4E6AE33C9FC07CB308DB3B3C9D20ED6639CCA703308717D4D9B009BC66842AECDA12AE6A380E62881FF2F2D82C68528AA6056583A48F3
mont-> n0[0] :kkk  n0[1] :0
num : 8

4373f0

r8 right 
0x2597a614513d63e9

rbx wrong 

```

##### 第三次调试尝试：

打开`latx-dump 11111`, `objdump`找到bn_mul_mont地址，从此tb开始进行追踪。

跟踪了10个tb左右，找了数据错误的开始。

定位到了mulx的错误。



#### 总结：

应该听lxy的多`瞪眼`指令翻译。

应该使用`--singalstep`和qemu  `-d cpu`直接比对 





bn_mul_mont



## TB_unlink

#### 现象：

打开tu，收到信号后无法解链，导致卡死。

#### 测试代码：

```c
#include <unistd.h>
#include <signal.h>
#include <stdlib.h>

static char var1 = 0L;
static char *var2 = &var1;

void do_exit (int i)
{
  exit (0);
}

int main(void)
{
  struct sigaction s;
  sigemptyset (&s.sa_mask);
  s.sa_handler = do_exit;
  s.sa_flags = 0;
  sigaction (SIGALRM, &s, NULL);
  alarm (1);
  /* The following loop is infinite, the division by zero should not
     be hoisted out of it.  */
  for (; (var1 == 0 ? 0 : (100 / var1)) == *var2; );
  return 0;
}
```

alarm的输入为秒数，有点慢，因此改成ualarm，0.3秒后触发信号。

#### 调试:

##### 复现问题:

对齐代码，主分支打开tu，直接编译，未复现错误。

猜测为没正确打开tu，查看相关代码，发现虽然打开tu，但依旧使用tb_gen_code，强行改为tu_gen_code，复现错误。



##### 修改相关代码：

大体知道代码在signal.c中，但未实际看过代码。

未找到反查逻辑的实现，上楼问杨兆鑫，找到相关代码，开始抽取相关逻辑。



##### 调试代码逻辑：

tu时大概率正确，大概10次一次错误。

在信号处理时增加打印，输出当前翻译器pid。

同时改为tb_gen_code 测试是否会发生错误，tb同样会错误，因此大概率是反查数据结构没有填写正确。

关tu，利用反查进行unlink



实际发现错误为：反查逻辑错误

```c
    searched_pc -= GETPC_ADJ;
    if (searched_pc < host_pc) return 0;
```

这里转换时，应该先-GETPC_ADJ，否则第一条指令会直接减飞。



再次测试，发现tb_gen_code情况下全部正确，tu下有时依旧无法正确反查。

循环执行，触发错误

```bash
time for i in {1..500}; do ./latx-x86_64 ~/a.out; done
```

在错误时，让翻译器输出线程号，然后死循环。

```bash
loongson@loongson-pc:~/lyz/lat/build64-dbg$ for i in {1..500}; do ./latx-x86_64 ~/a.out; done       
ww
hh
ww                                                 
lhl-debug tb_exit_to_qemu current_tb NULL pid 30965
```

```
sudo gdb
attach 30965
```

通过当前现场，得知tcg_tb_lookup未能正确找到对应的tb，此指令为nop，同时此pc前的几条指令能够正确索引tb

判断为tb->tc.size更新错误，阅读相关代码得知：

```c
void translate_tu(uint32 tb_num_in_tu, TranslationBlock **tb_list)
{
            tu_out_nop_fill((int32_t *)old_buf_ptr, nop_offset / 4);
            gen_code_size += nop_offset / 4;
}
```

此处gen_code_size不应该为nop_offset / 4 而应该为nop_offset。

修改后按tu翻译也正确。

```bash
time for i in {1..500}; do ./latx-x86_64 ~/a.out; done
```

循环500次  均能正确退出。



