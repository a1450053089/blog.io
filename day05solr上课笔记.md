# 1、企业级搜索引擎
* 1、开发一个简单的搜索引擎的思路
	* web服务配置-------solrconfig.xml
	* 数据源的配置------schema.xml
* 2、搭建solr服务
	* 下载包，并解压
		* 解压时需要一个路径
	* 修改配置文件
		* 如果使用jetty部署服务，可以通过快捷方式 java -jar，新增一个项目时，只需要配置schema.xml
		* 如果使用tomcat
			* 拷贝一个war
			* ext扩展包拷贝到webapps/solr/lib/
			* 由于solrconfig中制定了一些其他类库，需要zaisolrconfig中修改
	* 启动
* 3、solrj客户端
	* httpsolrserver 增删改查
	* debug-->mutilValued---类型转换异常
* 4、扩展
	* solr核心功能：**发布web服务**，管理索引(管理、索引)
	* ikanalyzer分词器使用
	* dataimport的导入

# 2、开发一个简单的搜索引擎的思路
* solr是什么？
	* 基于lucene的完整的搜索引擎
* 自有的搜索引擎功能说明
	* 对外提供了一个action，
		* 创建索引(读取数据库，创建索引库)
		* 查询索引(根据关键词查询索引)
	* 针对特有应用场景-虎嗅 (id,title,author,createTime,content,url)
* solr的核心概念
	* 为了区分多个不同数据源的索引，我们为索引库赋予一个新的名字，core。
		* 虎嗅搜索的索引数据，需要一个索引库。一般指定的路径是index，这个index其实机器上的文件夹。
	* core下面有至少两个配置文件
		* 一个文件是用来描述，action相关配置的
		* 一个文件是用来配置不同的业务数据字段类型的
![](img/2017-09-11_085606.png)
			* 因为solr是通用的搜索引擎，不同的数据源的字段名称，字段类型，是否分词可能不尽一样，我们需要一个单独的配置文件针对每个数据源进行配置。

# 3、搭建solr的服务
本质上是发布一个网站
* 安装部署的步骤说明
	* 下载安装包、解压安装包、修改配置文件、启动服务

#### 下载安装包
![](img/2017-09-11_090101.png)

#### 解析安装包
* 在解压之前，先创建一个目录
	* E:\itcast\env
* 解压步骤
![](img/2017-09-11_090600.png)

#### 修改配置文件
* solr提供一个简单的demo，不需要修改，可以直接启动。
#### 启动
* solr安装包说明
![](img/2017-09-11_090915.png)
* 在example目录下，发现一个jar包。
* 启动start.jar包
![](img/2017-09-11_091254.png)
* 找到启动的端口号
![](img/2017-09-11_091522.png)
* 在本地的浏览器上访问solr
	* http://localhost:8983/solr/#/

# 4、solrWeb界面
* 首页的概述信息：主要介绍solr运行的硬件环境和JVM
![](img/2017-09-11_093456.png)
* coreAdmin:对多个core进行管理
![](img/2017-09-11_094050.png)
	* 为什么会有多个core？
		* 虎嗅数据 
		* 京东数据
	* 添加一个core
		* 复制默认的collection1，删除掉core.properties，删除掉readme
		* 在界面点击add按钮，输入信息。
* java properties界面
![](img/2017-09-11_095852.png)
* core selector 界面，主要操作单个索引
	* 创建索引
		* 文档的类型，json，xml，CSV,自主上传
	* 查询索引
		* 查询索引的参数，参见文档。

# 5、Solr的配置文件
![](img/2017-09-11_103421.png)
* solrconfing.xml
	* 设置solrweb网站的一些功能。
		* lucene的版本号
		* 依赖的外部的其他类库
		* 索引库的实现工厂类 NRT
		* 各种Handler(增删改查、管理命令、集群管理)
* schema.xml
![](img/2017-09-11_105547.png)
* 定义数据源文档中的字段，并设置字段的类型
	* 定义文档中字段
		* id
			* name="id",type="string",indexed="false",stored="false"
		* title
	* 字段类型
		* 主要拷贝默认的字段
		* 如何要配置ik分词器，就需要单独的写一个fieldtype。
