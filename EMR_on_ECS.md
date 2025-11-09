# EMR on ECS

EMR on ECS是指在ECS上运行EMR，它将EMR的大数据处理能力与ECS的弹性灵活优势相结合，使得您能够更加便捷地配置和管理EMR集群，同时支持多种开源和自研大数据组件，适用于复杂的大数据处理和分析场景。



EMR主要由四部分组成，分别为集成的阿里云产品、开源组件、自研组件和集群管理。

![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/2224622471/CAEQVRiBgMCYsOC5rBkiIGJhODFmMzUxOWIyYzRiYzE5YjdiZDVmZjMxZWEwMGE04886482_20250115111141.016.svg)



- 集成的阿里云产品
  - EMR可以部署在阿里云ECS上。
  - 数据可以存储在阿里云OSS上。
  - EMR与DataWorks集成，您可以在DataWorks上使用EMR作为作业计算和数据存储引擎。
  - EMR Workflow提供全托管的工作流和任务调度服务。





- 自研组件

  为了让开源大数据组件和服务更好地运行在阿里云基础设施上，EMR提供的自研组件如下：

  - OSS-HDFS：兼容Hadoop分布式文件系统接口的对象存储解决方案，支持大数据计算任务通过标准HDFS协议直接访问阿里云OSS的数据。
  - JindoCache：分布式缓存解决方案，通过在内存中缓存数据块，提高数据读取性能并减少对底层存储系统的压力。
  - DLF-Auth：数据湖构建DLF产品提供，可以开启数据湖构建DLF的数据权限功能。













































