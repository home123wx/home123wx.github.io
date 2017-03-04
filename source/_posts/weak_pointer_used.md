---
title: 一种解决指针无效引用的方法
tags: [c++,template,弱指针]
date: 2017-03-04
categories: 编程
---

#### 无效引用

* 当多个指针指向同一个对象，在其中一个指针被delete后
* 其他指向该对象的指针不知道，其指向的内存已经无效
* 当访问指针指向的数据时出现访问错误。

#### 项目优化的需求

* 避免项目中无效引用的使用
* 在尽量少的变更代码的条件下能够实现该优化
* 不影响正常的逻辑

#### 解决方案

我们通过对被管理对象加`标签`，以及使用`弱指针`来代替旧的指针的方式来实现。

![结构图](https://github.com/home123wx/picture/blob/master/memory/weak_pointer/weak_pointer_pic.png?raw=true) 

##### 标签
可以把标签理解成一个`书签`。在对象创建时，对象中带有一个没有写任何内容的空白`书签`，在对象指针赋值给`弱指针`时，会在`书签`中填充内容，内容为`对象的指针`，并在`弱指针`中保存`书签`指针。

被管理对象通过继承`标签指针`来具备`书签`功能。

##### 弱指针
在说`弱指针`之前，先描述下我们定义的`强指针`，只要`强指针`存在，其指向的内容一定存在。

这里的`弱指针`相对于`强指针`，`弱指针`存在，其指向的内容可以不存在。既在`弱指针`指向的内存被释放，我们能够及时知道当前指针已经无效。

把对指针的所有操作封装到`弱指针`类中，让其能够模拟指针，这样可以在旧代码不变动的条件下，也就是说使用者可以完全不知道`弱指针`的存在，任然像正常指针一样使用。

利用`弱指针`的这种特性，我们正好能够解决无效引用带来的问题，并且能够满足优化提到的三点需求。

***

##### 代码
```c++
#ifndef _WEAK_TAG_POINTER_H_
#define _WEAK_TAG_POINTER_H_

#include <stdio.h>
#include <assert.h>

template<typename T> class WeakPointer;

template<typename T>
class WeakTagPointer {
private:
    /**
     * @desc 标签 1. 被标识类继承标签指针
     *            2. 当被标识类构建对象时，标签指针中引用计数默认为1
     *            3. 只有当被标识对象释放，标签中的pinter指针会被置为NULL
     *            4. 当标签的引用计数为0时，会释放标签自身
     */
    struct WeakTag {
        WeakTag(T* p): pointer(p), count(1) { }

        /**
         * @desc 减少引用计数，没有被引用释放自身
         */
        void WeakRelease()
        {
            --count;
            if (count <= 0) {
                delete this;
            }
        }

        /**
         * @desc 增加引用计数
         */
        void WeakAddRef() { ++count; }

        /**
         * @desc 获取标签指针
         */
        T* Pointer() const { return pointer; }

        /**
         * @desc 设置标签指针
         */
        void SetPointer(T* pObj) { pointer = pObj; }

    private:
        T* pointer; //标签指针
        int count;  //引用计数
    };

public:
    WeakTagPointer() : m_pWeakTag(NULL) {}

    /**
     * @desc 当子类(及被标识对象)析构时, 触发父类析构。
     *       1. 被标识对象释放，pointer标识被置NULL
     *       2. 通知其他使用者，该指针已无效
     */
    virtual ~WeakTagPointer()
    {
        if (m_pWeakTag != NULL) {
            m_pWeakTag->SetPointer(NULL);
            m_pWeakTag->WeakRelease();
        }
    }

public:
    /**
     * @desc 获取标签指针
     *       1. 指针未空时创建并返回, this指针为被标识对象
     *       2. 每次调用该函数，引用计数加1
     */
    WeakTag* TagPointer()
    {
        if (m_pWeakTag == NULL) {
            m_pWeakTag = new WeakTag(static_cast<T*>(this));
        }

        if (m_pWeakTag != NULL) {
            m_pWeakTag->WeakAddRef();
        }
        return m_pWeakTag;
    }

private:
    template<typename T2> friend class WeakPointer;
    WeakTag* m_pWeakTag;
};

//////////////////////////////////////////////////////////////////////////

/**
 * WeakPointer类，模拟指针的各种操作
 */
template<typename T>
class WeakPointer {
    //typedef typename T::WeakTag WeakTagType;
    struct WeakTagType : public T::WeakTag {};

public:
    WeakPointer():m_pTag(NULL) {}
    ~WeakPointer()
    {
        Release();
    }

public:
    /**
     * @desc 获取标签指针
     */
    T* operator->()
    {
        return GetPointer();
    }

    /**
     * @desc 获取指针对应的引用数据
     */
    T& operator*()
    {
        T* p = GetPointer();
        assert(p != NULL);
        return *p;
    }

    /**
     * @desc bool重载
     */
    operator bool()
    {
        return IsValid();
    }

    //////////////////////////////////////////////////////////////////////////
    //等于
    //////////////////////////////////////////////////////////////////////////
    /**
     * @desc 对指定类型指针赋值
     */
    void operator=(T* pObj)
    {
        WeakTagType* pTag = NULL;
        if (pObj != NULL) {
            //获取对象中标识
            WeakTagType* pTag = static_cast<WeakTagType*>(pObj->TagPointer());
            if (pTag != NULL) {
                //释放自身标识数据
                Release();
            }
        }
        //赋值
        m_pTag = pTag;
    }

    /**
     * @desc 同类型pointer的赋值
     * @param [in] pObj 同类型弱指针
     */
    WeakPointer& operator=(WeakPointer& obj)
    {
        if (this != &obj) {
            operator=(obj.GetPointer());
        }
        return *this;
    }

    /**
     * @desc 不同类型pointer的赋值
     * @param [in] pObj 其他类型弱指针
     */
    template<typename T2>
    void operator=(WeakPointer<T2>& obj)
    {
        operator=(obj.GetPointer());
    }

    //////////////////////////////////////////////////////////////////////////
    //等于等于
    //////////////////////////////////////////////////////////////////////////

    /**
     * @desc 判断同类型弱指针是否相同
     */
    bool operator==(WeakPointer& obj)
    {
        return IsEqual(obj);
    }

    /**
     * @desc 判断和表示类型的指针是否相同
     */
    bool operator==(T* pObj)
    {
        return IsEqual(pObj);
    }

    /**
     * @desc 判断和表示类型的指针是否相同, 特例NULL
     */
    bool operator==(const int&)
    {
        return !IsValid();
    }

    /**
     * @desc 判断和其他类型的弱指针是否不同
     */
    template<typename T2>
    bool operator==(WeakPointer<T2>& obj)
    {
        return IsEqual(obj);
    }

    //////////////////////////////////////////////////////////////////////////
    //不等于
    //////////////////////////////////////////////////////////////////////////
    /**
     * @desc 判断同类型弱指针是否不同
     */
    bool operator!=(WeakPointer& obj)
    {
        return !IsEqual(obj);
    }

    /**
     * @desc 判断和指针是否不同, 特例NULL
     */
    bool operator!=(const int&)
    {
        return IsValid();
    }

    /**
     * @desc 判断和表示类型的指针是否不同
     */
    bool operator!=(T* pObj)
    {
        return !IsEqual(pObj);
    }

    //////////////////////////////////////////////////////////////////////////
    //
    //////////////////////////////////////////////////////////////////////////
    /**
     * @desc 获取标签指针
     */
    T* GetPointer()
    {
        T* p = NULL;
        if (m_pTag != NULL) {
            p = static_cast<T*>(m_pTag->Pointer());
        }
        return p;
    }

    /**
     * @desc 判断标签的有效性
     * @return 是否有效
     */
    bool IsValid() const
    {
        return (m_pTag != NULL) && (m_pTag->Pointer() != NULL);
    }

private:
    WeakPointer(const WeakPointer&);

    /**
     * @desc 判断两个同类型弱指针是否相同
     */
    bool IsEqual(WeakPointer& obj)
    {
        return IsEqual(obj.m_pTag, obj.GetPointer());
    }

    /**
     * @desc 判断两个弱指针是否相同
     */
    template<typename T2>
    bool IsEqual(WeakPointer<T2>& obj)
    {
        return false;
    }

    bool IsEqual(WeakTagType* pTag, T* pObj)
    {
        bool b = false;
        if (m_pTag == NULL && pTag == NULL) {
            b = true;
        } else if (m_pTag != NULL && pTag != NULL) {
            bool b0 = (m_pTag == pTag);
            bool b1 = (m_pTag->Pointer() == pObj);
            b = (b0 && b1);
        }
        return b;
    }

    //
    bool IsEqual(T* pObj)
    {
        bool b = false;
        if (m_pTag != NULL && pObj != NULL) {
            b = (m_pTag->Pointer() == pObj);
        }
        return b;
    }

    /**
     * 释放标签数据
     */
    void Release()
    {
        if (m_pTag != NULL) {
            m_pTag->WeakRelease();
        }
    }

private:
    template<typename T2> friend class WeakPointer;
    WeakTagType* m_pTag;
};

#endif //_WEAK_TAG_POINTER_H_

```

[完整测试代码](https://github.com/home123wx/utils/tree/master/memory)
