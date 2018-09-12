---
title: 使用selenium模拟浏览器获取网页数据
tags: [python,selenium]
date: 2017-12-30
categories: Python
---


#### ChromeDriver资源

[chromedriver与chrome版本映射表](http://blog.csdn.net/huilan_same/article/details/51896672)

[chromedriver下载地址](http://chromedriver.storage.googleapis.com/index.html)

#### 代码说明

1. 通过百度搜索关键词，取出搜索结果
2. 通过百度搜索图片

#### 代码
``` python
# -*- coding=utf-8 -*-
import time

import sys

import win32gui

import win32con
from selenium import webdriver

reload(sys)
sys.setdefaultencoding('utf-8')

class ChromeDriver:
    def __init__(self, chrome_driver_path):
        self.chrome_driver = None
        self.chrome_driver_path = chrome_driver_path

    def start(self):
        print "启动chrome"
        # 启动chrome
        self.chrome_driver = webdriver.Chrome(self.chrome_driver_path)

    def stop(self):
        print "关闭chrome"
        # 关闭chrome
        if self.chrome_driver is not None:
            self.chrome_driver.quit()

    def open_web_page(self, url):
        print "打开网页 [%s]" % url
        # 模拟打开网页
        self.chrome_driver.get(url)
        time.sleep(2)

    def search_pic_for_baidu(self, pic_path):
        """
        使用baidu查询图片数据
        :param pic_path:
        :return:
        """
        self.open_web_page("https://www.baidu.com/")

        # 1.打开上传对话框
        self.chrome_driver.find_element_by_class_name("soutu-btn").click()
        time.sleep(1)
        self.chrome_driver.find_element_by_class_name("upload-pic").click()
        time.sleep(2)

        # 2.提交图片
        dialog = win32gui.FindWindow('#32770', u'打开')
        comboBoxEx32 = win32gui.FindWindowEx(dialog, 0, 'ComboBoxEx32', None)
        comboBox = win32gui.FindWindowEx(comboBoxEx32, 0, 'ComboBox', None)

        # 上面三句依次寻找对象，直到找到输入框Edit对象的句柄
        edit = win32gui.FindWindowEx(comboBox, 0, 'Edit', None)

        # 确定按钮Button
        button = win32gui.FindWindowEx(dialog, 0, 'Button', None)
        time.sleep(3)

        # 往输入框输入绝对地址
        win32gui.SendMessage(edit, win32con.WM_SETTEXT, None, pic_path)

        # 按button
        win32gui.SendMessage(dialog, win32con.WM_COMMAND, 1, button)
        time.sleep(1)

    # 使用百度查找内容
    def search_key_for_baidu(self, content, page_count):
        """
        使用baidu查找内容, 最多获取 page_count页
        :param content:
        :param page_count:
        :return:
        """
        self.chrome_driver.find_element_by_id("kw").clear()
        time.sleep(1)

        self.chrome_driver.find_element_by_id("kw").send_keys(content)
        self.chrome_driver.find_element_by_id("su").click()
        time.sleep(3)

        return self._get_key_search_data(page_count)

    # 获取搜索出的内容
    def _get_key_search_data(self, page_count):
        # 打开页数
        if page_count == 0:
            return

        result_list = self.chrome_driver.find_elements_by_class_name("result")
        for d in result_list:
            if d is not None:
                t = d.find_element_by_class_name("t")
                print t.text

        page = self.chrome_driver.find_element_by_id("page")
        if page is not None:
            _as = page.find_elements_by_tag_name("a")
            if len(_as) > 0:
                _next_url = _as[-1].get_attribute("href")
                page_count -= 1
                self.open_web_page(_next_url)
        else:
            page_count = 0

        # 递归调用
        self._get_key_search_data(page_count)


if __name__ == '__main__':
    driver_path = "chrome_driver路径"
    cd = ChromeDriver(driver_path)
    cd.start()

    # 使用baidu搜索关键词
    cd.open_web_page("https://www.baidu.com/")
    cd.search_key_for_baidu("python", 2)

    # 使用baidu搜索图片
    cd.search_pic_for_baidu("图片路径")

    # 休眠
    time.sleep(10)

    cd.stop()


```
