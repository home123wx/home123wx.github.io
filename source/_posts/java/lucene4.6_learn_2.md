---
title:  Lucene 4.6 实战 （2）检索文档
tags: [java,lucene]
date: 2016-09-21
categories: JAVA
---

### 检索文档
利用 Lucene 进行搜索也是非常方便的。在上篇文档中介绍了如何建立索引，现在我们就要在这个索引上进行搜索以找到包含某个关键词或短语的文档。Lucene 提供了几个基础的类来完成这个过程，它们分别是 `IndexSearcher`, `Term`, `Query`, `TermQuery`, `TopDocs`, `ScoreDoc`. 下面我们分别介绍这几个类的功能。

#### Query
这是一个抽象类，他有多个实现，比如 `TermQuery`, `BooleanQuery`, `PrefixQuery`. 这个类的目的是把用户输入的查询字符串封装成 Lucene 能够识别的 `Query`。

#### Term
Term 是搜索的基本单位，一个`Term`对象有两个`String`类型的域组成。生成一个 Term 对象可以有如下一条语句来完成：`Term term = new Term(“fieldName”,”queryWord”);` 其中第一个参数代表了要在文档的哪一个`Field`上进行查找，第二个参数代表了要查询的关键词。

#### TermQuery
`TermQuery` 是抽象类 `Query` 的一个子类，它同时也是 Lucene 支持的最为基本的一个查询类。生成一个 `TermQuery` 对象由如下语句完成： `TermQuery termQuery = new TermQuery(new Term(“fieldName”,”queryWord”));`它的构造函数只接受一个参数，那就是一个 `Term` 对象。

#### IndexSearcher
`IndexSearcher` 是用来在建立好的索引上进行搜索的。它只能以只读的方式打开一个索引，所以可以有多个 `IndexSearcher` 的实例在一个索引上进行操作。

#### TopDocs
`TopDocs` 保存由`IndexSearcher.search()`方法返回的具有较高评分的顶部文档。

#### ScoreDoc
提供对 `TopDocs` 中每条搜索结果的访问接口

#### 测试 Demo

各种Field搜索事例：
```java
package test;

import java.io.IOException;

import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.index.Term;
import org.apache.lucene.queryparser.classic.MultiFieldQueryParser;
import org.apache.lucene.queryparser.classic.ParseException;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.BooleanQuery;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.NumericRangeQuery;
import org.apache.lucene.search.PhraseQuery;
import org.apache.lucene.search.PrefixQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.TermQuery;
import org.apache.lucene.search.TermRangeQuery;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.store.Directory;
import org.apache.lucene.util.Version;
import org.wltea.analyzer.lucene.IKAnalyzer;

public class Searcher {

	public void BuildSearcher(Directory directory) {
		try {
			this.reader = DirectoryReader.open(directory);
			this.searcher = new IndexSearcher(reader);
			this.analyzer = new IKAnalyzer(false);  //当为true时，分词器进行最大词长切分
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	public TopDocs Search(Query query, int tops) {
		TopDocs td = null;
		if (query == null || this.searcher == null) {
			return td;
		}
		
		try {
			td = this.searcher.search(query, tops);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return td;
	}
	
	public Document GetDocById(int id) {
		Document doc = null;
		if (this.searcher != null) {
			try {
				doc = this.searcher.doc(id);
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		return doc;
	}
	
	//对单个域构建查询语句  
	public Query BuildSingleFieldQuery(String field, String key) {
		QueryParser parse = new QueryParser(Version.LUCENE_40, field, analyzer);
		//parse.setDefaultOperator(QueryParser.OR_OPERATOR);
		//parse.setDefaultOperator(QueryParser.AND_OPERATOR);
		Query query = null;
		try {
			query = parse.parse(key);
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return query;
	}
	
	//对多个域创建查询语句
	public Query BuildMultiFieldQuery(String[] fields, String key) {
		MultiFieldQueryParser parse=new MultiFieldQueryParser(Version.LUCENE_40,fields,analyzer);
		Query query = null;		
		try {
			query = parse.parse(key);
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}		
		return query;
	}
	
	//词条搜索
	public Query BuildTermQuery(String field, String key) {
		Query query = null;
		query =  new TermQuery(new Term(field, key));
		return query;
	}
	
	//前缀搜索 
	public Query BuildPrefixQuery(String field, String key) {
		Query query = null;
		query = new PrefixQuery(new Term(field, key));
		return query;
	}
	
	//短语搜索
	public Query BuildPhraseQuery() {
		PhraseQuery  query = new PhraseQuery();
		//设置短语间允许的最大间隔  
		query.setSlop(2);
		query.add(new Term("desc", "河"));
		query.add(new Term("desc", "在"));
		return query;
	}
	
	//通配符搜索
	public Query BuildWildcardQuery() {
		Query query = null;
		return query;
	}
	
	//字符串范围搜索
	public Query BuildStringRangeQuery(String field, String beg, String end, 
	                                   boolean begInclusive, boolean endInclusive) {
		Query query = TermRangeQuery.newStringRange(field, beg, end, 
		                                            begInclusive, endInclusive);
		return query;
	}
	
	//integer范围搜索 
	public Query BuildIntRangeQuery(String field, int beg, int end, 
	                                boolean begInclusive, boolean endInclusive) {
		Query query = NumericRangeQuery.newIntRange(field, beg, end, 
		                                            begInclusive, endInclusive);
		return query;
	}
	
	//float范围搜索
	public Query BuildFloatRangeQuery(String field, float beg, float end,
	                                  boolean begInclusive, boolean endInclusive) {
		Query query = NumericRangeQuery.newFloatRange(field, beg, end, 
		                                              begInclusive, endInclusive);
		return query;
	}
	
	//double范围搜索  
	public Query BuildDoubleRangeQuery(String field, double beg, double end,
	                                   boolean begInclusive, boolean endInclusive) {
		Query query = NumericRangeQuery.newDoubleRange(field, beg, end, 
		                                               begInclusive, endInclusive);
		return query;
	}
	
	//BooleanQuery
	public Query BuildBooleanQuery() {		
		BooleanQuery query = new BooleanQuery();
//		query.add(query_1, BooleanClause.Occur.SHOULD);
//		query.add(query_2, BooleanClause.Occur.SHOULD);
//		return query;
		return query;
	}
	
	private Analyzer analyzer;
	private IndexReader reader;
	private IndexSearcher searcher;
}
```

