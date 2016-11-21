---
layout: post
title: "mybatis生成器生成代码"
date: 2016-08-15 13:45
comments: true
categories: [java,mybatis]
---

这里记录官方的控制台生成方法。 

<!-- more -->

插件的下载

```bash
cd ~/software
mkdir mybatisGenerator
cd !$
git clone git@github.com:mybatis/generator.git
cd generator/core/mybatis-generator-core
mvn clean package
```

编译后在target目录生成了mybatis-generator-core-1.3.4-SNAPSHOT.jar 

接着还需要准备好mysql的java驱动包

```bash
cd ~/software
wget http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.39.tar.gz 
```

然后创建generatorConfig.xml（注：不一定非得像其他教程说的那样非得放到项目的resources目录下）,内容如下：

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
  <classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />

  <context id="DB2Tables" targetRuntime="MyBatis3">
    <jdbcConnection driverClass="COM.ibm.db2.jdbc.app.DB2Driver"
        connectionURL="jdbc:db2:TEST"
        userId="db2admin"
        password="db2admin">
    </jdbcConnection>

    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

    <javaModelGenerator targetPackage="test.model" targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

    <sqlMapGenerator targetPackage="test.xml"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

    <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

    <table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
      <property name="useActualColumnNames" value="true"/>
      <generatedKey column="ID" sqlStatement="DB2" identity="true" />
      <columnOverride column="DATE_FIELD" property="startDate" />
      <ignoreColumn column="FRED" />
      <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
    </table>

  </context>
</generatorConfiguration>
{% endcodeblock %}

接下来就是修改配置文件了
第一步先修改classPathEntry的值，把原来的/Program Files/IBM/SQLLIB/java/db2java.zip换成，现在mysql驱动:

```xml
<classPathEntry location="/home/evoup/software/mysql-connector-java-5.1.39.tar.gz" />
```
然后注意到context id="DB2Tables",这个如果是单个表的话不改也没什么问题，但是根据文档 

> A unique identifier for this context. This value will be used in some error messages.

只是一个上下文的标识，所以可以忽略按照默认就可以了，但是我还是把它改成了context id="context1" 
targetRuntime="Mybatis3"这也是一个默认的可选选项，所以可以直接去掉，所以这一段看起来是这样：

```xml
  <classPathEntry location="/home/evoup/software/mysql-connector-java-5.1.39.tar.gz" />
  <context id="context1" >
```

之后要替换掉jdbc驱动类

```xml
<jdbcConnection driverClass="COM.ibm.db2.jdbc.app.DB2Driver"
    connectionURL="jdbc:db2:TEST" 
    userId="db2admin"
    password="db2admin"> 
</jdbcConnection>
```
改成

```xml
<jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/myDb" userId="root" password="" />
```

再接下来的

```xml
<javaTypeResolver >                                                                                                                 
  <property name="forceBigDecimals" value="false" />                                                                                
</javaTypeResolver>       
```
官方是说

> This element specifies that we always want to use the java.math.BigDecimal type for DECIMAL and NUMERIC columns: 

是和数据库列映射到java的pojo的属性对应，可以指定成想要的类型，这里的DECIMAL和NUMERIC列想必是DB2数据库的字段类型了，这样就会强制映射成java.math.BigDecimal，我们用mysql，这个选项不得而知，应该也是同样会转换的。但我直接把这段给去了。

再下来的三段

```xml
    <javaModelGenerator targetPackage="test.model" targetProject="\MBGTestProject\src"> 
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>
    <sqlMapGenerator targetPackage="test.xml"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>
    <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="\MBGTestProject\src"> 
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>
```

其中

> The <javaModelGenerator> element is used to define properties of the Java model generator. The Java Model Generator builds primary key classes, record classes, and Query by Example classes that match the introspected table. This element is a required child element of the <context> element.

Model模型生成器,用来生成含有主键key的类，记录类 以及查询Example类。targetPackage是指定生成的model生成所在的包名，targetProject是指定在该项目下所在的路径。

> The <sqlMapGenerator> element is used to define properties of the SQL map generator. The SQL Map Generator builds a MyBatis/iBATIS formatted SQL map XML file for each introspected table.

sqlMapGenerator指Mapper映射文件生成所在的目录，为每一个数据库的表生成对应的SqlMap文件。

> The <javaClientGenerator> element is used to define properties of the Java client generator. The Java client Generator builds Java interfaces and classes that allow easy use of the generated Java model and XML map files 

javaClientGenerator客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码。

根据需要我把以上3个配置项改成了如下

```xml
<javaModelGenerator targetPackage="com.company.project.dao.model" targetProject="16-adsapi-facebook" />
<sqlMapGenerator targetPackage="com.company.project.dao.mapper" targetProject="16-adsapi-facebook" />
<javaClientGenerator targetPackage="com.company.project.dao.client" targetProject="16-adsapi-facebook" type="XMLMAPPER" />
```

最后一个配置

```xml
<table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
  <property name="useActualColumnNames" value="true"/>
  <generatedKey column="ID" sqlStatement="DB2" identity="true" />
  <columnOverride column="DATE_FIELD" property="startDate" />
  <ignoreColumn column="FRED" />
  <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
</table> 
```

参数比较多，我直接改成了,很清楚就是表映射为domain对象，对于mysql，schema这个参数可以省略的。

```xml
<table tableName="myTable" domainObjectName="DomainObjectName"/> 
```

于是最终的测试生成文件是这样的

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration >

    <classPathEntry location="/home/evoup/project/mybatisGen/mysql-connector-java-5.1.39/mysql-connector-java-5.1.39-bin.jar" />
    <context id="context1" >

        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://172.16.26.177:3306/adsapi" userId="dba" password="madsolution" />
        <javaModelGenerator targetPackage="com.madhouse.performad.adsapi.facebook.dao.model" targetProject="16-adsapi-facebook" />
        <sqlMapGenerator targetPackage="com.madhouse.performad.adsapi.facebook.dao.mapper" targetProject="16-adsapi-facebook" />
        <javaClientGenerator targetPackage="com.madhouse.performad.adsapi.facebook.dao.client" targetProject="/home/evoup/project/16-adsapi-facebook" type="XMLMAPPER" />
        <table schema="" tableName="facebook_ad" domainObjectName="FacebookAdModel"/>
 
    </context>
</generatorConfiguration>
```


生成
做了一个脚本

```bash
#!/bin/sh
MYBATIS_HOME=/home/evoup/project/mybatisGen/generator/core/mybatis-generator-core/target
#XML_HOME=/home/evoup/project/adsapi_3.0.19/adsapi-facebook/src/main/resources
XML_HOME=/home/evoup/project/adsapi_trunk/adsapi-facebook/src/main/resources
java -jar ${MYBATIS_HOME}/mybatis-generator-core-1.3.5-SNAPSHOT.jar -configfile ${XML_HOME}/generatorConfigFB.xml -overwrite
```
运行脚本就可以生成了，最后将生成以下文件，见下图：
![Alt text](/images/evoup/mbGenerateFiles.jpg)

最后补一个，xml文件还要放到spring项目的src/main/resource对应目录下，如果单单放在src/main/java对应目录下还是不行的

