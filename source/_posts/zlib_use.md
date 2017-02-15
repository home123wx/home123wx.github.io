---
title:  Windows下zlib库的使用
---

#### 下载zlib库
[zlib-1.2.10.tar.gz](https://pan.baidu.com/s/1c2aeiXU) 

#### 编译
解压后 `zlib-1.2.10/win32/Makefile.msc` 文件中介绍了如何编译：
> Makefile for zlib using Microsoft (Visual) C 
> zlib is copyright (C) 1995-2006 Jean-loup Gailly and Mark Adler
> Usage:
> nmake -f win32/Makefile.msc                                                (standard build)
> nmake -f win32/Makefile.msc LOC=-DFOO                            (nonstandard build)
> nmake -f win32/Makefile.msc LOC="-DASMV -DASMINF" \
> OBJA="inffas32.obj match686.obj"                                         (use ASM code, x86)
> nmake -f win32/Makefile.msc AS=ml64 LOC="-DASMV -DASMINF -I." \
> OBJA="inffasx64.obj gvmat64.obj inffas8664.obj"  (use ASM code, x64)

运行 `Visual Studio Tools/Visual Studio 2008 Command Prompt`命令控制行
``` 
# cd 解压目录
# nmake -f win32/Makefile.msc
```

#### 库的使用
只使用`zlib`库的话把下图中头文件放到工程路径下，包含`zlib.h`头文件，即可使用。

```cpp
#include "zlib/zlib.h"

void Test1();

int _tmain(int argc, _TCHAR* argv[]) 
{
	Test1();
	
	getchar();
}

void Test1()
{
    /* 原始数据 */
    unsigned char strSrc[] = "hello world! 中文";
    unsigned char buf[1024] = {0};
    unsigned char strDst[1024] = {0};
    unsigned long srcLen = sizeof(strSrc);
    unsigned long bufLen = sizeof(buf);
    unsigned long dstLen = sizeof(strDst);

    printf("Src string:%s\nLength:%ld\n", strSrc, srcLen);

    /* 压缩 */
    compress(buf, &bufLen, strSrc, srcLen);
    printf("After Compressed Length:%ld\n", bufLen);

    /* 解压缩 */
    uncompress(strDst, &dstLen, buf, bufLen);
    printf("After UnCompressed Length:%ld\n",dstLen);

    printf("UnCompressed String:%s\n",strDst);
}
```

使用minizip(压缩包)再额外包含`zlib-1.2.10\contrib`目录下文件到工程路径中

并在项目中添加下图头文件和实现文件

在压缩包中创建文件并写入内容
```cpp
#include "zlib/zip.h"

void Test3(const char* path);

int _tmain(int argc, _TCHAR* argv[]) 
{
	Test3();
	getchar();
}

void Test3(const char* path)
{
    zipFile zf = zipOpen64(path, APPEND_STATUS_CREATE);
    if (zf == NULL) {
        printf("创建%s失败\n", path);
        return;
    }
    
    //初始化写入zip的文件信息  
    zip_fileinfo zi;  
    zi.tmz_date.tm_sec = zi.tmz_date.tm_min  = zi.tmz_date.tm_hour =  
                         zi.tmz_date.tm_mday = zi.tmz_date.tm_mon =
                         zi.tmz_date.tm_year = 0;  
    zi.dosDate = 0;  
    zi.internal_fa = 0;  
    zi.external_fa = 0;  

    //1
    zipOpenNewFileInZip(zf, "t\\t.txt", &zi, NULL,
                        0, NULL, 0, NULL, Z_DEFLATED, Z_DEFAULT_COMPRESSION);
    char* buf = "Hello_1111";
    zipWriteInFileInZip(zf, buf, strlen(buf));
    //关闭zip文件  
    zipCloseFileInZip(zf);

    //2
    zipOpenNewFileInZip(zf, "t1\\t.txt", &zi, NULL,
                        0, NULL, 0, NULL, Z_DEFLATED, Z_DEFAULT_COMPRESSION);
    zipWriteInFileInZip(zf, buf, strlen(buf));
    //关闭zip文件  
    zipCloseFileInZip(zf);

    zipClose(zf, NULL);
}
```


