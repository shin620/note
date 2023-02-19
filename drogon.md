I may use drogon framework in my work, so I tried to install it and record how to install it.

# Install

First, I ran centos in docker.

```
docker run -it --rm --name drogon -p 8000:8000 centos:centos7 bash
```

I needed to install openssl before installing cmake.

```
yum install wget
yum install make
cd /usr/local/src
wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz --no-check-certificate
tar -zxf openssl-1.1.1g.tar.gz
cd openssl-1.1.1g
./config --openssldir=/usr/local/ssl
make
make install
openssl version
```

But I couldn't show a version. So I do following command.

```
[root@78cd71d7a2fb openssl-1.1.1g]# openssl version
openssl: error while loading shared libraries: libssl.so.1.1: cannot open shared object file: No such file or directory
[root@78cd71d7a2fb openssl-1.1.1g]# find /usr -name "libssl.so.1.1"
/usr/local/src/openssl-1.1.1g/libssl.so.1.1
/usr/local/lib64/libssl.so.1.1
[root@78cd71d7a2fb openssl-1.1.1g]# echo "/usr/local/lib64" >> /etc/ld.so.conf.d/lib64.conf
[root@78cd71d7a2fb openssl-1.1.1g]# cat /etc/ld.so.conf.d/lib64.conf
/usr/local/lib64
[root@78cd71d7a2fb openssl-1.1.1g]# ldconfig
[root@78cd71d7a2fb openssl-1.1.1g]# openssl version
OpenSSL 1.1.1g  21 Apr 2020
[root@78cd71d7a2fb openssl-1.1.1g]#
```

I installed cmake from github to use latest version.

```

cd tmp

yum install git gcc gcc-c++

git clone https://github.com/Kitware/CMake
cd CMake/
./bootstrap && make && make install
```

I upgraded gcc.
```
yum install centos-release-scl
yum install devtoolset-8
scl enable devtoolset-8 bash
```

I installed jsoncpp.

```
git clone https://github.com/open-source-parsers/jsoncpp
cd jsoncpp/
mkdir build
cd build
cmake ..
make && make install
```

I installed uuid.

```
yum install libuuid-devel
```

I installed zlib.

```
yum install zlib-devel
```

I installed Drogon.

```
cd /tmp
git clone https://github.com/drogonframework/drogon
cd drogon
git submodule update --init
mkdir build
cd build
cmake ..
make
make install
```

I checked drogon version to ensure that it is successfully installed.

```
[root@78cd71d7a2fb build]# drogon_ctl version
     _
  __| |_ __ ___   __ _  ___  _ __
 / _` | '__/ _ \ / _` |/ _ \| '_ \
| (_| | | | (_) | (_| | (_) | | | |
 \__,_|_|  \___/ \__, |\___/|_| |_|
                 |___/

A utility for drogon
Version: 1.8.3
Git commit: 29955becc17f0bb5e183e67c07a1018c5b0e6abf
Compilation:
  Compiler: /opt/rh/devtoolset-8/root/usr/bin/c++
  Compiler ID: GNU
  Compilation flags: -std=c++17 -I/usr/local/include
Libraries:
  postgresql: no  (pipeline mode: no)
  mariadb: no
  sqlite3: no
  openssl: yes
  brotli: no
  boost: no
  hiredis: no
  c-ares: no
```

# Quick Start

I created a project at /tmp/drogonpj

```
[root@78cd71d7a2fb build]# cd /tmp
[root@78cd71d7a2fb tmp]# drogon_ctl create project drogonpj
create a project named drogonpj
[root@78cd71d7a2fb tmp]# cd drogonpj/
[root@78cd71d7a2fb drogonpj]# tree .
.
|-- CMakeLists.txt
|-- build
|-- config.json
|-- controllers
|-- filters
|-- main.cc
|-- models
|   `-- model.json
|-- plugins
|-- test
|   |-- CMakeLists.txt
|   `-- test_main.cc
`-- views

7 directories, 6 files
```

I changed port number in main.cc from 80 to 8000.

I built the project.

```
cd build
cmake ..
make
```

I ran the program.

```
[root@78cd71d7a2fb build]# echo '<h1>Hello Drogon!</h1>' >>index.html
[root@78cd71d7a2fb build]# ./drogonpj
```

After that, I accessed 'http://localhost:8000' and saw 'Hello Drogon!' in the browser.

I tried dynamic site too.

I created controller.

```
drogon_ctl create controller TestCtrl
```

And I edited two files.

TestCtrl.h
```
#pragma once
#include <drogon/HttpSimpleController.h>
using namespace drogon;
class TestCtrl:public drogon::HttpSimpleController<TestCtrl>
{
public:
    virtual void asyncHandleHttpRequest(const HttpRequestPtr &req,
                                        std::function<void (const HttpResponsePtr &)> &&callback)override;
    PATH_LIST_BEGIN
    //list path definitions here

    //example
    //PATH_ADD("/path","filter1","filter2",HttpMethod1,HttpMethod2...);

    PATH_ADD("/",Get,Post);
    PATH_ADD("/test",Get);
    PATH_LIST_END
};
```

TestCtrl.cc
```
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void (const HttpResponsePtr &)> &&callback)
{
    //write your application logic here
    auto resp=HttpResponse::newHttpResponse();
    //NOTE: The enum constant below is named "k200OK" (as in 200 OK), not "k2000K".
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody("Hello World!");
    callback(resp);
}
```

I rebuilt the project.

```
cd ../build
cmake ..
make
./drogonpj
```

I accessed 'http://localhost:8000' or 'http://localhost:8000/test' and saw 'Hello World!'.
