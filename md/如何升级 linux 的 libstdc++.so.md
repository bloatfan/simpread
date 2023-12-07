> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/498529973)

以 centos7.4 为例，说明 libstdc++.so.6.0.19 升级到 libstdc++.so.6.0.24

ubuntu 也可以通过这个进行 libstdc++.so.6 进行升级

libstdc++ 的代码是在 gcc 的代码中，需要下载 gcc 代码，对其进行编译安装，设置软连接

**1 查看当前已经安装的 glibc 的版本**

目前安装的是 libstdc++.so.6.0.19

```
查看centos7上已经安装的libstdc++.so的版本
locate libstdc++.so.6
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib/libstdc++.so.6.0.19-gdb.py
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib/libstdc++.so.6.0.19-gdb.pyc
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib/libstdc++.so.6.0.19-gdb.pyo
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.py
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyc
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyo
/usr/lib64/libstdc++.so.6
/usr/lib64/libstdc++.so.6.0.19
/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.py
/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyc
/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyo

# 查看libstdc++.so中的GLIBC的版本支持
strings /usr/lib64/libstdc++.so.6 |grep GLIBC
GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19  // 支持的GLIBCXX最高版本：3.4.19
GLIBC_2.3
GLIBC_2.2.5
GLIBC_2.14
GLIBC_2.4
GLIBC_2.3.2
GLIBCXX_DEBUG_MESSAGE_LENGTH
```

**2. 下载 gcc 的代码 & 解压缩**

```
# 下载gcc7.3.0对应的glibc源代码
wget http://ftp.gnu.org/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.gz
--2022-04-14 09:40:08--  http://ftp.gnu.org/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.gz
Resolving ftp.gnu.org (ftp.gnu.org)... 209.51.188.20, 2001:470:142:3::b
Connecting to ftp.gnu.org (ftp.gnu.org)|209.51.188.20|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 112201917 (107M) [application/x-gzip]
Saving to: 'gcc-7.3.0.tar.gz'                   1% [=>                                                                                                                                                    ] 1,974,557   7.51KB/s  eta 1h 40m
...

$ls
gcc-7.3.0.tar.gz
$tar -xvf gcc-7.3.0.tar.gz
```

**3. 编译 gcc/libstdc++**

**3.1 准备依赖： 需要在 gcc 代码的根目录执行**

这一步必须执行。

```
cd gcc-7.3.0
./contrib/download_prerequisites
2022-04-14 11:47:39 URL: ftp://gcc.gnu.org/pub/gcc/infrastructure/gmp-6.1.0.tar.bz2 [2383840] -> "./gmp-6.1.0.tar.bz2" [1]
2022-04-14 11:49:20 URL: ftp://gcc.gnu.org/pub/gcc/infrastructure/mpfr-3.1.4.tar.bz2 [1279284] -> "./mpfr-3.1.4.tar.bz2" [1]
2022-04-14 11:49:56 URL: ftp://gcc.gnu.org/pub/gcc/infrastructure/mpc-1.0.3.tar.gz [669925] -> "./mpc-1.0.3.tar.gz" [1]
2022-04-14 11:52:05 URL: ftp://gcc.gnu.org/pub/gcc/infrastructure/isl-0.16.1.tar.bz2 [1626446] -> "./isl-0.16.1.tar.bz2" [1]
gmp-6.1.0.tar.bz2: OK
mpfr-3.1.4.tar.bz2: OK
mpc-1.0.3.tar.gz: OK
isl-0.16.1.tar.bz2: OK
All prerequisites downloaded successfully.
```

**3.2 配置生成 Makefile**

mkdir build;cd build;

../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib

--disable-multilib: 用于关闭多个多个 arch 的库，比如只需要 64 位的，不需要 32 位的库，那就添加这个配置

