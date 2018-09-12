---
title:  Windows 使用 libtiff
tags: [c++,windows,libtiff]
date: 2016-10-12
categories: C++
---

#### 准备工作
* 下载 [tiff-4.0.6](http://pan.baidu.com/s/1kVhQJyf)
* 解压  `[工作目录]/tiff/tiff-4.0.6`

#### 编译 libtiff
* 启动 VS Tools 中`命令行工具`
* 进入工作路径 `cd /d [解压路径]`
* 执行编译命令 `nmake /f makefile.vc`

#### 提取头文件和库文件
* 在任意路径，新建目录 `libdiff`, 在`libdiff`中新建`include 和 lib`
* 拷贝 `[解压路径]/libtiff/[libtiff.dll, libtiff.lib, libtiff_i.lib]` 到 lib 目录中
* 拷贝 `解压路径/libtiff/[tif_config.h, tif_dir.h, tiff.h, tiffconf.h, tiffio.h, tiffiop.h, tiffvers.h, uvcode.h]` 到 include 目录中

#### 编写 Demo
* VS 中创建工程，把上一步创建的目录拷贝到新创建的工程中
* 添加 include 到`VC++目录->包含目录`, 添加 lib 到`VC++目录->库目录` 
* 添加 `libtiff.lib` 和 `libtiff_i.lib` 到`linker->输入->附加依赖项`中

```cpp
#include <stdlib.h>
#include "tiffio.h"

int _tmain(int argc, _TCHAR* argv[])
{
    TIFF* tiff = NULL;
    int width, height;

    tiff = TIFFOpen("1.tif","r");

    if(tiff !=NULL) {
        TIFFGetField(tiff, TIFFTAG_IMAGEWIDTH, &width);
        TIFFGetField(tiff, TIFFTAG_IMAGELENGTH, &height);

        printf("宽度：%d, 高度：%d\n", width, height);
    }
    system("pause");	
    return 0;
}
```

#### 问题
 连接时出现：
> LINK : warning LNK4098: 默认库“MSVCRT”与其他库的使用冲突；请使用 /NODEFAULTLIB:library

解决方法：
> 添加 `msvcrt.lib` 到`linker->输入->忽略特定默认库`中，重新编译