* solr结构
![](img/2017-09-11_110618.png)
在实际的工作中，我们在使用solr时。大部分的情况只需要修改schema.xml文件。
修改schema.xml文件时，大部分的情况下，只需要拷贝一些默认的fieldType。除非有自己特定的需求，比如想使用ik分词器，就得自己写一个fieldType。

# 6、配置虎嗅搜索引擎
* 设置一个core
	* 复制collection1，删除core.properties
* 修改schema.xml文件
```shell
<?xml version="1.0" encoding="UTF-8" ?>
<schema name="example" version="1.5">
	<field name="_version_" type="long" indexed="true" stored="true"/>
	<field name="_root_" type="string" indexed="true" stored="false"/>
	
	<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
	
	 <field name="title" type="text_general" indexed="true" stored="true" multiValued="true"/>
	 
	 <field name="author" type="text_general" indexed="true" stored="true"/>
	 
	 <field name="url" type="text_general" indexed="true" stored="true"/>
	 
	 <field name="content" type="text_general" indexed="false" stored="true" multiValued="true"/>
	 
	<uniqueKey>id</uniqueKey>
	
	<fieldType name="string" class="solr.StrField" sortMissingLast="true" />
	 <fieldType name="long" class="solr.TrieLongField" precisionStep="0" positionIncrementGap="0"/>
	 <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
      <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <!-- in this example, we will only use synonyms at query time
        <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
        -->
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>
</schema> 	 
```

# 7、将solr部署到tomcat下(在企业中将tomcat部署linux)
* jetty和tomcat比较
		* jetty目前没有市场广泛使用
		* jetty和tomcat在某些应用场景下各有千秋
![](img/2017-09-11_112507.png)

* 解压tomcat并重命名
* 拷贝solr.war到tomcat下
![](img/2017-09-11_112807.png)
* 启动下tomcat，用tomcat自动解压下包，访问失败
* 将solr要用的扩展包拷贝到tomcat
	* tomcat lib---->所有的项目
	* tomcat/webapps/projectname/web-info/lib
![](img/2017-09-11_113518.png)
* 找不到配置文件
![](img/2017-09-11_114227.png)
	* 单独创建一个solrhome，将索引库拷贝到solrhome下
	* 启动程序的时候，指定solrhome的信息，修改web.xml
```
E:\itcast\env\tomcat4solr\apache-tomcat-8.0.24\webapps\solr\WEB-INF
``` 　
![](img/2017-09-11_120141.png)
		* lib地址配置的路径设置，是以core所在的目录为基准去寻找。 ../ 表示上一层的目录。

# 8、使用java API操作solr
远程操作，创建索引和查询索引

#### 1、创建maven项目,导入solr的依赖
```
  		<dependency>
            <groupId>org.apache.solr</groupId>
            <artifactId>solr-solrj</artifactId>
            <version>4.10.2</version>
        </dependency>

 <!-- 由于solr的上级项目将 commons-logging排除了，需要单独的引用-->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
```

#### 2、编写测试代码
```shell
 /**
     * 创建索引
     * @throws SolrServerException
     * @throws IOException
     */
    private static void createIndex() throws SolrServerException, IOException {
        //1、连接solr，由于solr是一个web服务。
        //  http://127.0.0.1:8080/solr/ 使用默认collection1
        //  http://127.0.0.1:8080/solr/#/collection1 这种方式不行，去掉#就可以了
        String url = "http://127.0.0.1:8080/solr/collection1";
        SolrServer solrServer = new HttpSolrServer(url);
        //2、创建索引
        // 2.1、准备一个document对象
        SolrInputDocument doc = new SolrInputDocument();
        doc.addField("id","22222");
        doc.addField("title","我爱我的祖国");
        doc.addField("author","传智播客");
        doc.addField("content","这里是地下城与勇士专区 您的问题可能不属于这个分类下的..可能无法及时得到满意回答 建议您从新手动分类下. 这样才能快点得到精确的答案哦");
        doc.addField("url","https://www.itcast.cn/");
        // 2.2、调用方法添加索引
        solrServer.add(doc);
        //3、commit提交索引
        solrServer.commit();
    }
```