测试程序入口：
```java
package test;

import java.util.List;

import test.IndexBuilder;
import test.Searcher;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.IndexableField;
import org.apache.lucene.search.BooleanClause;
import org.apache.lucene.search.BooleanQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.store.Directory;

public class IKAnalyzerDemo {
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		IndexBuilder builder = new IndexBuilder();
		builder.Build();
		Directory directory = builder.getDirectory();

		//搜索过程**********************************
		Searcher searcher = new Searcher();
		searcher.BuildSearcher(directory);
				
		Query query = null;
		TopDocs topDocs = null;
		String keyword = "张三北京";
		String[] keys = {"name", "desc"};
		
		//test-1		
		query = searcher.BuildSingleFieldQuery("name", keyword);
		//System.out.println(query.toString());		
		topDocs = searcher.Search(query, 5);
		Display(searcher, topDocs);
		System.out.println("-----------------------------------------------");
		
		//test-2
		query = searcher.BuildMultiFieldQuery(keys, "北京");
		//System.out.println(query.toString());
		topDocs = searcher.Search(query, 5);
		Display(searcher, topDocs);
		System.out.println("-----------------------------------------------");
		
		//test-3
		query = searcher.BuildTermQuery("name", "赵六");
		//System.out.println(query.toString());
		topDocs = searcher.Search(query, 5);
		Display(searcher, topDocs);
		System.out.println("-----------------------------------------------");
		
		//test-4
		query = searcher.BuildPrefixQuery("desc", "天津");
		topDocs = searcher.Search(query, 5);
		Display(searcher, topDocs);
		System.out.println("-----------------------------------------------");
		
		//test-5
//		query = searcher.BuildPhraseQuery();
//		System.out.println(query.toString());
//		topDocs = searcher.Search(query, 5);
//		Display(searcher, topDocs);
//		System.out.println("-----------------------------------------------");
		
		//test-6
		Query query1 = searcher.BuildIntRangeQuery("age", 19, 19, true, true);
		System.out.println(query1.toString());
		topDocs = searcher.Search(query1, 5);
		Display(searcher, topDocs);
		System.out.println("-----------------------------------------------");
		
		//test-7
		BooleanQuery bq = (BooleanQuery)searcher.BuildBooleanQuery();
		bq.add(query, BooleanClause.Occur.SHOULD);
		bq.add(query1, BooleanClause.Occur.SHOULD);
		topDocs = searcher.Search(bq, 5);
		Display(searcher, topDocs);
		System.out.println("-----------------------------------------------");
	}
	
	public static void Display(Searcher searcher, TopDocs topDocs) {
		System.out.println("命中：" + topDocs.totalHits);
		//输出结果
		ScoreDoc[] scoreDocs = topDocs.scoreDocs;
		for (int i = 0; i < topDocs.totalHits; i++) {
			Document targetDoc = searcher.GetDocById(scoreDocs[i].doc);
			//System.out.println("内容：" + targetDoc.toString());
			List<IndexableField> l = targetDoc.getFields();
			for (IndexableField item : l ) {
				System.out.print(item.name() + ":" +item.stringValue() + " | ");
			}
			System.out.println("score:" + scoreDocs[i].score);
		}
	}
}
```

#### 输出结果

```
加载扩展词典：ext_1.dic
加载扩展停止词典：stopword.dic
命中：1
ID:0 | name:张三 | age:20 | time:20160919012839498 | desc:张三在北京生活，在北京工作 | score:0.501279
-----------------------------------------------
命中：2
ID:0 | name:张三 | age:20 | time:20160919012839498 | desc:张三在北京生活，在北京工作 | score:0.3345567
ID:1 | name:李四 | age:19 | time:20160919012839594 | desc:李四是河北人，在北京工作 | score:0.23656732
-----------------------------------------------
命中：0
-----------------------------------------------
命中：1
ID:2 | name:王五 | age:32 | time:20160919012839595 | desc:王五是山东人，在天津工作 | score:1.0
-----------------------------------------------
age:[19 TO 19]
命中：1
ID:1 | name:李四 | age:19 | time:20160919012839594 | desc:李四是河北人，在北京工作 | score:1.0
-----------------------------------------------
命中：2
ID:1 | name:李四 | age:19 | time:20160919012839594 | desc:李四是河北人，在北京工作 | score:0.35355338
ID:2 | name:王五 | age:32 | time:20160919012839595 | desc:王五是山东人，在天津工作 | score:0.35355338
-----------------------------------------------
```
