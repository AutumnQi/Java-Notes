# POM文件的结构

## 1.什么是POM文件

POM即`Project Object Model`是Maven的基础工作单元，通常以xml文件的格式出现。当项目启动时，Maven在POM文件中获取配置信息然后执行。

可以在POM中指定的项目依赖项，可以执行的插件或目标，构建配置文件等等；也可以指定其他信息，例如项目版本，描述，开发人员，邮件列表等。

## 2.POM文件的目录

### 2.1 最小的POM文件要求如下：

- `project` 根
- `modelVersion` -应该设置为4.0.0
- `groupId` -项目组的ID。
- `artifactId` -项目的ID。
- `version` -指定组下的项目版本

一个POM要求配置其groupId，artifactId和version。这三个值构成项目的完全限定项目名称，即<groupId>：<artifactId>：<version>的形式。

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```

### 2.2 POM的继承

POM文件也可以进行继承，只需要在POM文件中的<parent></parent>中指定继承的父文件即可，若父文件在特定的位置，也可以通过<relativePath>元素指定父文件的相对路径。

子POM会继承父POM的以下内容：

- `dependencies`
- `developers and contributors`
- `plugin lists` (including reports)
- `plugin executions with matching ids`
- `plugin configuration`
- `resources`

标准的springboot项目的POM文件中，继承`spring-boot-starter-parent`的POM文件，其中指定了一些通用的核心依赖，主要包括对Maven扩展的管理包

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.3.4.RELEASE</version>
  <relativePath/> <!-- lookup parent from repository -->
</parent>
```

### 2.3 POM的聚合

聚合类似于继承，不同点在于

- 继承是在子POM中指定要继承的父POM
- 聚合是在父POM中指定其会作为Module出现的子POM

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
 <!--使用<modules>部分指定会作为module插入的子POM的相对路径-->
  <modules>
    <module>../my-module</module>
  </modules>
</project>
```

聚合相比较于继承，可以实现一个子POM同时继承多个父POM

当然，继承和聚合可以同时使用，只需要遵循以下的三个规则

- Specify in every child POM who their parent POM is.
- Change the parent POMs packaging to the value "pom" .
- Specify in the parent POM the directories of its modules (children POMs)

### 2.4 项目的插入和变量

在<properties>中指定的变量可以应用到之后的内容中，示例如下

```xml
<project>
  ...
  <properties>
    <mavenVersion>3.0</mavenVersion>
  </properties>
 
  <dependencies>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-artifact</artifactId>
      <version>${mavenVersion}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-core</artifactId>
      <version>${mavenVersion}</version>
    </dependency>
  </dependencies>
  ...
</project>
```



