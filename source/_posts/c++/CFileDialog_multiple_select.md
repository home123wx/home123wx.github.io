---
title: CFileDialog路径缓存的扩充
tags: [c++,windows,mfc]
date: 2017-03-25
categories: [C++,MFC]
---

#### 问题描述
`CFileDialog`开启多选后，能够选中的文件个数有限。

#### 问题原因
`CFileDialog`中默认的`Buffer`空间不足以存放所选中的文件路径字符串集合，所以字符串内容会被截断，保存的信息不够完整。

>对话框中，在选中文件后，会有一个`Buffer`来保存所有选中文件的字符串，还有一个变量保存`Buffer`的长度。这些信息都保存在`OPENFILENAME`结构中。
>缓存首地址： `m_ofn.lpstrFile`
>缓存长度：`m_ofn.nMaxFile`

#### 解决方案
1. 每次申请足够大的缓存来存放路径字符串
2. 根据选中的内容，动态计算路径字符串的长度，大于默认值后，再手动申请缓存空间

##### 方案1 -- 定长缓存
```cpp
const int M = 1024 * 1024;

//构建对话框
CFileDialog dlgFile(TRUE, NULL, NULL,OFN_ALLOWMULTISELECT |
                    OFN_HIDEREADONLY | OFN_NODEREFERENCELINKS |
                    OFN_EXPLORER, szFilter, NULL, 0, FALSE);
	                      
//定义缓存大小，临时区域
int nMax = 10 * M;
fileDlg.m_ofn.nMaxFile = nMax;
fileDlg.m_ofn.lpstrFile = new TCHAR[nMax];

//处理路径
if (fd.DoModal() == IDOK) {
	POSITION pos = fileDlg.GetStartPosition(); 
	while (pos != NULL) {
		CString path = fileDlg.GetNextPathName(pos);
		//DoSomething
	}
}

//释放资源
if (fileDlg.m_ofn.lpstrFile != NULL) {
	delete[] fileDlg.m_ofn.lpstrFile;
	fileDlg.m_ofn.lpstrFile = NULL;
}
```

##### 方案2 -- 动态缓存
使用时，Win7以上系统，构造函数参数 `bVistaStyle` 赋值为`FALSE`。

``` cpp
CMutipleSelFileDialog dlgFile(TRUE, NULL, NULL,
                            OFN_ALLOWMULTISELECT | OFN_HIDEREADONLY|
                            OFN_NODEREFERENCELINKS | OFN_EXPLORER,
                            szFilter, NULL, 0, FALSE);
```

*头文件*
```cpp
#ifndef _MULTIPLE_SEL_FILE_DIALOG_H_
#define _MULTIPLE_SEL_FILE_DIALOG_H_

#include <string>
using std::string;

class CMutipleSelFileDialog : public CFileDialog {
    DECLARE_DYNAMIC(CMutipleSelFileDialog)

public:
    CMutipleSelFileDialog(BOOL bOpenFileDialog, // TRUE for FileOpen, FALSE for FileSaveAs
                          LPCTSTR lpszDefExt = NULL,
                          LPCTSTR lpszFileName = NULL,
                          DWORD dwFlags = OFN_HIDEREADONLY | OFN_OVERWRITEPROMPT,
                          LPCTSTR lpszFilter = NULL,
                          CWnd* pParentWnd = NULL,
                          DWORD dwSize = 0,
                          BOOL bVistaStyle = TRUE
                          );
    virtual ~CMutipleSelFileDialog();

public:
    virtual int DoModal();
    CString GetNextPathName(POSITION& pos) const;
    POSITION GetStartPosition();
    void SetTitleName(const string& name);

    DECLARE_MESSAGE_MAP()

private:
    virtual void OnFileNameChange();
    //BOOL IsWin7_or_Later();
    void Release();

private:
    bool   m_bParsed;
    TCHAR* m_pFolder;
    TCHAR* m_pFiles;
};

#endif //_MULTIPLE_SEL_FILE_DIALOG_H_
```

