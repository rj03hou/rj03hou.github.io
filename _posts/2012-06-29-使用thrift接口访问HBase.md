---
layout: post 
title: "使用thrift接口访问HBase"
subTitle: 
heroImageUrl: 
date: 2012-6-29
tags: ["Hbase"]
keywords: 
---

hbase用java来操作是最方便，也效率最高的方式。但java并非轻量级，不方便在任何环境下调试。而且不同的开发人员熟悉的语言不一样，开发效率也不一样。hbase 通过thrift，还可以用python,ruby,cpp,perl等语言来操作。

thrift是facebook开发开源的类似google的protobuf的远程调用组件。但protobuf只有数据的序列化，且只支持二进制协议，没有远程调用部分。protobuf原生支持cpp,python,java,另外还有第三方实现的objectc，ruby等语言。而thrift是实现了序列化，传输，协议定义，远程调用等功能，跨语言能力更多。某些方面二者可以互相替代，但一些方面则各有适用范围。

Thrift的一个介绍：[http://www.ibm.com/developerworks/cn/java/j-lo-apachethrift/](http://www.ibm.com/developerworks/cn/java/j-lo-apachethrift/)

**安装过程：**

<pre lang="sh">
yum install automake libtool flex bison pkgconfig gcc-c++ boost-devel libevent-devel zlib-devel python-devel ruby-devel
wget https://github.com/downloads/libevent/libevent/libevent-2.0.19-stable.tar.gz
./configure --prefix=/usr/local/libevent-2.0.19
make && make install   
wget libbit-vector-perl_7.2.orig.tar.gz
cd Bit-Vector-7.2/ 
perl Makefile.PL  
make && install 
./configure --prefix=/usr/local/thrift-0.8.0 --with-libevent=/usr/local/libevent-2.0.19/
make && make install
</pre>

**启动：**
<pre lang="sh">
nohup hbase thrift -threadpool start >thrift.log 2>&1 &;
</pre>
**示例程序：**

python简单示例程序如下：
<pre lang="python">#!/usr/bin/env python
import sys
sys.path.append('/usr/local/thrift-0.8.0/bin/gen-py')
from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol
from hbase import Hbase
from hbase.ttypes import *
transport = TSocket.TSocket('localhost', 9090)
transport = TTransport.TBufferedTransport(transport)
protocol = TBinaryProtocol.TBinaryProtocol(transport)
client = Hbase.Client(protocol)
transport.open()
tableName="test"
#create table t1 with column family contents
try:
    client.createTable(tableName,[ColumnDescriptor(name="contents:", maxVersions=1),])
except AlreadyExists:
    client.disableTable(tableName)
    client.deleteTable(tableName)
    client.createTable(tableName,[ColumnDescriptor(name="contents:", maxVersions=1),])
#put thress rows to test
mutations = [Mutation(column="contents:", value="content")]
client.mutateRow(tableName,"1", mutations)
client.mutateRow(tableName,"2", mutations)
client.mutateRow(tableName,"3", mutations)
#get row from test
print client.getRow("t1","1")</pre>
python更加复杂一些的一个例子：[http://blog.csdn.net/fanzy618/article/details/4091602](http://blog.csdn.net/fanzy618/article/details/4091602)

perl示例程序见：[http://code.google.com/p/hbase-thrift/source/browse/trunk/perl/test/tables.t?r=2](http://code.google.com/p/hbase-thrift/source/browse/trunk/perl/test/tables.t?r=2)

通过对python接口的使用，推荐使用java，因为java的文档更加丰富一些。如果使用python或者perl需要阅读生成的接口文件。

**遇到的问题**
<div>需要安装教新版本的libevent，否则在编译的会报下面的错误
<pre>
libtool: compile:  g++ -DHAVE_CONFIG_H -I. -I../.. -I/usr/include/include -I./src -Wall -g -O2 -MT libthriftnb_la-TNonblockingServer.lo -MD -MP -MF .deps/libthriftnb_la-TNonblockingServer.Tpo -c src/server/TNonblockingServer.cpp  -fPIC -DPIC -o .libs/libthriftnb_la-TNonblockingServer.o
src/server/TNonblockingServer.cpp: In destructor 'virtual apache::thrift::server::TNonblockingServer::~TNonblockingServer()':
src/server/TNonblockingServer.cpp:539: error: 'event_base_free' was not declared in this scope
make[4]: *** [libthriftnb_la-TNonblockingServer.lo] Error 1
</pre>
</div>
<div>python客户端抛出错误：
<pre>socket.error: [Errno 104] Connection reset by peer</pre>
解决方法：
<pre> 
12/06/01 17:55:40 ERROR server.THsHaServer: Read an invalid frame size of -2147418111. Are you using TFramedTransport on the client side?
 要想提高传输效率，必须使用TFramedTransport或TBufferedTransport.但对-hsha，-nonblocking两种服务器模式，必须使用TFramedTransport。将其改为线程方式试试。
 [zhouhh@Hadoop48 hbase-0.94.0]$ hbase thrift -p 19090 -threadpool start
 ...
 12/06/01 18:02:17 DEBUG thrift.ThriftServerRunner: Using binary protocol
 12/06/01 18:02:17 INFO thrift.ThriftServerRunner: starting TBoundedThreadPoolServer on /0.0.0.0:19090; min worker threads=16, max worker threads=1000, max queued requests=1000
</pre>
</div>
**Thrift不同语言环境需求：**
<pre>
	* C++
		* Boost 1.33.1+
		* libevent (optional, to build the nonblocking server)
		* zlib (optional)

	* Java
		* Java 1.5+
		* Apache Ant
		* Apache Ivy (recommended)
		* Apache Commons Lang (recommended)
		* SLF4J

	* C#: Mono 1.2.4+ (and pkg-config to detect it) or Visual Studio 2005+
	* Python 2.4+ (including header files for extension modules)
	* PHP 5.0+ (optionally including header files for extension modules)
	* Ruby 1.8+ (including header files for extension modules)
	* Erlang R12 (R11 works but not recommended)
	* Perl 5
		* Bit::Vector
		* Class::Accessor

packages 需求：

	1. For ruby, install ruby-full ruby-dev librspec-ruby rake rubygems libdaemons-ruby libgemplugin-ruby mongrel.

	2. For python, install python-dev python-twisted.

	3. For perl, install libbit-vector-perl.

	4. For php, install php5-dev php5-cli.

	5. For c_glib, install libglib2.0-dev (Debian Lenny Users => sudo apt-get -t lenny-backports install libglib2.0-dev)

	6. For erlang, install erlang-base erlang-eunit erlang-dev

	7. For csharp, install mono-gmcs libmono-dev libmono-system-web2.0-cil

	8. For haskell, install ghc6 cabal-install libghc6-binary-dev libghc6-network-dev libghc6-http-dev
</pre>
<div>抄袭来源：</div>
<div>[http://abloz.com/2012/06/01/python-operating-hbase-thrift-to.html](http://abloz.com/2012/06/01/python-operating-hbase-thrift-to.html)</div>
<div>[http://blog.csdn.net/fanzy618/article/details/4091602](http://blog.csdn.net/fanzy618/article/details/4091602)</div>