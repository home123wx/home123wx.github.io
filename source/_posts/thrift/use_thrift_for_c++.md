---
title: Thrift使用 -- C++ 例子
tags: [thrift]
date: 2019-01-03
categories: C++
---


[源码](https://github.com/home123wx/thrift_demo)

#### 编写Thrift文件
定义`student`结构，写入文件`student.thrift`中，内容如下：
```
struct Student{
    1: i32 sno,
    2: string sname,
    3: bool ssex,
    4: i16 sage,
}

service Serv{
    void put(1: Student s), 
    Student get(1:i32 sno),
}
```

#### 生成CPP文件
执行如下命令：
```
thrift --gen cpp thrift_file/student.thrift
```
会生成`gen-cpp`目录，包含有
```
-rw-r--r-- 1 root root 20494 Jan  3 11:38 Serv.cpp
-rw-r--r-- 1 root root 11154 Jan  3 10:34 Serv.h
-rw-r--r-- 1 root root  1482 Jan  3 11:38 Serv_server.skeleton.cpp
-rw-r--r-- 1 root root   261 Jan  3 10:34 student_constants.cpp
-rw-r--r-- 1 root root   347 Jan  3 10:34 student_constants.h
-rw-r--r-- 1 root root  4041 Jan  3 10:34 student_types.cpp
-rw-r--r-- 1 root root  1783 Jan  3 10:34 student_types.h
```

#### 编写服务端
复制`gen-cpp`目录下`Serv_server.skeleton.cpp`改名为`server.cpp`，修改代码：
```
#include "Serv.h"
#include <thrift/protocol/TBinaryProtocol.h>
#include <thrift/server/TSimpleServer.h>
#include <thrift/transport/TServerSocket.h>
#include <thrift/transport/TBufferTransports.h>
#include <map>

using namespace ::apache::thrift;
using namespace ::apache::thrift::protocol;
using namespace ::apache::thrift::transport;
using namespace ::apache::thrift::server;

class ServHandler : virtual public ServIf {
  std::map<int, Student> m_studentMap;
 public:
  ServHandler() {
    // Your initialization goes here
  }

  void put(const Student& s) {
    // Your implementation goes here
    m_studentMap.insert(std::make_pair(s.sno, s));
    printf("put sno=%d, sname=%s, ssex=%d, sage=%d\n", s.sno, s.sname.c_str(), s.ssex, s.sage);
  }

  void get(Student& _return, const int32_t sno) {
    printf("get sno=%d\n", sno);
    // Your implementation goes here
    if (m_studentMap.find(sno) != m_studentMap.end()) {
        _return = m_studentMap[sno];
    } else {
        _return.sno = -1;
    }
  }

};

int main(int argc, char **argv) {
  int port = 9090;
  ::apache::thrift::stdcxx::shared_ptr<ServHandler> handler(new ServHandler());
  ::apache::thrift::stdcxx::shared_ptr<TProcessor> processor(new ServProcessor(handler));
  ::apache::thrift::stdcxx::shared_ptr<TServerTransport> serverTransport(new TServerSocket(port));
  ::apache::thrift::stdcxx::shared_ptr<TTransportFactory> transportFactory(new TBufferedTransportFactory());
  ::apache::thrift::stdcxx::shared_ptr<TProtocolFactory> protocolFactory(new TBinaryProtocolFactory());

  TSimpleServer server(processor, serverTransport, transportFactory, protocolFactory);
  server.serve();
  return 0;
}
```

#### 编写客户端
```
#include "Serv.h"

#include <transport/TSocket.h>
#include <transport/TBufferTransports.h>
#include <protocol/TBinaryProtocol.h>
 
using namespace apache::thrift;
using namespace apache::thrift::protocol;
using namespace apache::thrift::transport;

int main(int argc, char **argv) {
    ::apache::thrift::stdcxx::shared_ptr<TSocket> socket(new TSocket("localhost", 9090));
    ::apache::thrift::stdcxx::shared_ptr<TTransport> transport(new TBufferedTransport(socket));
    ::apache::thrift::stdcxx::shared_ptr<TProtocol> protocol(new TBinaryProtocol(transport));

    ServClient client(protocol);
    transport->open();

    Student s;
    s.sno = 123;
    s.sname = "张三";
    s.ssex = 1;
    s.sage = 30;   

    client.put(s);
    
    Student ss;
    printf("get! sno=123\n");
    client.get(ss, 123);
    if (ss.sno != -1) {
        printf("find name=%s, sex=%d, age=%d, no=%d\n", ss.sname.c_str(), ss.ssex, ss.sage, ss.sno);
    }

    printf("get! sno=111\n");
    client.get(ss, 111);
    if (ss.sno != -1) {
        printf("find name=%s, sex=%d, age=%d, no=%d\n", ss.sname.c_str(), ss.ssex, ss.sage, ss.sno);
    } else {
        printf("not find\n");
    }

    transport->close();

    return 0;
}
```

#### 运行结果

服务端：
```
[root@happy-anan demo_3]# ./server 
put sno=123, sname=张三, ssex=1, sage=30
get sno=123
get sno=111
```

客户端：
```
[root@happy-anan demo_3]# ./client 
get! sno=123
find name=张三, sex=1, age=30, no=123
get! sno=111
not find
```

#### 编写Makefile
```
OBJ             = ${OBJ_DIR}student_constants.o ${OBJ_DIR}student_types.o ${OBJ_DIR}Serv.o
OBJ_DIR		= ./obj/
GCC		= g++ -std=c++11
LIBS_DIR   	= -L/usr/local/lib
THRIFT_DIR 	= /usr/local/include/thrift
LIBS		= -lpthread -lthrift
CPP_OPTS	= -Wall -O2
GEN_INC  	= -I./gen-cpp -I/usr/local/include/thrift
RUN_LIBS 	= -Wl,-rpath -Wl,/usr/local/lib

demo_3_a 	= ${OBJ_DIR}demo_3.a

all:demo_3.a server client

demo_3.a:${OBJ}
	ar -cr $(OBJ_DIR)$@ $(OBJ)

server:${OBJ_DIR}server.o
	${GCC} ${CPP_OPTS} $< -o $@ ${demo_3_a} ${RUN_LIBS} ${LIBS}

client:${OBJ_DIR}client.o
	${GCC} ${CPP_OPTS} $< -o $@ ${demo_3_a} ${RUN_LIBS} ${LIBS}

${OBJ_DIR}server.o:server.cpp
	${GCC} -c ${CPP_OPTS} ${GEN_INC}  $< -o $@

${OBJ_DIR}client.o:client.cpp
	${GCC} -c ${CPP_OPTS} ${GEN_INC}  $< -o $@

${OBJ_DIR}student_constants.o:./gen-cpp/student_constants.cpp
	${GCC} -c ${CPP_OPTS} $< -o $@

${OBJ_DIR}student_types.o:./gen-cpp/student_types.cpp
	${GCC} -c ${CPP_OPTS} $< -o $@

${OBJ_DIR}Serv.o:./gen-cpp/Serv.cpp
	${GCC} -c ${CPP_OPTS} $< -o $@

clean:
	${RM} -rf obj/*
	${RM} -rf client server
```

#### 遇到的问题
```
# 1. 直接使用g++编译
g++ -g -I/usr/local/include/thrift -L/usr/local/lib/ Serv.cpp student_types.cpp student_constants.cpp Serv_server.skeleton.cpp -o server -lthrift 
# 有错误提示
undefined reference to `apache::thrift::server::TSimpleServer……`
网上有说把 `-lthrift` 放到最后，修改后无效。
使用Makefile后正常编译。
```
