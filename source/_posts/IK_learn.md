---
title:  IK中文分词的使用
---

### 下载 IKAnalyzer
[IKAnalyzer](http://pan.baidu.com/s/1c2HpMik) 提取密码: wuq4

压缩包内容：
- IKAnalyzer2012FF_u1.jar `(IKAnalyzer核心jar包)`
- IKAnalyzer.cfg.xml `(配置文件，可以在这里配置停词表和扩展词库)`
- IKAnalyzer中文分词器V2012_FF使用手册.pdf `(使用手册)`
- stopword.dic `(停词表, 过滤词)`

### 配置文件
配置是否加载，以及加载哪些扩展字典和停止词字典，配置文件和扩展文件都放到程序**输出路径**下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
<properties>  
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 
	<entry key="ext_dict">ext_1.dic;ext_2.dic;</entry> 
	-->
	<!--用户可以在这里配置自己的扩展停止词字典
	<entry key="ext_stopwords">stopword_1.dic;stopword_2.dic;</entry>
	--> 	
</properties>
```

### 单独使用IK进行分词

```java
package test;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.StringReader;

import org.wltea.analyzer.cfg.DefaultConfig;
import org.wltea.analyzer.core.IKSegmenter;
import org.wltea.analyzer.core.Lexeme;
import org.wltea.analyzer.dic.Dictionary;

public class IKTest {
	public static void main(String[] args) {
		// 载入字典
		Dictionary.initial(DefaultConfig.getInstance());

		String text = "公文编辑器是大数据的电子文档的2016里约奥运会";
		System.out.println("原文：" + text);
		System.out.println("------------------------------");
		
		// 创建分词对象  
		StringReader reader = new StringReader(text);
		
		//写
		OutputStreamWriter out = null;
		File f = new File("out.txt");
		
		IKSegmenter ik = new IKSegmenter(reader,true);//当为true时，分词器进行最大词长切分
		try {
			out = new OutputStreamWriter(new FileOutputStream(f), "UTF-8");
			
			Lexeme lexeme = null;
		    while((lexeme = ik.next())!=null) {
		        System.out.println(lexeme.getLexemeText());
		        out.write(lexeme.getLexemeText()+"\n");
		    }
		} catch (IOException e) {
		    e.printStackTrace();
		} finally{
		    reader.close();
		    try {
		    	if (out != null) {
		    		out.close();
		    	}
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}
```

#### 运行结果
#####  未使用扩展词典和停止词典
```
原文：公文编辑器是大数据的电子文档的2016里约奥运会
------------------------------
公文
编辑器
是
大
数据
的
电子
文档
的
2016
里约
奥运会
```

##### 使用扩展词典和停止词典
扩展词典和停用词典文件的编码必须是`UTF-8`

扩展词典内容：
> 2016里约奥运会
> 电子文档
> 大数据

停止词典内容：
> 是
> 的

```
加载扩展词典：ext_1.dic
加载扩展停止词典：stopword.dic
原文：公文编辑器是大数据的电子文档的2016里约奥运会
------------------------------
公文
编辑器
大数据
电子文档
2016里约奥运会
```