```
# 配置生成makefile
$ mkdir build
$ cd build
[root@xxxx build]# ../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking target system type... x86_64-pc-linux-gnu
checking for a BSD-compatible install... /usr/bin/install -c
checking whether ln works... yes
checking whether ln -s works... yes
checking for a sed that does not truncate output... /usr/bin/sed
checking for gawk... gawk
checking for libatomic support... yes
checking for libcilkrts support... yes
checking for libitm support... yes
checking for libsanitizer support... yes
checking for libvtv support... yes
checking for libmpx support... yes
checking for libhsail-rt support... yes
checking for gcc... gcc
checking for C compiler default output file name... a.out
checking whether the C compiler works... yes
checking whether we are cross compiling... no
checking for suffix of executables...
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking for g++... g++
checking whether we are using the GNU C++ compiler... yes
checking whether g++ accepts -g... yes
checking whether g++ accepts -static-libstdc++ -static-libgcc... yes
checking for gnatbind... no
checking for gnatmake... no
checking whether compiler driver understands Ada... no
checking how to compare bootstrapped objects... cmp --ignore-initial=16 $$f1 $$f2
checking for objdir... .libs
configure: WARNING: using in-tree isl, disabling version check
*** This configuration is not supported in the following subdirectories:
     gnattools gotools target-libada target-libhsail-rt target-libgfortran target-libbacktrace target-libgo target-libffi target-libobjc target-liboffloadmic
    (Any other directories should still work fine.)
checking for default BUILD_CONFIG... bootstrap-debug
checking for --enable-vtable-verify... no
checking for bison... no
checking for byacc... no
checking for yacc... no
checking for bison... no
checking for gm4... no
checking for gnum4... no
checking for m4... m4
checking for flex... no
checking for lex... no
checking for flex... no
checking for makeinfo... no
/my_data/code/gcc_7.3/gcc-7.3.0/missing: line 81: makeinfo: command not found
checking for expect... no
checking for runtest... no
checking for ar... ar
checking for as... as
checking for dlltool... no
checking for ld... (cached) /opt/rh/devtoolset-7/root/usr/libexec/gcc/x86_64-redhat-linux/7/ld
checking for lipo... no
checking for nm... nm
checking for ranlib... ranlib
checking for strip... strip
checking for windres... no
checking for windmc... no
checking for objcopy... objcopy
checking for objdump... objdump
checking for readelf... readelf
checking for cc... cc
checking for c++... c++
checking for gcc... gcc
checking for gfortran... gfortran
checking for gccgo... no
checking for ar... no
checking for ar... ar
checking for as... no
checking for as... as
checking for dlltool... no
checking for dlltool... no
checking for ld... no
checking for ld... ld
checking for lipo... no
checking for lipo... no
checking for nm... no
checking for nm... nm
checking for objcopy... no
checking for objcopy... objcopy
checking for objdump... no
checking for objdump... objdump
checking for ranlib... no
checking for ranlib... ranlib
checking for readelf... no
checking for readelf... readelf
checking for strip... no
checking for strip... strip
checking for windres... no
checking for windres... no
checking for windmc... no
checking for windmc... no
checking where to find the target ar... host tool
checking where to find the target as... host tool
checking where to find the target cc... just compiled
checking where to find the target c++... just compiled
checking where to find the target c++ for libstdc++... just compiled
checking where to find the target dlltool... host tool
checking where to find the target gcc... just compiled
checking where to find the target gfortran... host tool
checking where to find the target gccgo... host tool
checking where to find the target ld... host tool
checking where to find the target lipo... host tool
checking where to find the target nm... host tool
checking where to find the target objcopy... host tool
checking where to find the target objdump... host tool
checking where to find the target ranlib... host tool
checking where to find the target readelf... host tool
checking where to find the target strip... host tool
checking where to find the target windres... host tool
checking where to find the target windmc... host tool
checking whether to enable maintainer-specific portions of Makefiles... no
configure: creating ./config.status
config.status: creating Makefile
```

**3.3 执行编译**

make -j n 指定多线程编译，默认是单线程的 (比如： make -j 16)

