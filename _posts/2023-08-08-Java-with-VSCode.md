---
title: "Java with VSCode"
date:  2023-08-08 10:47:21 +0800
categories: [language]
tags: [vscode]
---

download java jdk

[适用于 Windows 的 Java 下载](https://www.java.com/zh-CN/download/windows_manual.jsp?locale=zh_CN)

download maven

[Downloading Apache Maven 3.9.4](https://maven.apache.org/download.cgi)

修改镜像库地址

配置文件 `C:\Program Files\Maven\apache-maven-3.9.3\conf\settings.xml`

```xml
<mirrors>
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>*</mirrorOf>
        <name>阿里云公共仓库</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>*</mirrorOf>
        <name>阿里云谷歌仓库</name>
        <url>https://maven.aliyun.com/repository/google</url>
    </mirror>
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>*</mirrorOf>
        <name>阿里云阿帕奇仓库</name>
        <url>https://maven.aliyun.com/repository/apache-snapshots</url>
    </mirror>
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>*</mirrorOf>
        <name>阿里云spring仓库</name>
        <url>https://maven.aliyun.com/repository/spring</url>
    </mirror>
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>*</mirrorOf>
        <name>阿里云spring插件仓库</name>
        <url>https://maven.aliyun.com/repository/spring-plugin</url>
    </mirror>
</mirrors>
```



编辑环境变量

[How do I set or change the PATH system variable?](https://www.java.com/en/download/help/path.html)

[How to Set the PATH Variable in Windows](https://techpp.com/2021/08/26/set-path-variable-in-windows-guide/)

```ini
JAVA_HOME=C:\Program Files\Java\jre-1.8
MAVEN_HOME=C:\Program Files\Maven\apache-maven-3.9.3
# Path
%MAVEN_HOME%\bin
%JAVA_HOME%\bin
```

变量生效

```bash
setx PATH "%MAVEN_HOME%\bin;%PATH%"
echo %PATH%
```

```shell
# 查看maven版本
mvn -v
java -version
```

VSCode Install extension Extension Pack for java， 修改vscode用户设置（JSON）

```json
"java.configuration.runtimes": [
  {
    "name": "JavaSE-1.8",
    "path": "C:\\Program Files\\Java\\jre-1.8"
  }
],
"maven.executable.path":"C:\\Program Files\\Maven\\apache-maven-3.9.3\\bin\\mvn",
"maven.terminal.customEnv": [
  {
      "environmentVariable": "JAVA_HOME",
      "value": "C:\\Program Files\\Java\\jre-1.8"
  }
],
"java.project.referencedLibraries": [
  "library/**/*.jar"
],
"java.configuration.maven.userSettings":"C:\\Program Files\\Maven\\apache-maven-3.9.3\\conf\\settings.xml"
```

从vscode maven 组件 新建项目,根据项目需要下载package，[maven repository](https://mvnrepository.com/artifact/com.ververica/flink-connector-mysql-cdc),国内无法访问，需要代理

```xml
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-java-bridge</artifactId>
  <version>1.17.1</version>
  <scope>provided</scope>
</dependency>
```





