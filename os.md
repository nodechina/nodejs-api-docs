# 系统

    稳定性: 4 - API 冻结

提供一些基本的操作系统相关函数。

使用 `require('os')` 访问这个模块。

## os.tmpdir()

返回操作系统的默认临时文件夹

## os.endianness()

返回 CPU 的字节序，可能的是 "BE" 或 "LE"。

## os.hostname()

返回操作系统的主机名。

## os.type()

返回操作系统名。

## os.platform()

返回操作系统名

## os.arch()

返回操作系统 CPU 架构，可能的值有 "x64"、"arm" 和 "ia32"。

## os.release()

返回操作系统的发行版本

## os.uptime()

返回操作系统运行的时间，以秒为单位。

## os.loadavg()

显示原文其他翻译纠错
返回一个包含 1、5、15 分钟平均负载的数组。

平均负载是系统的一个指标，操作系统计算，用一个很小的数字表示。理论上来说，平均负载最好比系统里的 CPU 低。  
  
平均负载是一个非常 UNIX-y 的概念，windows 系统没有相同的概念。所以 windows 总是返回 `[0, 0, 0]`。

## os.totalmem()

返回系统内存总量，单位为字节。

## os.freemem()

返回操作系统空闲内存量，单位是字节。

## os.cpus()

返回一个对象数组，包含所安装的每个 CPU/内核的信息：型号、速度（单位 MHz）、时间（一个包含 user、nice、sys、idle 和 irq 所使用 CPU/内核毫秒数的对象）。

os.cpus 的例子:

    [ { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 252020,
           nice: 0,
           sys: 30340,
           idle: 1070356870,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 306960,
           nice: 0,
           sys: 26980,
           idle: 1071569080,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 248450,
           nice: 0,
           sys: 21750,
           idle: 1070919370,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 256880,
           nice: 0,
           sys: 19430,
           idle: 1070905480,
           irq: 20 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 511580,
           nice: 20,
           sys: 40900,
           idle: 1070842510,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 291660,
           nice: 0,
           sys: 34360,
           idle: 1070888000,
           irq: 10 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 308260,
           nice: 0,
           sys: 55410,
           idle: 1071129970,
           irq: 880 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 266450,
           nice: 1480,
           sys: 34920,
           idle: 1072572010,
           irq: 30 } } ]

## os.networkInterfaces()

获得网络接口列表：

    { lo:
       [ { address: '127.0.0.1',
           netmask: '255.0.0.0',
           family: 'IPv4',
           mac: '00:00:00:00:00:00',
           internal: true },
         { address: '::1',
           netmask: 'ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff',
           family: 'IPv6',
           mac: '00:00:00:00:00:00',
           internal: true } ],
      eth0:
       [ { address: '192.168.1.108',
           netmask: '255.255.255.0',
           family: 'IPv4',
           mac: '01:02:03:0a:0b:0c',
           internal: false },
         { address: 'fe80::a00:27ff:fe4e:66a1',
           netmask: 'ffff:ffff:ffff:ffff::',
           family: 'IPv6',
           mac: '01:02:03:0a:0b:0c',
           internal: false } ] }

## os.EOL

定义了操作系统的 End-of-line 的常量。