*实现文件*
```cpp
#include "stdafx.h"
#include "MultipleSelFileDialog.h"
#include <cderr.h>

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

IMPLEMENT_DYNAMIC(CMutipleSelFileDialog, CFileDialog)

BEGIN_MESSAGE_MAP(CMutipleSelFileDialog, CFileDialog)
//{{AFX_MSG_MAP(CFECFileDialog)
//}}AFX_MSG_MAP
END_MESSAGE_MAP()

CMutipleSelFileDialog::CMutipleSelFileDialog(BOOL bOpenFileDialog, LPCTSTR lpszDefExt,
                                             LPCTSTR lpszFileName, DWORD dwFlags,
                                             LPCTSTR lpszFilter, CWnd* pParentWnd,
                                             DWORD dwSize, BOOL bVistaStyle):
            CFileDialog(bOpenFileDialog, lpszDefExt, lpszFileName,
                        dwFlags, lpszFilter, pParentWnd, dwSize, bVistaStyle)
{
    m_pFiles  = NULL;
    m_pFolder = NULL;
    m_bParsed = false;
}

CMutipleSelFileDialog::~CMutipleSelFileDialog()
{
    Release();

    if (m_ofn.lpstrTitle != NULL) {
        delete[] m_ofn.lpstrTitle;
        m_ofn.lpstrTitle = NULL;
    }
}

int CMutipleSelFileDialog::DoModal()
{
    Release();

    int ret = CFileDialog::DoModal();
    if (ret == IDCANCEL) {
        DWORD err = CommDlgExtendedError();
        if (err == FNERR_BUFFERTOOSMALL/*0x3003*/ && m_pFiles != NULL) {
            ret = IDOK;
        }
    }
	return ret;
}

CString CMutipleSelFileDialog::GetNextPathName(POSITION& pos) const
{
    if (m_pFiles == NULL) {
        return CFileDialog::GetNextPathName(pos);
    }

    ASSERT(pos);    
    TCHAR *ptr = (TCHAR *)pos;

    CString ret = m_pFolder;
    ret += _T("\\");
    ret += ptr;

    ptr += _tcslen(ptr) + 1;
    if (*ptr) {
        pos = (POSITION)ptr;
    } else {
        pos = NULL;
    }
    return ret;
}

POSITION CMutipleSelFileDialog::GetStartPosition()
{
    if (m_pFiles == NULL) {
        return CFileDialog::GetStartPosition();
    }

    if (!m_bParsed)
    {
        CString temp = m_pFiles;
        temp.Replace(_T("\" \""), _T("\""));
        temp.Delete(0, 1);                      // remove leading quote mark
        temp.Delete(temp.GetLength() - 1, 1);   // remove trailing space
    
        _tcscpy(m_pFiles, temp);
    
        TCHAR *ptr = m_pFiles;
        while (*ptr) {
            if ('\"' == *ptr) {
                *ptr = '\0';
            }
            ++ptr;
        }
        m_bParsed = TRUE;
    }
    return (POSITION)m_pFiles;
}

void CMutipleSelFileDialog::SetTitleName(const string& name)
{
    if (m_ofn.lpstrTitle != NULL) {
        delete[] m_ofn.lpstrTitle;
        m_ofn.lpstrTitle = NULL;
    }

    int nLen = name.size() + 1;
    char* pBuf = new char[nLen];
    memset(pBuf, 0, nLen);
    memcpy(pBuf, name.c_str(), nLen);
    m_ofn.lpstrTitle = pBuf;
}

void CMutipleSelFileDialog::OnFileNameChange()
{
    TCHAR dummy_buffer;

	//Win7以上，构造函数参数 bVistaStyle 赋值为flase，否则GetParent()为NULL
	//导致无消息接受窗口，无法获取以下信息
    HWND hWnd = GetParent() == NULL ? m_hWnd : GetParent()->m_hWnd;
    
    // 所有选中文件名集合的的长度.如：("xxx1.dat" "xxx2.dat")
    // 每一个文件名包含在双引号中，文件名之间有空格符
    int nfiles = CommDlg_OpenSave_GetSpec(hWnd, &dummy_buffer, 1);

    // 去除文件名，文件目录的长度.如：("x:\xxx\xxx\")
    int nfolder = CommDlg_OpenSave_GetFolderPath(hWnd, &dummy_buffer, 1);

    // 选中内容长度超过了默认的缓存区长度
    if (nfiles + nfolder > m_ofn.nMaxFile) {
        m_bParsed = FALSE;
        Release();

        m_pFiles = new TCHAR[nfiles + 1];
        CommDlg_OpenSave_GetSpec(hWnd, m_pFiles, nfiles);

        m_pFolder = new TCHAR[nfolder + 1];
        CommDlg_OpenSave_GetFolderPath(hWnd, m_pFolder, nfolder);
    }
    else if (m_pFiles != NULL) {
        Release();
    }
    CFileDialog::OnFileNameChange();
}

void CMutipleSelFileDialog::Release()
{
    if (m_pFiles != NULL) {
        delete[] m_pFiles;
        m_pFiles = NULL;
    }

    if (m_pFolder != NULL) {
        delete[] m_pFolder;
        m_pFolder = NULL;
    }
}
```