```
#执行编译 （至少1.5 小时，忘记加time统计了）
[root@xxx build]# ls
Makefile  config.log  config.status  serdep.tmp
[root@xxx build]# make   
[ -f stage_final ] || echo stage3 > stage_final
make[1]: Entering directory '/my_data/code/gcc_7.3/gcc-7.3.0/build'
make[2]: Entering directory '/my_data/code/gcc_7.3/gcc-7.3.0/build'
make[2]: Leaving directory '/my_data/code/gcc_7.3/gcc-7.3.0/build'
make[2]: Entering directory '/my_data/code/gcc_7.3/gcc-7.3.0/build'
Configuring stage 1 in ./intl
configure: creating cache ./config.cache
checking whether make sets $(MAKE)... yes
checking for a BSD-compatible install... /usr/bin/install -c
checking whether NLS is requested... yes
checking for msgfmt... /usr/bin/msgfmt
checking for gmsgfmt... /usr/bin/msgfmt
checking for xgettext... /usr/bin/xgettext
checking for msgmerge... /usr/bin/msgmerge
checking for x86_64-pc-linux-gnu-gcc... gcc
checking for C compiler default output file name... a.out
checking whether the C compiler works... yes
checking whether we are cross compiling... no
checking for suffix of executables...
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking for x86_64-pc-linux-gnu-ranlib... ranlib
checking for library containing strerror... none required
checking how to run the C preprocessor... gcc -E
checking for grep that handles long lines and -e... /usr/bin/grep
......
config.status: executing libtool commands
configure: summary of build options:

  Version:           GNU MP 6.1.0
  Host type:         none-pc-linux-gnu
  ABI:               standard
  Install prefix:    /usr/local
  Compiler:          gcc
  Static libraries:  yes
  Shared libraries:  no

make[3]: Entering directory '/my_data/code/gcc_7.3/gcc-7.3.0/build/gmp'
gcc `test -f 'gen-fac.c' || echo '../../gmp/'`gen-fac.c -o gen-fac
./gen-fac 64 0 >fac_table.h || (rm -f fac_table.h; exit 1)
gcc `test -f 'gen-fib.c' || echo '../../gmp/'`gen-fib.c -o gen-fib
./gen-fib header 64 0 >fib_table.h || (rm -f fib_table.h; exit 1)
./gen-fib table 64 0 >mpn/fib_table.c || (rm -f mpn/fib_table.c; exit 1)
gcc `test -f 'gen-bases.c' || echo '../../gmp/'`gen-bases.c -o gen-bases -lm
./gen-bases header 64 0 >mp_bases.h || (rm -f mp_bases.h; exit 1)
./gen-bases table 64 0 >mpn/mp_bases.c || (rm -f mpn/mp_bases.c; exit 1)
gcc `test -f 'gen-trialdivtab.c' || echo '../../gmp/'`gen-trialdivtab.c -o gen-trialdivtab -lm
./gen-trialdivtab 64 8000 >trialdivtab.h || (rm -f trialdivtab.h; exit 1)
gcc `test -f 'gen-jacobitab.c' || echo '../../gmp/'`gen-jacobitab.c -o gen-jacobitab
./gen-jacobitab >mpn/jacobitab.h || (rm -f mpn/jacobitab.h; exit 1)
gcc `test -f 'gen-psqr.c' || echo '../../gmp/'`gen-psqr.c -o gen-psqr -lm
./gen-psqr 64 0 >mpn/perfsqr.h || (rm -f mpn/perfsqr.h; exit 1)
make  all-recursive
make[4]: Entering directory '/my_data/code/gcc_7.3/gcc-7.3.0/build/gmp'
Making all in tests
make[5]: Entering directory '/my_data/code/gcc_7.3/gcc-7.3.0/build/gmp/tests'
Making all in .
/bin/sh ../libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I. -I../../../gmp/mpn -I..  -D__GMP_WITHIN_GMP -I../../../gmp -DOPERATION_`echo toom54_mul | sed 's/_$//'`  -DNO_ASM -g -c -o toom54_mul.lo toom54_mul.c
libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I../../../gmp/mpn -I.. -D__GMP_WITHIN_GMP -I../../../gmp -DOPERATION_sec_div_r -DNO_ASM -g -c sec_div_r.c -o sec_div_r.o
/bin/sh ../libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I. -I../../../gmp/mpn -I..  -D__GMP_WITHIN_GMP -I../../../gmp -DOPERATION_`echo sec_pi1_div_qr | sed 's/_$//'`  -DNO_ASM -g -c -o sec_pi1_div_qr.lo sec_pi1_div_qr.c
libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I../../../gmp/mpn -I.. -D__GMP_WITHIN_GMP -I../../../gmp -DOPERATION_sec_pi1_div_qr -DNO_ASM -g -c sec_pi1_div_qr.c -o sec_pi1_div_qr.o

...
...
// log 实在太长了，中间的省略了
libtool: compile:  /my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/xgcc -B/my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/ -B/usr/local/x86_64-pc-linux-gnu/bin/ -B/usr/local/x86_64-pc-linux-gnu/lib/ -isystem /usr/local/x86_64-pc-linux-gnu/include -isystem /usr/local/x86_64-pc-linux-gnu/sys-include -DHAVE_CONFIG_H -I../../../libatomic/config/x86 -I../../../libatomic/config/posix -I../../../libatomic -I. -Wall -Werror -pthread -g -O2 -MT tas_16_1_.lo -MD -MP -MF .deps/tas_16_1_.lo.Ppo -DN=16 -DIFUNC_ALT=1 -mcx16 -c ../../../libatomic/tas_n.c  -fPIC -DPIC -o .libs/tas_16_1_.o
libtool: compile:  /my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/xgcc -B/my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/ -B/usr/local/x86_64-pc-linux-gnu/bin/ -B/usr/local/x86_64-pc-linux-gnu/lib/ -isystem /usr/local/x86_64-pc-linux-gnu/include -isystem /usr/local/x86_64-pc-linux-gnu/sys-include -DHAVE_CONFIG_H -I../../../libatomic/config/x86 -I../../../libatomic/config/posix -I../../../libatomic -I. -Wall -Werror -pthread -g -O2 -MT tas_16_1_.lo -MD -MP -MF .deps/tas_16_1_.lo.Ppo -DN=16 -DIFUNC_ALT=1 -mcx16 -c ../../../libatomic/tas_n.c -o tas_16_1_.o >/dev/null 2>&1
/bin/sh ./libtool --tag=CC   --mode=link /my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/xgcc -B/my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/ -B/usr/local/x86_64-pc-linux-gnu/bin/ -B/usr/local/x86_64-pc-linux-gnu/lib/ -isystem /usr/local/x86_64-pc-linux-gnu/include -isystem /usr/local/x86_64-pc-linux-gnu/sys-include    -Wall -Werror  -pthread -g -O2   -Wl,-O1   -o libatomic_convenience.la  gload.lo gstore.lo gcas.lo gexch.lo glfree.lo lock.lo init.lo fenv.lo fence.lo flag.lo load_1_.lo store_1_.lo cas_1_.lo exch_1_.lo fadd_1_.lo fsub_1_.lo fand_1_.lo fior_1_.lo fxor_1_.lo fnand_1_.lo tas_1_.lo load_2_.lo store_2_.lo cas_2_.lo exch_2_.lo fadd_2_.lo fsub_2_.lo fand_2_.lo fior_2_.lo fxor_2_.lo fnand_2_.lo tas_2_.lo load_4_.lo store_4_.lo cas_4_.lo exch_4_.lo fadd_4_.lo fsub_4_.lo fand_4_.lo fior_4_.lo fxor_4_.lo fnand_4_.lo tas_4_.lo load_8_.lo store_8_.lo cas_8_.lo exch_8_.lo fadd_8_.lo fsub_8_.lo fand_8_.lo fior_8_.lo fxor_8_.lo fnand_8_.lo tas_8_.lo load_16_.lo store_16_.lo cas_16_.lo exch_16_.lo fadd_16_.lo fsub_16_.lo fand_16_.lo fior_16_.lo fxor_16_.lo fnand_16_.lo tas_16_.lo   load_16_1_.lo store_16_1_.lo cas_16_1_.lo exch_16_1_.lo fadd_16_1_.lo fsub_16_1_.lo fand_16_1_.lo fior_16_1_.lo fxor_16_1_.lo fnand_16_1_.lo tas_16_1_.lo
libtool: link: ar rc .libs/libatomic_convenience.a .libs/gload.o .libs/gstore.o .libs/gcas.o .libs/gexch.o .libs/glfree.o .libs/lock.o .libs/init.o .libs/fenv.o .libs/fence.o .libs/flag.o .libs/load_1_.o .libs/store_1_.o .libs/cas_1_.o .libs/exch_1_.o .libs/fadd_1_.o .libs/fsub_1_.o .libs/fand_1_.o .libs/fior_1_.o .libs/fxor_1_.o .libs/fnand_1_.o .libs/tas_1_.o .libs/load_2_.o .libs/store_2_.o .libs/cas_2_.o .libs/exch_2_.o .libs/fadd_2_.o .libs/fsub_2_.o .libs/fand_2_.o .libs/fior_2_.o .libs/fxor_2_.o .libs/fnand_2_.o .libs/tas_2_.o .libs/load_4_.o .libs/store_4_.o .libs/cas_4_.o .libs/exch_4_.o .libs/fadd_4_.o .libs/fsub_4_.o .libs/fand_4_.o .libs/fior_4_.o .libs/fxor_4_.o .libs/fnand_4_.o .libs/tas_4_.o .libs/load_8_.o .libs/store_8_.o .libs/cas_8_.o .libs/exch_8_.o .libs/fadd_8_.o .libs/fsub_8_.o .libs/fand_8_.o .libs/fior_8_.o .libs/fxor_8_.o .libs/fnand_8_.o .libs/tas_8_.o .libs/load_16_.o .libs/store_16_.o .libs/cas_16_.o .libs/exch_16_.o .libs/fadd_16_.o .libs/fsub_16_.o .libs/fand_16_.o .libs/fior_16_.o .libs/fxor_16_.o .libs/fnand_16_.o .libs/tas_16_.o .libs/load_16_1_.o .libs/store_16_1_.o .libs/cas_16_1_.o .libs/exch_16_1_.o .libs/fadd_16_1_.o .libs/fsub_16_1_.o .libs/fand_16_1_.o .libs/fior_16_1_.o .libs/fxor_16_1_.o .libs/fnand_16_1_.o .libs/tas_16_1_.o
libtool: link: ranlib .libs/libatomic_convenience.a
libtool: link: ( cd ".libs" && rm -f "libatomic_convenience.la" && ln -s "../libatomic_convenience.la" "libatomic_convenience.la" )
/bin/sh ./libtool --tag=CC   --mode=link /my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/xgcc -B/my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/ -B/usr/local/x86_64-pc-linux-gnu/bin/ -B/usr/local/x86_64-pc-linux-gnu/lib/ -isystem /usr/local/x86_64-pc-linux-gnu/include -isystem /usr/local/x86_64-pc-linux-gnu/sys-include    -Wall -Werror  -pthread -g -O2 -version-info 3:0:2 -Wl,--version-script,../../../libatomic/libatomic.map   -o libatomic.la -rpath /usr/local/lib/../lib64 gload.lo gstore.lo gcas.lo gexch.lo glfree.lo lock.lo init.lo fenv.lo fence.lo flag.lo load_1_.lo store_1_.lo cas_1_.lo exch_1_.lo fadd_1_.lo fsub_1_.lo fand_1_.lo fior_1_.lo fxor_1_.lo fnand_1_.lo tas_1_.lo load_2_.lo store_2_.lo cas_2_.lo exch_2_.lo fadd_2_.lo fsub_2_.lo fand_2_.lo fior_2_.lo fxor_2_.lo fnand_2_.lo tas_2_.lo load_4_.lo store_4_.lo cas_4_.lo exch_4_.lo fadd_4_.lo fsub_4_.lo fand_4_.lo fior_4_.lo fxor_4_.lo fnand_4_.lo tas_4_.lo load_8_.lo store_8_.lo cas_8_.lo exch_8_.lo fadd_8_.lo fsub_8_.lo fand_8_.lo fior_8_.lo fxor_8_.lo fnand_8_.lo tas_8_.lo load_16_.lo store_16_.lo cas_16_.lo exch_16_.lo fadd_16_.lo fsub_16_.lo fand_16_.lo fior_16_.lo fxor_16_.lo fnand_16_.lo tas_16_.lo   load_16_1_.lo store_16_1_.lo cas_16_1_.lo exch_16_1_.lo fadd_16_1_.lo fsub_16_1_.lo fand_16_1_.lo fior_16_1_.lo fxor_16_1_.lo fnand_16_1_.lo tas_16_1_.lo
libtool: link: /my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/xgcc -B/my_data/code/gcc_7.3/gcc-7.3.0/build/./gcc/ -B/usr/local/x86_64-pc-linux-gnu/bin/ -B/usr/local/x86_64-pc-linux-gnu/lib/ -isystem /usr/local/x86_64-pc-linux-gnu/include -isystem /usr/local/x86_64-pc-linux-gnu/sys-include    -shared  -fPIC -DPIC  .libs/gload.o .libs/gstore.o .libs/gcas.o .libs/gexch.o .libs/glfree.o .libs/lock.o .libs/init.o .libs/fenv.o .libs/fence.o .libs/flag.o .libs/load_1_.o .libs/store_1_.o .libs/cas_1_.o .libs/exch_1_.o .libs/fadd_1_.o .libs/fsub_1_.o .libs/fand_1_.o .libs/fior_1_.o .libs/fxor_1_.o .libs/fnand_1_.o .libs/tas_1_.o .libs/load_2_.o .libs/store_2_.o .libs/cas_2_.o .libs/exch_2_.o .libs/fadd_2_.o .libs/fsub_2_.o .libs/fand_2_.o .libs/fior_2_.o .libs/fxor_2_.o .libs/fnand_2_.o .libs/tas_2_.o .libs/load_4_.o .libs/store_4_.o .libs/cas_4_.o .libs/exch_4_.o .libs/fadd_4_.o .libs/fsub_4_.o .libs/fand_4_.o .libs/fior_4_.o .libs/fxor_4_.o .libs/fnand_4_.o .libs/tas_4_.o .libs/load_8_.o .libs/store_8_.o .libs/cas_8_.o .libs/exch_8_.o .libs/fadd_8_.o .libs/fsub_8_.o .libs/fand_8_.o .libs/fior_8_.o .libs/fxor_8_.o .libs/fnand_8_.o .libs/tas_8_.o .libs/load_16_.o .libs/store_16_.o .libs/cas_16_.o .libs/exch_16_.o .libs/fadd_16_.o .libs/fsub_16_.o .libs/fand_16_.o .libs/fior_16_.o .libs/fxor_16_.o .libs/fnand_16_.o .libs/tas_16_.o .libs/load_16_1_.o .libs/store_16_1_.o .libs/cas_16_1_.o .libs/exch_16_1_.o .libs/fadd_16_1_.o .libs/fsub_16_1_.o .libs/fand_16_1_.o .libs/fior_16_1_.o .libs/fxor_16_1_.o .libs/fnand_16_1_.o .libs/tas_16_1_.o    -pthread -Wl,--version-script -Wl,../../../libatomic/libatomic.map   -pthread -Wl,-soname -Wl,libatomic.so.1 -o .libs/libatomic.so.1.2.0
libtool: link: (cd ".libs" && rm -f "libatomic.so.1" && ln -s "libatomic.so.1.2.0" "libatomic.so.1")
libtool: link: (cd ".libs" && rm -f "libatomic.so" && ln -s "libatomic.so.1.2.0" "libatomic.so")
libtool: link: ar rc .libs/libatomic.a  gload.o gstore.o gcas.o gexch.o glfree.o lock.o init.o fenv.o fence.o flag.o load_1_.o store_1_.o cas_1_.o exch_1_.o fadd_1_.o fsub_1_.o fand_1_.o fior_1_.o fxor_1_.o fnand_1_.o tas_1_.o load_2_.o store_2_.o cas_2_.o exch_2_.o fadd_2_.o fsub_2_.o fand_2_.o fior_2_.o fxor_2_.o fnand_2_.o tas_2_.o load_4_.o store_4_.o cas_4_.o exch_4_.o fadd_4_.o fsub_4_.o fand_4_.o fior_4_.o fxor_4_.o fnand_4_.o tas_4_.o load_8_.o store_8_.o cas_8_.o exch_8_.o fadd_8_.o fsub_8_.o fand_8_.o fior_8_.o fxor_8_.o fnand_8_.o tas_8_.o load_16_.o store_16_.o cas_16_.o exch_16_.o fadd_16_.o fsub_16_.o fand_16_.o fior_16_.o fxor_16_.o fnand_16_.o tas_16_.o load_16_1_.o store_16_1_.o cas_16_1_.o exch_16_1_.o fadd_16_1_.o fsub_16_1_.o fand_16_1_.o fior_16_1_.o fxor_16_1_.o fnand_16_1_.o tas_16_1_.o
libtool: link: ranlib .libs/libatomic.a
libtool: link: ( cd ".libs" && rm -f "libatomic.la" && ln -s "../libatomic.la" "libatomic.la" )
true  DO=all multi-do # make
make[4]: Leaving directory '/my_data/code/gcc_7.3/gcc-7.3.0/build/x86_64-pc-linux-gnu/libatomic'
make[3]: Leaving directory '/my_data/code/gcc_7.3/gcc-7.3.0/build/x86_64-pc-linux-gnu/libatomic'
make[2]: Leaving directory '/my_data/code/gcc_7.3/gcc-7.3.0/build/x86_64-pc-linux-gnu/libatomic'
make[1]: Leaving directory '/my_data/code/gcc_7.3/gcc-7.3.0/build'
```

