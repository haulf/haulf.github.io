---
layout: post
title:  "在Mac下编译和安装Lua"
date:   2018-04-05 15:54:15
categories: Lua语言编程
tags: Lua Mac
excerpt: 家里用的是Mac一体机，与公司用的Windows和Linux不同。打算在Mac里下载Lua最新版本源代码进行编译和安装。
---

* content
{:toc}

## 在Mac下编译和安装Lua


### 1 起因

​    在公司里都是用Windows和Linux，系统中都安装了最新版本的Lua。家里一直使用的是Mac系统，打算在Mac里下载和安装Lua。 

### 2 详细步骤

#### 2.1 下载最新的lua源码

​    从Lua的官方网站可知，目前Lua的最新版本是5.3.4。下面采用curl工具下载源代码包。

```shell
➜ /Users/aihaofeng/Documents/workspace/temp curl -R -O http://www.lua.org/ftp/lua-5.3.4.tar.gz
  % Total % Received % Xferd Average Speed Time Time Time Current
                                 Dload Upload Total Spent Left Speed
100 296k 100 296k 0 0 11179 0 0:00:27 0:00:27 --:--:-- 8833
```

#### 2.2 解压缩源代码

​    源代码包不大，很快就下载下来了。用tar命令进行解压缩。

```
➜ /Users/aihaofeng/Documents/workspace/temp tar -xvf lua-5.3.4.tar.gz
x lua-5.3.4/
x lua-5.3.4/Makefile
x lua-5.3.4/doc/
x lua-5.3.4/doc/luac.1
x lua-5.3.4/doc/manual.html
x lua-5.3.4/doc/manual.css
x lua-5.3.4/doc/contents.html
x lua-5.3.4/doc/lua.css
x lua-5.3.4/doc/osi-certified-72x60.png
x lua-5.3.4/doc/logo.gif
x lua-5.3.4/doc/lua.1
x lua-5.3.4/doc/index.css
x lua-5.3.4/doc/readme.html
x lua-5.3.4/src/
x lua-5.3.4/src/ldblib.c
x lua-5.3.4/src/lmathlib.c
x lua-5.3.4/src/loslib.c
x lua-5.3.4/src/lvm.c
x lua-5.3.4/src/ldo.h
x lua-5.3.4/src/lua.h
x lua-5.3.4/src/lgc.h
x lua-5.3.4/src/ltm.h
x lua-5.3.4/src/loadlib.c
x lua-5.3.4/src/lmem.c
x lua-5.3.4/src/lstate.h
x lua-5.3.4/src/Makefile
x lua-5.3.4/src/lzio.h
x lua-5.3.4/src/luaconf.h
x lua-5.3.4/src/lopcodes.c
x lua-5.3.4/src/lua.c
x lua-5.3.4/src/lundump.h
x lua-5.3.4/src/lbaselib.c
x lua-5.3.4/src/ltable.c
x lua-5.3.4/src/ldump.c
x lua-5.3.4/src/liolib.c
x lua-5.3.4/src/llimits.h
x lua-5.3.4/src/lfunc.h
x lua-5.3.4/src/lualib.h
x lua-5.3.4/src/lzio.c
x lua-5.3.4/src/lctype.c
x lua-5.3.4/src/lmem.h
x lua-5.3.4/src/llex.h
x lua-5.3.4/src/ltable.h
x lua-5.3.4/src/lstring.c
x lua-5.3.4/src/ldebug.h
x lua-5.3.4/src/lbitlib.c
x lua-5.3.4/src/lprefix.h
x lua-5.3.4/src/llex.c
x lua-5.3.4/src/linit.c
x lua-5.3.4/src/lobject.h
x lua-5.3.4/src/lapi.h
x lua-5.3.4/src/ldebug.c
x lua-5.3.4/src/ldo.c
x lua-5.3.4/src/lvm.h
x lua-5.3.4/src/lauxlib.c
x lua-5.3.4/src/luac.c
x lua-5.3.4/src/lctype.h
x lua-5.3.4/src/lstring.h
x lua-5.3.4/src/lcorolib.c
x lua-5.3.4/src/lutf8lib.c
x lua-5.3.4/src/lgc.c
x lua-5.3.4/src/lstate.c
x lua-5.3.4/src/lundump.c
x lua-5.3.4/src/ltablib.c
x lua-5.3.4/src/lauxlib.h
x lua-5.3.4/src/ltm.c
x lua-5.3.4/src/lparser.c
x lua-5.3.4/src/lcode.h
x lua-5.3.4/src/lobject.c
x lua-5.3.4/src/lcode.c
x lua-5.3.4/src/lopcodes.h
x lua-5.3.4/src/lfunc.c
x lua-5.3.4/src/lapi.c
x lua-5.3.4/src/lparser.h
x lua-5.3.4/src/lua.hpp
x lua-5.3.4/src/lstrlib.c
x lua-5.3.4/README
```

