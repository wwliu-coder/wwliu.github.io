+++
date = '2025-05-14T11:23:16+08:00'
draft = false
title = '如何从源码安装redis'
+++
由于博主比较菜，连redis是什么都没接触过，另外近期想冲一下大厂，所以最近会就redis相关的问题总结成一个专题，那么开始吧。

# 专题一：安装源码并总结redis代码整体结构
**1. 安装源码**

源码安装参考：[https://redis.io/docs/latest/operate/oss_and_stack/install/build-stack/ubuntu-focal/](https://redis.io/docs/latest/operate/oss_and_stack/install/build-stack/ubuntu-focal/)
```bash 
# Update development tools and libraries
apt-get update
apt-get install -y sudo
sudo apt-get install -y --no-install-recommends \
    ca-certificates \
    wget \
    dpkg-dev \
    gcc \
    g++ \
    libc6-dev \
    libssl-dev \
    make \
    git \
    python3 \
    python3-pip \
    python3-venv \
    python3-dev \
    unzip \
    rsync \
    clang \
    automake \
    autoconf \
    gcc-10 \
    g++-10 \
    libtool

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 100 --slave /usr/bin/g++ g++ /usr/bin/g++-10

pip3 install cmake==3.31.6
sudo ln -sf /usr/local/bin/cmake /usr/bin/cmake
cmake --version

# Download source code
wget -O redis-8.0.0.tar.gz https://github.com/redis/redis/archive/refs/tags/8.0.0.tar.gz
tar xvf redis-8.0.0.tar.gz
rm -rf redis-8.0.0.tar.gz

# Make
make -j "$(nproc)" all
sudo make install
redis-server --version
redis-cli --version

# Testing
make test

# Start redis-server with redis.conf
redis-server redis.conf
redis-cli INFO # in another bash
```

**2. 源码结构剖析**

源码结构剖析参考极客时间的大佬，如图所示
<img src="/wwliu.github.io/images/redis_structure.png"/>

1. deps 目录

主要是redis依赖的第三方库，包括 Redis 的 C 语言版本客户端代码 hiredis、jemalloc 内存分配器代码、readline 功能的替代代码 linenoise，以及 lua 脚本代码

2. src 目录

包含了 Redis 所有功能模块的代码文件，也是 Redis 源码的重要组成部分

3. tests 目录

Redis 实现的测试代码可以分成四部分，分别是单元测试（对应 unit 子目录），Redis Cluster 功能测试（对应 cluster 子目录）、哨兵功能测试（对应 sentinel 子目录）、主从复制功能测试（对应 integration 子目录）。

4. utils 目录

属于辅助性功能，包括用于创建 Redis Cluster 的脚本、用于测试 LRU 算法效果的程序，以及可视化 rehash 过程的程序

5. 配置文件

Redis 源码总目录下其实还包含了两个重要的配置文件，一个是 Redis 实例的配置文件 redis.conf，另一个是哨兵的配置文件 sentinel.conf

附上我下载的redis-8.0.0版本的本地文件
```bash
(base) usrname@HOST-xx-xx-xx-xx:~/ws/redis-8.0.0$ tree -L 2 -I '*.o'
.
├── 00-RELEASENOTES
├── BUGS
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── INSTALL
├── LICENSE.txt
├── MANIFESTO
├── Makefile
├── README.md
├── REDISCONTRIBUTIONS.txt
├── SECURITY.md
├── TLS.md
├── codecov.yml
├── deps
│   ├── Makefile
│   ├── README.md
│   ├── fast_float
│   ├── fpconv
│   ├── hdr_histogram
│   ├── hiredis
│   ├── jemalloc
│   ├── linenoise
│   └── lua
├── dump.rdb
├── modules
│   ├── Makefile
│   ├── common.mk
│   ├── redisbloom
│   ├── redisearch
│   ├── redisjson
│   ├── redistimeseries
│   └── vector-sets
├── redis-full.conf
├── redis.conf
├── runtest
├── runtest-cluster
├── runtest-moduleapi
├── runtest-sentinel
├── sentinel.conf
├── src
│   ├── Makefile
│   ├── Makefile.dep
│   ├── acl.c
│   ├── acl.d
│   ├── adlist.c
│   ├── adlist.d
│   ├── adlist.h
│   ├── ae.c
│   ├── ae.d
│   ├── ae.h
│   ├── ae_epoll.c
│   ├── ae_evport.c
│   ├── ae_kqueue.c
│   ├── ae_select.c
│   ├── anet.c
│   ├── anet.d
│   ├── anet.h
│   ├── aof.c
│   ├── aof.d
│   ├── asciilogo.h
│   ├── atomicvar.h
│   ├── bio.c
│   ├── bio.d
│   ├── bio.h
│   ├── bitops.c
│   ├── bitops.d
│   ├── blocked.c
│   ├── blocked.d
│   ├── cJSON.d
│   ├── call_reply.c
│   ├── call_reply.d
│   ├── call_reply.h
│   ├── childinfo.c
│   ├── childinfo.d
│   ├── cli_commands.c
│   ├── cli_commands.d
│   ├── cli_commands.h
│   ├── cli_common.c
│   ├── cli_common.d
│   ├── cli_common.h
│   ├── cluster.c
│   ├── cluster.d
│   ├── cluster.h
│   ├── cluster_legacy.c
│   ├── cluster_legacy.d
│   ├── cluster_legacy.h
│   ├── commands
│   ├── commands.c
│   ├── commands.d
│   ├── commands.def
│   ├── commands.h
│   ├── config.c
│   ├── config.d
│   ├── config.h
│   ├── connection.c
│   ├── connection.d
│   ├── connection.h
│   ├── connhelpers.h
│   ├── crc16.c
│   ├── crc16.d
│   ├── crc16_slottable.h
│   ├── crc64.c
│   ├── crc64.d
│   ├── crc64.h
│   ├── crccombine.c
│   ├── crccombine.d
│   ├── crccombine.h
│   ├── crcspeed.c
│   ├── crcspeed.d
│   ├── crcspeed.h
│   ├── db.c
│   ├── db.d
│   ├── debug.c
│   ├── debug.d
│   ├── debugmacro.h
│   ├── defrag.c
│   ├── defrag.d
│   ├── dict.c
│   ├── dict.d
│   ├── dict.h
│   ├── ebuckets.c
│   ├── ebuckets.d
│   ├── ebuckets.h
│   ├── endianconv.c
│   ├── endianconv.d
│   ├── endianconv.h
│   ├── eval.c
│   ├── eval.d
│   ├── eventnotifier.c
│   ├── eventnotifier.d
│   ├── eventnotifier.h
│   ├── evict.c
│   ├── evict.d
│   ├── expire.c
│   ├── expire.d
│   ├── fmacros.h
│   ├── fmtargs.h
│   ├── function_lua.c
│   ├── function_lua.d
│   ├── functions.c
│   ├── functions.d
│   ├── functions.h
│   ├── geo.c
│   ├── geo.d
│   ├── geo.h
│   ├── geohash.c
│   ├── geohash.d
│   ├── geohash.h
│   ├── geohash_helper.c
│   ├── geohash_helper.d
│   ├── geohash_helper.h
│   ├── hnsw.d
│   ├── hyperloglog.c
│   ├── hyperloglog.d
│   ├── intset.c
│   ├── intset.d
│   ├── intset.h
│   ├── iothread.c
│   ├── iothread.d
│   ├── kvstore.c
│   ├── kvstore.d
│   ├── kvstore.h
│   ├── latency.c
│   ├── latency.d
│   ├── latency.h
│   ├── lazyfree.c
│   ├── lazyfree.d
│   ├── listpack.c
│   ├── listpack.d
│   ├── listpack.h
│   ├── listpack_malloc.h
│   ├── localtime.c
│   ├── localtime.d
│   ├── logreqres.c
│   ├── logreqres.d
│   ├── lolwut.c
│   ├── lolwut.d
│   ├── lolwut.h
│   ├── lolwut5.c
│   ├── lolwut5.d
│   ├── lolwut6.c
│   ├── lolwut6.d
│   ├── lzf.h
│   ├── lzfP.h
│   ├── lzf_c.c
│   ├── lzf_c.d
│   ├── lzf_d.c
│   ├── lzf_d.d
│   ├── memtest.c
│   ├── memtest.d
│   ├── mkreleasehdr.sh
│   ├── module.c
│   ├── module.d
│   ├── modules
│   ├── monotonic.c
│   ├── monotonic.d
│   ├── monotonic.h
│   ├── mstr.c
│   ├── mstr.d
│   ├── mstr.h
│   ├── mt19937-64.c
│   ├── mt19937-64.d
│   ├── mt19937-64.h
│   ├── multi.c
│   ├── multi.d
│   ├── networking.c
│   ├── networking.d
│   ├── notify.c
│   ├── notify.d
│   ├── object.c
│   ├── object.d
│   ├── pqsort.c
│   ├── pqsort.d
│   ├── pqsort.h
│   ├── pubsub.c
│   ├── pubsub.d
│   ├── quicklist.c
│   ├── quicklist.d
│   ├── quicklist.h
│   ├── rand.c
│   ├── rand.d
│   ├── rand.h
│   ├── rax.c
│   ├── rax.d
│   ├── rax.h
│   ├── rax_malloc.h
│   ├── rdb.c
│   ├── rdb.d
│   ├── rdb.h
│   ├── redis-benchmark
│   ├── redis-benchmark.c
│   ├── redis-benchmark.d
│   ├── redis-check-aof
│   ├── redis-check-aof.c
│   ├── redis-check-aof.d
│   ├── redis-check-rdb
│   ├── redis-check-rdb.c
│   ├── redis-check-rdb.d
│   ├── redis-cli
│   ├── redis-cli.c
│   ├── redis-cli.d
│   ├── redis-sentinel
│   ├── redis-server
│   ├── redis-trib.rb
│   ├── redisassert.c
│   ├── redisassert.d
│   ├── redisassert.h
│   ├── redismodule.h
│   ├── release.c
│   ├── release.d
│   ├── release.h
│   ├── replication.c
│   ├── replication.d
│   ├── resp_parser.c
│   ├── resp_parser.d
│   ├── resp_parser.h
│   ├── rio.c
│   ├── rio.d
│   ├── rio.h
│   ├── script.c
│   ├── script.d
│   ├── script.h
│   ├── script_lua.c
│   ├── script_lua.d
│   ├── script_lua.h
│   ├── sds.c
│   ├── sds.d
│   ├── sds.h
│   ├── sdsalloc.h
│   ├── sentinel.c
│   ├── sentinel.d
│   ├── server.c
│   ├── server.d
│   ├── server.h
│   ├── setcpuaffinity.c
│   ├── setcpuaffinity.d
│   ├── setproctitle.c
│   ├── setproctitle.d
│   ├── sha1.c
│   ├── sha1.d
│   ├── sha1.h
│   ├── sha256.c
│   ├── sha256.d
│   ├── sha256.h
│   ├── siphash.c
│   ├── siphash.d
│   ├── slowlog.c
│   ├── slowlog.d
│   ├── slowlog.h
│   ├── socket.c
│   ├── socket.d
│   ├── solarisfixes.h
│   ├── sort.c
│   ├── sort.d
│   ├── sparkline.c
│   ├── sparkline.d
│   ├── sparkline.h
│   ├── stream.h
│   ├── strl.c
│   ├── strl.d
│   ├── syncio.c
│   ├── syncio.d
│   ├── syscheck.c
│   ├── syscheck.d
│   ├── syscheck.h
│   ├── t_hash.c
│   ├── t_hash.d
│   ├── t_list.c
│   ├── t_list.d
│   ├── t_set.c
│   ├── t_set.d
│   ├── t_stream.c
│   ├── t_stream.d
│   ├── t_string.c
│   ├── t_string.d
│   ├── t_zset.c
│   ├── t_zset.d
│   ├── testhelp.h
│   ├── threads_mngr.c
│   ├── threads_mngr.d
│   ├── threads_mngr.h
│   ├── timeout.c
│   ├── timeout.d
│   ├── tls.c
│   ├── tls.d
│   ├── tracking.c
│   ├── tracking.d
│   ├── unix.c
│   ├── unix.d
│   ├── util.c
│   ├── util.d
│   ├── util.h
│   ├── valgrind.sup
│   ├── version.h
│   ├── vset.d
│   ├── ziplist.c
│   ├── ziplist.d
│   ├── ziplist.h
│   ├── zipmap.c
│   ├── zipmap.d
│   ├── zipmap.h
│   ├── zmalloc.c
│   ├── zmalloc.d
│   └── zmalloc.h
├── tests
│   ├── README.md
│   ├── assets
│   ├── cluster
│   ├── helpers
│   ├── instances.tcl
│   ├── integration
│   ├── modules
│   ├── sentinel
│   ├── support
│   ├── test_helper.tcl
│   ├── tmp
│   └── unit
└── utils
    ├── build-static-symbols.tcl
    ├── cluster_fail_time.tcl
    ├── corrupt_rdb.c
    ├── create-cluster
    ├── gen-test-certs.sh
    ├── generate-command-code.py
    ├── generate-commands-json.py
    ├── generate-fmtargs.py
    ├── generate-module-api-doc.rb
    ├── graphs
    ├── hyperloglog
    ├── install_server.sh
    ├── lru
    ├── redis-copy.rb
    ├── redis-sha1.rb
    ├── redis_init_script
    ├── redis_init_script.tpl
    ├── releasetools
    ├── reply_schema_linter.js
    ├── req-res-log-validator.py
    ├── req-res-validator
    ├── speed-regression.tcl
    ├── srandmember
    ├── systemd-redis_multiple_servers@.service
    ├── systemd-redis_server.service
    ├── tracking_collisions.c
    └── whatisdoing.sh

35 directories, 350 files
```