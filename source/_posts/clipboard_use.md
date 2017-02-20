---
title:  Windows剪切板项目中遇到的问题
---

#### 操作描述
复制word内容，word会把多种类型数据写入剪切板中，我们获取的是剪切板中RTF类型的数据。

#### 问题
使用系统函数获取剪切板数据。

```c++
if (OpenClipboard(NULL)) 
{
	UINT cfRTF = RegisterClipboardFormat(CF_RTF);
	if (IsClipboardFormatAvailable(cfRTF)) 
	{
		HANDLE hRtfData = GetClipboardData(cfRTF);
		if (hRtfData != NULL) 
		{
			//pBuf为剪切板中获取的RTF数据
			char* pBuf = (char*)GlobalLock(hRtfData);
			GlobalUnlock(hRtfData);
		}
	}
	CloseClipboard();
}
```
以上代码在两台机器上的表现不同:
> 机器1：调用`GetClipboardData()`函数，返回`NULL`,`GetLastError()`返回`1418`

> 机器2：出现两种不同的情况
* 调用`GetClipboardData()`函数，返回`NULL`,`GetLastError()`返回`1460`
* 调用`GetClipboardData()`函数，返回正确的数据句柄

#### 1. ERROR_CLIPBOARD_NOT_OPEN (1418)
* 错误信息提示没有打开剪切板，但是可以看出代码中，打开了剪切板并判断了其中是否包含RTF数据，但是就是在以上两个函数都执行成功的前提下，调用`GetClipboardData()`返回`NULL`。

* 带着以上问题在网上查找相关资料，偶然发现有人说加断点的原因（由于获取剪切板数据错误，我在`OpenClipboard(NULL)`处加了断点），去掉断点后仍然返回`NULL`，但是错误码变成了`1460`，这样两个问题合并成了一个问题。

* 其实在**`机器2`**上我也添加了断点，但是没有出现`1418`的错误，直接返回的是`1460`的错误，这里我判断可能是和机器相关，没有继续深入的调查。

#### 2. ERROR_TIMEOUT (1460)
在分析这个问题前，先说下我在网上找的剪切板的的一种机制：**`延迟提交`**

> **延迟提交**，只需剪贴板拥有者进程在调用`SetClipboardData()`将数据句柄（第二个参数）设置为NULL即可。

> 延迟提交的拥有者进程需要做的主要工作是对`WM_RENDERFORMAT`、`WM_DESTORYCLIPBOARD`和`WM_RENDERALLFORMATS`等剪贴板延迟提交消息的处理。  
> 
> 当其他进程调用`GetClipboardData()`函数时，系统将会向延迟提交数据的剪贴板拥有者进程发送`WM_RENDERFORMAT`消息。

> 剪贴板拥有者进程在此消息的响应函数中应使用相应的格式和实际的数据句柄来调用`SetClipboardData()`函数，但无需再调用`OpenClipboard()`和`EmptyClipboard()`去打开和清空剪贴板了，在设置完数据也无须调用`CloseClipboard()`关闭剪贴板。

> 如果其他进程打开了剪贴板并且调用`EmptyClipboard()`函数去清空剪贴板的内容，接管剪贴板的拥有权时，系统将向延迟提交的剪贴板拥有者进程发送`WM_DESTROYCLIPBOARD`消息， 以通知该进程对剪贴板拥有权的丧失。而失去剪贴板拥有权的进程在收到该消息后则不会再向剪贴板提交数据。

> 另外，在延迟提交进程在提交完所有要提交的数据后也会收到此消息。

> 如果延迟提交剪贴板拥有者进程将要终止，系统将会为其发送一条`WM_RENDERALLFORMATS`消息，在响应函数中可自定义是否调用`SetClipboardData()`来设置数据。

Word在把数据放到剪切板中时使用了**`延迟提交`**技术。在我们编写的程序中调用`GetClipboardData()`，触发了word的`WM_RENDERFORMAT`消息，word把数据处理并写到剪切板中，最后返回给调用者。

>在word设置数据到剪切板，再到返回给调用者，中间使用的时间过长会出现`ERROR_TIMEOUT`错误

#### 分析结果：
* 由于机器配置不同，处理速度各异，机器配置好的处理速度快的，在系统设置的响应时间内，能完成数据设置到剪切板的，就能够正确的获取剪切板数据。
* 根据推测系统调用`GetClipboardData()`的响应时间有可能是可以设置的，目前在网络上还没有找到相应的设置方法。

