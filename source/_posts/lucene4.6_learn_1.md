---
title:  Lucene 4.6 实战 （1）创建索引
---

### 引言
前一段时间做了一些和Lucene相关的工作，最近想把当初的学习过程，以及编写的Demo整理了一下。

### 搜索应用程序和 Lucene 之间的关系
![PIC](https://www.ibm.com/developerworks/cn/java/j-lo-lucene1/fig001.jpg)

### 建立索引
建立索引主要涉及以下这几个类:
#### Document
`Document`是用来描述文档的，这里的文档可以指一封电子邮件，或者是一个文本文件。一个`Document`对象由多个`Field`对象组成的。可以把一个`Document`对象想象成数据库中的一个记录，而每个`Field`对象就是记录的一个字段。
#### Field
`Field`对象是用来描述一个文档的某个属性的，比如一封电子邮件的标题和内容可以用两个`Field`对象分别描述。
#### Analyzer
在一个文档被索引之前，首先需要对文档内容进行分词处理，这部分工作就是`Analyzer`来做的。`Analyzer`类是一个抽象类，它有多个实现。针对不同的语言和应用需要选择适合的`Analyzer`。**中文分词我们选择了IKAnalyzer **。`Analyzer`把分词后的内容交给`IndexWriter`来建立索引。
#### IndexWriter
`IndexWriter`是 Lucene 用来创建索引的一个核心的类，他的作用是把一个个的 `Document`对象加到索引中来。
#### Directory
这个类代表了 Lucene 的索引的存储的位置，这是一个抽象类，它目前有两个实现，第一个是`FSDirectory`，它表示一个存储在文件系统中的索引的位置。第二个是`RAMDirectory`，它表示一个存储在内存当中的索引的位置。

#### 测试 Demo
数据简单封装:
```java
package test;

import java.util.Date;

public class DocumentData {
	public DocumentData() {
		id = 0;
		age = 0;
	}
	
	public DocumentData(int id, int age, String name, String desc, Date tm) {
		this.id = id;
		this.age = age;
		this.name = name;
		this.desc = desc;
		this.createTime = tm;
	}
	
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getDesc() {
		return desc;
	}
	public void setDesc(String desc) {
		this.desc = desc;
	}
	public Date getCreateTime() {
		return createTime;
	}
	public void setCreateTime(Date createTime) {
		this.createTime = createTime;
	}
	private int id;
	private int age;
	private String name;
	private String desc;
	private Date createTime;
}

```

创建数据索引:
```java
package test;

import test.DocumentData;

import java.io.IOException;
import java.util.Date;

import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.document.DateTools;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.IntField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.IndexWriterConfig.OpenMode;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.RAMDirectory;
import org.apache.lucene.util.Version;
import org.wltea.analyzer.lucene.IKAnalyzer;

public class IndexBuilder {
	public void Build() {
		String s1 = "张三在北京生活，在北京工作";
		String s2 = "李四是河北人，在北京工作";
		String s3 = "王五是山东人，在天津工作";
		String s4 = "赵六是河南人，在深圳工作";
		String s5 = "赵四是东北人，在深圳工作";
		
		//实例化IKAnalyzer分词器
		Analyzer analyzer = new IKAnalyzer();
		
		//内存中创建索引
		directory = new RAMDirectory();
		//创建索引保存到硬盘中
		//File dataPath = new File("E:\\java_workspace\\LuceneData\\data");
		//directory = FSDirectory.open(dataPath);
		
		IndexWriterConfig iwConfig = new IndexWriterConfig(Version.LUCENE_40, analyzer);
		iwConfig.setOpenMode(OpenMode.CREATE_OR_APPEND);
		
		IndexWriter iWriter = null;
		try {
			iWriter = new IndexWriter(directory, iwConfig);		
			//1
			DocumentData data1 = new DocumentData(0, 20, "张三", s1, new Date());
			Document doc_1 = GenDoc(data1);
			//2
			DocumentData data2 = new DocumentData(1, 19, "李四", s2, new Date());
			Document doc_2 = GenDoc(data2);
			//3
			DocumentData data3 = new DocumentData(2, 32, "王五", s3, new Date());
			Document doc_3 = GenDoc(data3);
			//4
			DocumentData data4 = new DocumentData(3, 44, "赵六", s4, new Date());
			Document doc_4 = GenDoc(data4);
			//5
			DocumentData data5 = new DocumentData(4, 28, "赵四", s5, new Date());
			Document doc_5 = GenDoc(data5);
			
			//添加
			iWriter.addDocument(doc_1);
			iWriter.addDocument(doc_2);
			iWriter.addDocument(doc_3);
			iWriter.addDocument(doc_4);
			iWriter.addDocument(doc_5);
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			try {
				if (iWriter != null) {
					iWriter.close();	
				}				
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	protected Document GenDoc(DocumentData data) {		
		String tm1 = DateTools.dateToString(data.getCreateTime(), DateTools.Resolution.MILLISECOND);	
		Document d = new Document();
		d.add(new Field("ID", new Integer(data.getId()).toString(), TextField.TYPE_STORED));
		d.add(new Field("name", data.getName(), TextField.TYPE_STORED));			
		d.add(new IntField("age", data.getAge(), Field.Store.YES));		
		d.add(new Field("time", tm1, TextField.TYPE_STORED));
		d.add(new Field("desc", data.getDesc(), TextField.TYPE_STORED));
		return d;
	}
	
	public Directory getDirectory() {
		return directory;
	}

	public void setDirectory(Directory directory) {
		this.directory = directory;
	}

	private Directory directory;
}
```
