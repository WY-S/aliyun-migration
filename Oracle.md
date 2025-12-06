# Oracle

Oracle数据库版本众多，主要从9i到12c（12.1, 12.2），再到近年来的“C”版本（18c, 19c, 21c, 23c等），差异主要体在于技术演进、云原生支持、性能提升、新功能和特性（如容器化、多租户、内存计算、自治数据库、JSON支持、AI/ML集成）等，尤其从12c开始引入CDB/PDB架构，对后续版本影响深远，而18c-21c是围绕云转型和数据库整合的过渡性版本，19c是长期支持版(LTS)，23c是最新应用开发版本。 

**从12c开始引入CDB/PDB架构**

**主要版本系列及差异**

1. **9i - 11g (经典时代)**
   - **9i**: 引入RAC（Real Application Clusters），是集群化应用的基石.
   - **10g**: 引入网格计算 (Grid Computing)，开始支持更广泛的跨服务器环境.
   - **11g (R1/R2)**: 关键版本，大量功能增强，如分区表、闪回查询、自动存储管理 (ASM)，是中小型企业的重要选择.
2. **12c (云时代开启)**
   - **12c (12.1/12.2)**: **最重大变革**，引入**多租户（Multitenant）架构**，一个数据库实例可承载多个租户 (PDB)，极大提升资源利用率.
3. **18c - 21c (云创新与过渡)**
   - **命名变化**: 从 12cR2 后，Oracle 采用年度发布（如18c, 19c），更贴近云环境.
   - **18c**: 云版，主要提供快速部署和多租户增强.
   - **19c**: **长期支持版本 (LTS)**，稳定性极高，是很多企业首选的稳定平台.
   - **21c**: 引入了**Autonomous Database (自治数据库)**概念，支持更多数据类型 (JSON).
4. **23c (应用开发与AI时代)**
   - **23c (最新应用开发版本)**: 重点在**应用开发（AppDev）**，支持**JSON Relational Duality (JSON关系对偶)**（一套数据用SQL和JSON访问），集成AI/ML、图数据库功能. 



**主要差异总结**

- **架构**: 从单实例到集群 (RAC)，再到多租户 (CDB/PDB).
- **云原生**: 12c后越来越重视云特性、PaaS服务、自治管理.
- **性能**: 内存计算 (In-Memory)、RAC、分区等技术不断进化.
- **数据管理**: 对JSON、图、混合数据类型支持增强.
- **AI/ML**: 23c开始更深度集成AI能力.
- **维护**: LTS版本（如19c）提供更长生命周期支持. 





**如何看版本**
在命令行执行 `SELECT * FROM v$version;` 即可查看当前数据库版本. 









# Oracle SQL





```sql
# 查看数据库版本
SELECT * FROM v$version;

# 
SELECT log_mode FROM v$database;

ALTER DATABASE ARCHIVELOG;

SHUTDOWN IMMEDIATE;

STARTUP MOUNT;


shutdown immediate;
startup mount;
alter database archivelog;
alter database open;
archive log list;


select SUPPLEMENTAL_LOG_DATA_MIN,
SUPPLEMENTAL_LOG_DATA_PK,
SUPPLEMENTAL_LOG_DATA_UI,
SUPPLEMENTAL_LOG_DATA_FK,
SUPPLEMENTAL_LOG_DATA_ALL,
SUPPLEMENTAL_LOG_DATA_PL from v$database;


ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;

alter database add supplemental log data (primary key,unique index) columns;



SQL> select instance_name from v$instance;
INSTANCE_NAME
----------------
ORCLCDB



SELECT username FROM all_users;















```



