**3.4 安装 & 更新 soft link**

```
# 安装
make install 
    # 安装完后，编译出来的libstdc++.so.6.0.24  安装到 /usr/local/lib64/下面
    # 可以 find / -name libstdc++.so*查找

# 把安装后的libstdc++.so.6.0.24 拷贝到/usr/lib64
cp /usr/local/lib64/libstdc++.so.6.0.24  /usr/lib64/

$ locate libstdc++.so.6
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib/libstdc++.so.6.0.19-gdb.py
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib/libstdc++.so.6.0.19-gdb.pyc
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib/libstdc++.so.6.0.19-gdb.pyo
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.py
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyc
/opt/rh/devtoolset-7/root/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyo
/usr/lib64/libstdc++.so.6
/usr/lib64/libstdc++.so.6.0.19
/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.py
/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyc
/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyo

# 创建软连接
ll libstdc++*
lrwxrwxrwx 1 root root       19 Mar 10 18:43 libstdc++.so.6 -> libstdc++.so.6.0.19
-rwxr-xr-x 1 root root   995840 Sep 30  2020 libstdc++.so.6.0.19
-rwxr-xr-x 1 root root 11521888 Apr 14 14:28 libstdc++.so.6.0.24
[root@xxxx lib64]# rm libstdc++.so.6
rm: remove symbolic link 'libstdc++.so.6'? y
[root@xxx lib64]# ln -sf libstdc++.so.6.0.24 libstdc++.so.6
[root@xxx lib64]# ll libstdc++.so.6*
lrwxrwxrwx 1 root root       19 Apr 14 16:31 libstdc++.so.6 -> libstdc++.so.6.0.24
-rwxr-xr-x 1 root root   995840 Sep 30  2020 libstdc++.so.6.0.19
-rwxr-xr-x 1 root root 11521888 Apr 14 14:28 libstdc++.so.6.0.24
```

**3.5 确认 glibcxx 的版本**

```
[root@xxx lib64]# strings  libstdc++.so.6 |grep GLIBC
GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_3.4.20   // 新增glibcxx支持的版本
GLIBCXX_3.4.21
GLIBCXX_3.4.22
GLIBCXX_3.4.23
GLIBCXX_3.4.24
GLIBC_2.2.5
GLIBC_2.3
GLIBC_2.14
GLIBC_2.16      // 新增glibc支持的版本
GLIBC_2.17
GLIBC_2.3.2
GLIBCXX_DEBUG_MESSAGE_LENGTH

。。。。
```

升级完成。