How Search Works
====================

在线部分
----------
1. front door
  * 前端接入
  * 灰度发布，ab测试
  * 查询意图分类，调用主索引服务，answer服务

2. 索引聚合
  * 索引顶层聚合器进行查询，计算相关性，排序，生成摘要，返回结果
  
3. index serve
  * 线上索引仅仅是爬取网页的一部分，爬取的部分仅仅是全网的一小部分
  * 索引根据热度是分级的
  * 某层次的索引本身可以根据流量有多份副本，每份本身分片存储在不同服务器上
  * 存储结构简单来讲是倒排表，查询时是多个关键字倒排表的交集，由于计算量巨大，所以会有多个关键字的联合缓存，第一次查询时取的一个关键词的所有文档检查其余关键词直到得到第一个页面内容(经黑盒测试，实际情况可能是如果检测到关键词毫无关联，可能放弃交集操作，直接丢弃部分关键词)

4. answer service
  * 与一般web应用无异


离线部分
----------
1. 爬虫抓取

2. 预处理，去重(simhash)，构建倒排索引，静态排序(如PageRank)，以及摘要需要的元数据等
  * 传统做法是MapReduce批处理，针对不同优先级/热度的内容分批构建多个pipeline，需要多个全网页面的拷贝(N=级数)和许多冗余计算(猜测)
  * Percolator咖啡因系统实现了实时更新 (随机访问是必须的，因为去重，计算rank，以及数据存储均需要随机访问)

3. 更新到线上索引
