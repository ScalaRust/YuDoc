## 主流日志管理解决方案
* ELK stack
> Elasticsearch+ logstash + kibana

* Graylog
> elasticsearch + mongodb + graylog-server

## ELK Stack
最著名的开源日志管理解决方案是ELK Stack，之所以称为Stack，是因为它不是一个软件包，而是由同一个团队开发的开源工具组合：Elasticsearch、Logstash、Kibana、及周边工具。
* Elasticsearch：是一个非常强大和高度可伸缩的搜索引擎，可以存储大量数据并作为集群使用。主要存储收集来的日志，并根据设置的索引，进行日志检索。
* Logstash：具有许多功能的日志转发器。支持多种类型的输入、过滤和输出。此外，Logstash可以处理许多编解码器，例如Json。
Kibana：用户界面，可以查看日志条目、创建丰富的仪表盘。
ELK Stack的优点：
1.成名更早
2.知名度更高
3.灵活度高

## Graylog
Graylog是一个强大的日志平台。可以很容易对结构化和非结构化日志进行管理以及调试应用程序。依赖Elasticsearch和MongoDB。Graylog的主服务从客户端节点获取数据，同时还提供Web接口，方便用户可视化聚合来的日志。
Graylog的优点包括以下方面：
* 免费的开源工具
* 相比ELK更优秀的报警功能
* 更好的交互，通过跟踪Graylog收到的错误堆栈，工程师可以了解源代码中的上下文。这大量节省了排错的时间和精力
* 强大的搜索功能，支持TB级别的查询
* 有归档功能，超过30天的所有内容都可以存储在廉价存储中，在出现查询需求时，可以重新导入到Graylog

## 总结
两种解决方案在功能上非常相似，但仍有一些差异需要考虑。

* 定位： Graylog更为纯粹，定位为强大的日志解决方案，而ELK则是大数据解决方案。
* 日志收集： Graylog可以通过网络协议直接从应用程序接收结构化日志和标准syslog。相反，ELK是使用Logstash分析已收集的纯文本日志的解决方案，然后解析并将它们传递给ElasticSearch。
* 可用性：在ELK中，Kibana扮演仪表盘的角色并显示从Logstash收到的数据。Graylog在这点上更方便，因为它提供了单一应用程序解决方案（不包括ElasticSearch作为灵活的数据存储），具有几乎相同的功能。因此，部署所需的时间更短。与ELK相比，Graylog开箱即用，且具有出色的权限系统，而Kibana则不具备此功能。
* 界面： 作为Graylog具有直观的GUI，并提供警报、报告和自定义分析功能。最重要的是，它能在多个日志源和跨机房收集数TB的数据。