#### 高亮展示的原理
* 在索引库中搜索出与关键词匹配的数据
* 在结果数据中，进行高亮。
	* 再次进行分词，定位到关键词，然后加标签。
![](img/2017-09-11_164412.png)
* 高亮代码
```shell
 private static void queryAndHighLight() throws SolrServerException {
        //查询索引
        //1.solrserver  httpsolrserver
        HttpSolrServer httpSolrServer = new HttpSolrServer("http://127.0.0.1:8080/solr/collection1");
        //2.查询
        String query = "祖国";

        SolrQuery solrQuery = new SolrQuery(query);
        // 设置高亮的步骤
        // 1.开启
        solrQuery.setHighlight(true);
        // 2.设置摘要的长度
        solrQuery.setHighlightFragsize(100);
        // 3.添加html标签
        solrQuery.setHighlightSimplePre("<font color='red'>");
        solrQuery.setHighlightSimplePost("</font>");
        // 4. 指定哪些字段要添加高亮
        solrQuery.addHighlightField("title,content");
        QueryResponse res = httpSolrServer.query(solrQuery);
        //5、由于高亮的原理是基于已经查询出来的结果集，进行二次分词并添加html。
        // 所以在获取高亮的结果是，需要单独调用一个方法。
        Map<String, Map<String, List<String>>> highlighting = res.getHighlighting();
        for (Map.Entry<String, Map<String, List<String>>> map1 : highlighting.entrySet()) {
            System.out.println("map1-------------key:"+map1.getKey());
            Map<String, List<String>> map2 = map1.getValue();
            for (Map.Entry<String, List<String>> map2EntrySet : map2.entrySet()) {
                System.out.println(map2EntrySet.getKey());
                List<String> list = map2EntrySet.getValue();
                System.out.println(list);
            }
            System.out.println("-----------------------------------------------------------------------");

        }
    }
```
* 高亮的结果
![](img/2017-09-11_170843.png)


# 8、扩展内容
#### solr总结
* solr是一个企业级的搜索引擎
	* solr是基于lucene，在lucene的基础上做了一些封装。
	* 封装主要是**发布了一个web服务**
	* 其次才是对数据源做了通用的适配
* solr为什么要发布一个web服务？
	* 发布web服务的功能有，远程操作lucene索引库。
	* 由于lucene是一套类库，在创建索引时，需要指定当前服务器上的一个固定的目录作为索引库。如果业务是一个分布式的系统，就没有办法将数据实时创建索引，如果分布式系统中每个节点自己维护一套索引库，意义就不大了。
![](img/2017-09-11_194855.png)

#### lucene中有ik分词器，solr如何配置？
* 修改schema.xml的fieldType设置成ikanalyzer
* 需要单独导入ikanalyzer的jar
	* tomcat -->lib
	* tomcat -->webapps ->project-->web-info-->lib
* 拷贝ikanalyzer.jar
* 修改配置文件

#### 设置datahandler
* 拷贝jar包
* 创建一个配置文件 data-config.xml
```shell
<dataConfig>
  <dataSource type="JdbcDataSource"
              driver="com.mysql.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/spider?characterEncoding=UTF-8"
              user="root"
              password="root"/>
  <document>
    <entity name="article"
            query="select id,title,content,authorfrom article">
        <field column = "id" name="id"/>        
        <field column = "title" name = "title" />
        <field column = "content" name="author" />
		<field column = "author" name="author" />
    </entity>
  </document>
</dataConfig>
```
* 修改solrconfig.xml
```shell
  <requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
      <lst name="defaults">
         <str name="config">data-config.xml</str>
      </lst>
  </requestHandler>
```
* 在浏览器上刷新 
http://127.0.0.1:8080/solr/#/huxiu_article/dataimport//dataimport


#### 思考题：
在海量数据下，创建索引速度非常慢？如何提高创建索引的速度？