#### 2.3 进入源码目录进行编译

​    源代码里的Makefile文件写的很通用，直接采用make命令，加上操作系统类型参数就可以直接编译了。这里是Mac系统，所以带的参数为macosx。

```
➜ /Users/aihaofeng/Documents/workspace/temp cd lua-5.3.4

➜ /Users/aihaofeng/Documents/workspace/temp/lua-5.3.4 make
Please do 'make PLATFORM' where PLATFORM is one of these:
   aix bsd c89 freebsd generic linux macosx mingw posix solaris
See doc/readme.html for complete instructions.

➜ /Users/aihaofeng/Documents/workspace/temp/lua-5.3.4 make macosx
cd src && /Library/Developer/CommandLineTools/usr/bin/make macosx
/Library/Developer/CommandLineTools/usr/bin/make all SYSCFLAGS="-DLUA_USE_MACOSX" SYSLIBS="-lreadline" CC=cc
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lapi.o lapi.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lcode.o lcode.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lctype.o lctype.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o ldebug.o ldebug.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o ldo.o ldo.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o ldump.o ldump.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lfunc.o lfunc.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lgc.o lgc.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o llex.o llex.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lmem.o lmem.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lobject.o lobject.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lopcodes.o lopcodes.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lparser.o lparser.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lstate.o lstate.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lstring.o lstring.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o ltable.o ltable.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o ltm.o ltm.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lundump.o lundump.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lvm.o lvm.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lzio.o lzio.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lauxlib.o lauxlib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lbaselib.o lbaselib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lbitlib.o lbitlib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lcorolib.o lcorolib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o ldblib.o ldblib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o liolib.o liolib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lmathlib.o lmathlib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o loslib.o loslib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lstrlib.o lstrlib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o ltablib.o ltablib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lutf8lib.o lutf8lib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o loadlib.o loadlib.c
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o linit.o linit.c
ar rcu liblua.a lapi.o lcode.o lctype.o ldebug.o ldo.o ldump.o lfunc.o lgc.o llex.o lmem.o lobject.o lopcodes.o lparser.o lstate.o lstring.o ltable.o ltm.o lundump.o lvm.o lzio.o lauxlib.o lbaselib.o lbitlib.o lcorolib.o ldblib.o liolib.o lmathlib.o loslib.o lstrlib.o ltablib.o lutf8lib.o loadlib.o linit.o
ranlib liblua.a
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o lua.o lua.c
cc -o lua lua.o liblua.a -lm -lreadline
cc -O2 -Wall -Wextra -DLUA_COMPAT_5_2 -DLUA_USE_MACOSX -c -o luac.o luac.c
cc -o luac luac.o liblua.a -lm -lreadline
```

#### 2.4 查看编译是否成功

​    采用`make test`命令进行测试。其实就是执行编译后的可执行文件，这个可执行文件在`src/`目录下。

```shell
➜ /Users/aihaofeng/Documents/workspace/temp/lua-5.3.4 make test
src/lua -v
Lua 5.3.4 Copyright (C) 1994-2017 Lua.org, PUC-Rio
```

#### 2.5 安装Lua到环境中

​    采用`make install`命令进行安装。为了避免没有权限，加上sudo，输入密码。

```shell
➜ /Users/aihaofeng/Documents/workspace/temp/lua-5.3.4 sudo make install
Password:
cd src && mkdir -p /usr/local/bin /usr/local/include /usr/local/lib /usr/local/man/man1 /usr/local/share/lua/5.3 /usr/local/lib/lua/5.3
cd src && install -p -m 0755 lua luac /usr/local/bin
cd src && install -p -m 0644 lua.h luaconf.h lualib.h lauxlib.h lua.hpp /usr/local/include
cd src && install -p -m 0644 liblua.a /usr/local/lib
cd doc && install -p -m 0644 lua.1 luac.1 /usr/local/man/man1
```

#### 2.6 测试是否安装成功

​    在终端里直接输入lua，得到如下界面，说明安装成功。

```shell
➜ /Users/aihaofeng/Documents/workspace/temp/lua-5.3.4 lua
Lua 5.3.4 Copyright (C) 1994-2017 Lua.org, PUC-Rio
> os.exit()
```

### 3 小结

​    从上面的安装步骤看，其和在Linux下编译安装基本是一样的。

### 4 参考资料

* [MacOS Lua update to v5.3.3](https://www.jianshu.com/p/5df2e3efc0a7)
