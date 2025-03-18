---
title: lucene6.6.0学习心得
date: 2017-09-01 15:55:18
tags:
---

## lucene是什么？

官方网站：使用Java语言编写的高性能全文检索工程。适合任何需要全文检索的应用，特别是跨平台的。  
本菜鸟：空间换速度，查的快，毫秒级查询，秒杀like！

## 导入依赖

    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-core</artifactId>
        <version>6.6.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-analyzers-common</artifactId>
        <version>6.6.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-analyzers-smartcn</artifactId>
        <version>6.6.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-queryparser</artifactId>
        <version>6.6.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-highlighter</artifactId>
        <version>6.6.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-memory</artifactId>
        <version>6.6.0</version>
    </dependency>

## 创建索引

lucene将需要进行检索的内容经过指定的分词器（Analyzer）分词后存为索引（一般存在文件系统FSDirectory，也可以是内存RAMDirectory）。

    String indexPath = "D:/index"; 
    //得到索引存放目录对象
    Directory dir = FSDirectory.open(Paths.get(indexPath, new String[0]));
    //这个是官方的中文分词器，还有其他MMAnalyzer、IkAnalyzer等
    SmartChineseAnalyzer analyzer = new SmartChineseAnalyzer();
    //索引写入类需要的配置类
    IndexWriterConfig iwc = new IndexWriterConfig(analyzer);
    //索引写入类
    IndexWriter iw = new IndexWriter(dir, iwc);
    
    //User:{Integer id,String name,String sign}
    List<User> userList = new ArrayList<User>();
    userList.add(new User(1,"老大","我是我们家的大哥大！"));
    userList.add(new User(2,"老二","我是呆萌的二哈！"));
    userList.add(new User(3,"老三","我是专门拆散家庭人人讨厌的小三！"));
    userList.add(new User(4,"老四","我是小可爱！"));
    
    for(User user : userList){
        //document类似数据库的table
        Document doc = new Document();
        //Field类似数据库的column
        doc.add(new StoredField("id",user.getId()));
        doc.add(new StringField("name",user.getName(),Field.store.YES));
        doc.add(new TextField("sign",user.getSign(),Field.store.YES));
        iw.addDocument(doc);
    }
    
    //创建索引完成，关闭索引写入类
    iw.close();

#### 各种Field类

| 名称                          | 说明                                                                                                                                                |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| TextField                   | 分词并存入索引                                                                                                                                           |
| StringField                 | 逐字的存为索引                                                                                                                                           |
| IntPoint                    | Int类型存为索引以提供精确/范围查询                                                                                                                               |
| LongPoint                   | Long类型存为索引以提供精确/范围查询                                                                                                                              |
| DoublePoint                 | DoublePoint类型存为索引以提供精确/范围查询                                                                                                                       |
| SortedDocValuesField        | String类型存为索引以提供排序                                                                                                                                 |
| SortedSetDocValuesField     | String类型的set集合存为索引以提供排序                                                                                                                           |
| NumericDocValuesField       | Long类型存为索引以提供排序                                                                                                                                   |
| SortedNumericDocValuesField | Long类型的set集合存为索引以提供排序                                                                                                                             |
| StoredField                 | 只存储值，不创建索引。注：SortedDocValuesField、SortedSetDocValuesField、NumericDocValuesField、SortedNumericDocValuesField使用时若需要结果中显示排序值，则需要实例化同一个字段的StoredField |

## 查询索引

lucene读取索引，根据指定的Query，或者BooleanQuery(多条件)等查询出结果根据得分情况排序。

    //创建索引读取对象
    IndexReader reader = DirectoryReader.open(FSDirectory.open(Paths.get(luceneIndexPath, new String[0])));
    //创建索引查找对象
    IndexSearcher searcher = new IndexSearcher(reader);
    //创建基础查询单元
    Term term = new Term("sign","萌");
    //创建查询
    TermQuery query = new TermQuery(term);
    //查询 第二个参数代表查询得分高的前3个
    TopDocs search = searcher.search(query, 3);
    //得到命中
    ScoreDoc[] hits = search.scoreDocs;
    if(hits.length > 0){
        for(ScoreDoc sd : hits){
            Document doc = searcher.doc(sd.doc);
            System.out.println("id:"+ doc.get("id") +",name:"+ doc.get("name") +",sign:"+  doc.get("sign"));
        }
    }

#### 多条件查询

    //创建官方中文分词器
    SmartChineseAnalyzer analyzer = new SmartChineseAnalyzer();
    //创建一个查询解析器
    QueryParser parser = new QueryParser("sign", analyzer);
    //解析传入的查询内容 例如我家 -> sign:我，sign:家，如果只是TermQuery则不进行分析解析sign:我家
    Query query = parser.parse("我家");
    //创建BooleanClause
    BooleanClause c1 = new BooleanClause(new TermQuery(new Term("name", "大")), Occur.SHOULD);
    BooleanClause c2 = new BooleanClause(query, Occur.SHOULD);
    //添加查询条件到查询器中
    BooleanQuery booleanQuery = new BooleanQuery.Builder().add(c1).add(c2).build();
    
    TopDocs search = searcher.search(booleanQuery, 3);

#### Occur类说明

| 名称             | 说明                 |
| -------------- | ------------------ |
| Occur.FILTER   | 类似MUST，除非这些条件不参与计分 |
| Occur.MUST     | 条件必须出现在匹配结果中       |
| Occur.MUST_NOT | 条件必须不出现在匹配结果中      |
| Occur.SHOULD   | 条件应该出现在结果中，但不是必须的  |

#### 排序

如果需要排序，则在创建索引的时候如下设置  

    //存入索引，但是并未存入值
    doc.add(new IntPoint("sort", 1));
    //添加排序
    doc.add(new NumericDocValuesField("sort", 1));
    //只存储值，不然结果集无法出现
    doc.add(new StoredField("sort", 1));  

查询时则  

    //排序的字段为sort，类型为int，false == asc,true == desc
    SortField sortField = new SortField("sort", SortField.Type.INT, false);
    Sort sort = new Sort(sortField);
    TopDocs results = searcher.search(query, 3, sort);

#### 高亮显示

    //创建一个高亮类，传入的SimpleHTMLFormatter默认为给匹配处添加<B></B>标签，query就是searcher.search(query, 3)中的同一个query
    //另一个构造器Highlighter("<i>","<i/>");
    Highlighter highlighter = new Highlighter(new SimpleHTMLFormatter(),new QueryScorer(query));
    //设置最多返回多少匹配片段
    highlighter.setTextFragmenter(new SimpleFragmenter(20));
    //得到高亮的串
    String sign = highlighter.getBestFragment(analyzer, "sign", doc.get("sign"));

以上是本小菜的学习总结，若有错误，请大佬指教！
