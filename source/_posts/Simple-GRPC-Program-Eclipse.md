---
title: Simple_GRPC_Program_Eclipse
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2019-12-03 13:09:21
tags:
    - tips
    - 技术
    - 学习笔记
keywords: GRPC
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123843.jpg
---

#Simple GRPC Java Project on Eclipse
## Some new definitions
1. **maven中的Group Id和Artifact Id**: groupid和artifactId被统称为“坐标”是为了保证项目唯一性而提出的，如果你要把你项目弄到maven本地仓库去，你想要找到你的项目就必须根据这两个id去查找。
　　groupId一般分为多个段，这里我只说两段，第一段为域，第二段为公司名称。域又分为org、com、cn等等许多，其中org为非营利组织，com为商业组织。举个apache公司的tomcat项目例子：这个项目的groupId是org.apache，它的域是org（因为tomcat是非营利项目），公司名称是apache，artigactId是tomcat。
　　比如我创建一个项目，我一般会将groupId设置为cn.snowin，cn表示域为中国，snowin是我个人姓名缩写，artifactId设置为testProj，表示你这个项目的名称是testProj，依照这个设置，你的包结构最好是cn.snowin.testProj打头的，如果有个StudentDao，它的全路径就是cn.snowin.testProj.dao.StudentDao。
2. **Eclipse中workingset和working space的区别：**working space:完整的各个项目都罗列在内；workingset：可分门别类的将这些项目放到不同逻辑文件夹下。    

## Steps
1. 在Eclipse的workspace中新建一个Maven project。默认选择archetype为Group Id： org.apache.maven.archetypes, artifact Id为： maven-archetype-quickstart。
2. 设置groupid为：com.yusur, Artifact Id为grpcClient.
3. 在生成的pom文件中，添加GRPC的依赖。
4. 在src/main目录下新建文件夹，命名为```proto```文件夹，在文件夹中新建文件，命名为```*.proto```，该文件作为运行protobuffer的必要文件，定义了client端与server端使用的接口和通信协议。
5. ```mvn compile```命令可以调用Protobuffer的compiler编译生成相应的java文件。将java文件移动到src文件目录下，自己新建client端或server端文件，正常调用，编译，测试即可。
6. GRPC也是一个maven工程，在用mvn package打包成jar包后可以通过添加到postgresql项目中，并添加相关引用路径完成。
7. 至此将GRPC项目引入到JDBC工程中。
8. protobuffer pom文件设置：
 
```xml
      <dependency>
          <groupId>com.google.code.findbugs</groupId>
          <artifactId>jsr305</artifactId>
          <version>3.0.1</version>
      </dependency>
      <dependency>
          <groupId>io.grpc</groupId>
          <artifactId>grpc-all</artifactId>
          <version>1.23.0</version>
      </dependency>
      <extensions>
          <extension>
              <groupId>kr.motd.maven</groupId>
              <artifactId>os-maven-plugin</artifactId>
              <version>1.6.2</version>
          </extension>
      </extensions>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>3.6.1</version>
              <configuration>
                  <source>8</source>
                  <target>8</target>
              </configuration>
          </plugin>
          <plugin>
              <groupId>org.xolstice.maven.plugins</groupId>
              <artifactId>protobuf-maven-plugin</artifactId>
              <version>0.6.1</version>
              <extensions>true</extensions>
              <configuration>
                  <!--
                    The version of protoc must match protobuf-java. If you don't depend on
                    protobuf-java directly, you will be transitively depending on the
                    protobuf-java version that grpc depends on.
                  -->
                    <protocArtifact>com.google.protobuf:protoc:3.9.1:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.0.0:exe:${os.detected.classifier}</pluginArtifact>
                    <protoSourceRoot>src/main/proto</protoSourceRoot>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>  
```

 


