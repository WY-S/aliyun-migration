# demo
请根据购买的实例信息选择相应的demo验证。
```
├── README.md
├── beta
│   ├── README.md
│   ├── pom.xml
│   ├── run_consumer.sh
│   ├── run_producer.sh
│   └── src
├── vpc
│   ├── README.md
│   ├── pom.xml
│   ├── build.sh
│   └── src
└── vpc-ssl
    ├── README.md
    ├── pom.xml
    ├── run_consumer.sh
    ├── run_producer.sh
    └── src
```

## beta
公测实例使用demo，废弃。

## vpc
商业化vpc实例使用默认接入点接入demo。

## vpc-ssl
商业化公网实例使用ssl接入点接入demo。





主要看：vpc-ssl

注意：这个project适配jdk 1.8，其他版本可能有兼容性错误。



配置文件已经改好。kafka.properties、kafka_client_jaas.conf



[使用Java SDK在VPC内通过默认接入点收发消息-云消息队列 Kafka 版-阿里云](https://help.aliyun.com/zh/apsaramq-for-kafka/cloud-message-queue-for-kafka/getting-started/use-the-default-endpoint-to-send-and-receive-messages?spm=a2c4g.11186623.help-menu-68138.d_1_4_0.4c2e350fTHYV75&scm=20140722.H_99957._.OR_help-T_cn~zh-V_1)



遇到错误：

（1）java.lang.NoClassDefFoundError: com/fasterxml/jackson/databind/JsonNode

```java
缺少包。
    
在pom.xml里加上
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.15.0</version> <!-- Use a recent, stable version -->
        </dependency>
```



（2）SDK 24 does not support source version 1.6.

```java
版本不兼容。
    
    用JDK 1.8来运行代码
```



（3）Failed to construct kafka producer

```java
kafka构建失败。
    windows环境下，kafka.properties的路径要用双斜线。

ssl.truststore.location=D:\\apps\\java workspace\\java-codes\\kafka-java-demo\\vpc-ssl\\src\\main\\resources\\only.4096.client.truststore.jks
    
```

