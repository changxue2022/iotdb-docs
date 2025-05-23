# 可视化控制台

可视化控制台的部署可参考文档 [可视化控制台部署](../Deployment-and-Maintenance/workbench-deployment_timecho.md) 章节。

## 第1章 产品介绍
IoTDB可视化控制台是在IoTDB企业版时序数据库基础上针对工业场景的实时数据收集、存储与分析一体化的数据管理场景开发的扩展组件，旨在为用户提供高效、可靠的实时数据存储和查询解决方案。它具有体量轻、性能高、易使用的特点，完美对接 Hadoop 与 Spark 生态，适用于工业物联网应用中海量时间序列数据高速写入和复杂分析查询的需求。

## 第2章 使用说明
IoTDB的可视化控制台包含以下功能模块：
| **功能模块** | **功能说明**                                             |
| ------------ | ------------------------------------------------------------ |
| 实例管理     | 支持对连接实例进行统一管理，支持创建、编辑和删除，同时可以可视化呈现多实例的关系，帮助客户更清晰的管理多数据库实例 |
| 首页         | 支持查看数据库实例中各节点的服务运行状态（如是否激活、是否运行、IP信息等），支持查看集群、ConfigNode、DataNode运行监控状态，对数据库运行健康度进行监控，判断实例是否有潜在运行问题 |
| 测点列表     | 支持直接查看实例中的测点信息，包括所在数据库信息（如数据库名称、数据保存时间、设备数量等），及测点信息（测点名称、数据类型、压缩编码等），同时支持单条或批量创建、导出、删除测点 |
| 数据模型     | 支持查看各层级从属关系，将层级模型直观展示                   |
| 数据查询     | 支持对常用数据查询场景提供界面式查询交互，并对查询数据进行批量导入、导出 |
| 统计查询     | 支持对常用数据统计场景提供界面式查询交互，如最大值、最小值、平均值、总和的结果输出。 |
| SQL操作      | 支持对数据库SQL进行界面式交互，单条或多条语句执行，结果的展示和导出 |
| 趋势         | 支持一键可视化查看数据整体趋势，对选中测点进行实时&历史数据绘制，观察测点实时&历史运行状态 |
| 分析         | 支持将数据通过不同的分析方式（如傅里叶变换等）进行可视化展示 |
| 视图         | 支持通过界面来查看视图名称、视图描述、结果测点以及表达式等信息，同时还可以通过界面交互快速的创建、编辑、删除视图 |
| 数据同步     | 支持对数据库间的数据同步任务进行直观创建、查看、管理，支持直接查看任务运行状态、同步数据和目标地址，还可以通过界面实时观察到同步状态的监控指标变化 |
| 权限管理     | 支持对权限进行界面管控，用于管理和控制数据库用户访问和操作数据库的权限 |
| 审计日志     | 支持对用户在数据库上的操作进行详细记录，包括DDL、DML和查询操作。帮助用户追踪和识别潜在的安全威胁、数据库错误和滥用行为 |

主要功能展示：
* 首页
![首页.png](/img/%E9%A6%96%E9%A1%B5.png)
* 测点列表
![测点列表.png](/img/workbench-1.png)
* 数据查询
![数据查询.png](/img/%E6%95%B0%E6%8D%AE%E6%9F%A5%E8%AF%A2.png)
* 趋势
![历史趋势.png](/img/%E5%8E%86%E5%8F%B2%E8%B6%8B%E5%8A%BF.png)