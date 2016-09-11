title: maven
date: 2015-10-30 14:40:45
tags: maven
categories: developEnv
---

管理系统依赖
统一依赖的来源和版本，保持开发环境一致，避免莫名错误

管理项目生命周期
构建，编译，测试，发布，更新版本。。。。

<!-- more -->

## 安装
* 前提
确保已经安装好了jdk，并且配置好了环境变量JAVA_HOME

* [下载](http://maven.apache.org/download.cgi)

* 解压
解压下载的maven到你想要的目录

      tar xzvf apache-maven-3.3.9-bin.tar.gz

* 环境变量
新增 环境变量`M2_HOME`，其值为Maven的家目录，即上步maven解压的目录；
修改 环境变量`PATH`， 在末尾加入 `:$M2_HOME/bin`

* 测试

    mvn -v

## setting.xml


定义Maven的全局环境信息。有2个地址：

`$M2_HOME/conf/setting.xml` : 全局的配置
`$User_HOME/.m2/setting.xml` : 当前用户的配置
当这两个文件同时存在的时候，那么对于相同的配置信息用户家目录下面的settings.xml中定义的会覆盖Maven安装目录下面的settings.xml中的定义。

setting.xml有点像这样：

```
  <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
	      http://maven.apache.org/xsd/settings-1.0.0.xsd">

      <profiles>
	  <!-- dev -->
	  <profile>
	      <id>dev</id>
	      <properties>
		  <profile.id>dev</profile.id>
		  <packagingExcludes> *.html
		  </packagingExcludes>
	      </properties>
	      <!-- local repositories of dependencies -->
	      <repositories>
		  <repository>
		      <id>nexus</id>
		      <url>http://IP:8081/nexus/content/groups/public/</url>
		      <releases>
			  <enabled>true</enabled>
		      </releases>
		      <snapshots>
			  <enabled>true</enabled>
		      </snapshots>
		  </repository>
	      </repositories>
	      <!-- local Repositories of plugins -->
	      <pluginRepositories>
		  <pluginRepository>
		    <id>nexus</id>
		    <url>http://IP:8081/nexus/content/groups/public/</url>
		    <releases>
		      <enabled>true</enabled>
		    </releases>
		    <snapshots>
		      <enabled>true</enabled>
		    </snapshots>
		  </pluginRepository>
	      </pluginRepositories>
	  </profile>
	  
	  <!-- test -->
	  <profile>
	      <id>test</id>
	      <properties>
		  <profile.id>test</profile.id>
		  <packagingExcludes> *.html
		  </packagingExcludes>
	      </properties>

	  </profile>
	  
	  <!-- prod -->
	  <profile>
	      <id>prod</id>
	      <properties>
		  <profile.id>prod</profile.id>
		  <packagingExcludes> *.html
		  </packagingExcludes>
	      </properties>

	  </profile>

	  <!-- 发布到nexus -->
	  <profile>
	      <id>nexus</id>
	      <properties>
		  <profile.id>nexus</profile.id>
		  <!-- 打包不传递依赖 -->
		  <packagingExcludes>WEB-INF/lib/*, assets/**, *.html</packagingExcludes>
	      </properties>
	  </profile>
      </profiles>
      
      <activeProfiles>
	  <activeProfile>dev</activeProfile>
      </activeProfiles>
      
      


      <!-- in mirrors section -->
      <mirrors>
	  <mirror>
	      <id>mirrorOfAll</id>
	      <mirrorOf>*</mirrorOf>
	      <url>http://IP:8081/nexus/content/groups/public/</url>
	  </mirror>
      </mirrors>

      <!-- in servers section -->
      <servers>
	  <server>
	      <id>snapshots</id>
	      <username>user</username>
	      <password>password</password>
	  </server>
	  <server>
	      <id>releases</id>
	      <username>user</username>
	      <password>password</password>
	  </server>
      </servers>
  </settings>
```

`IP` `user` `password` 都是本地资源库（nexus）相关的配置。


[参考资源](http://blog.csdn.net/jinshuaiwang/article/details/23686099)

## pom.xml
maven 管理 java 工程都是使用 `pom.xml`, 它长得有点像这样
```
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
	  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>

      <groupId>top.giveme5.base</groupId>
      <artifactId>gm5-base-root</artifactId>
      <version>1.0.0-SNAPSHOT</version>

      <packaging>pom</packaging>


      <!--项目的名称, Maven产生的文档用-->
      <name>gm5-base-root</name>

      <!-- 项目的详细描述, Maven 产生的文档用。  当这个元素能够用HTML格式描述时（例如，CDATA中的文本会被解析器忽略，就可以包含HTML标 签）， 不鼓励使用纯文本描述。如果你需要修改产生的web站点的索引页面，你应该修改你自己的索引页文件，而不是调整这里的文档。-->
      <description>giveme5-base-root</description>


      <!-- SCM：Software Configuration Management -->
      <scm>
	  <!--SCM的URL,该URL描述了版本库和如何连接到版本库。欲知详情，请看SCMs提供的URL格式和列表。该连接只读。-->
	  <connection>scm:git:git@IP/giveme5-base/giveme5-base-root.git</connection>

	  <!--给开发者使用的，类似connection元素。即该连接不仅仅只读-->
	  <developerConnection>scm:git:git@IP:giveme5-base/giveme5-base-root.git</developerConnection>

	  <!--指向项目的可浏览SCM库（例如ViewVC或者Fisheye）的URL。-->
	  <url>http://IP/giveme5-base-root</url>
      </scm>

      <!-- local repositories of dependencies -->
      <repositories>
	  <repository>
	      <id>nexus</id>
	      <url>http://IP:8081/nexus/content/groups/public/</url>
	      <releases>
		  <enabled>true</enabled>
	      </releases>
	      <snapshots>
		  <enabled>true</enabled>
	      </snapshots>
	  </repository>
      </repositories>
      <!-- local Repositories of plugins -->
      <pluginRepositories>
	  <pluginRepository>
	    <id>nexus</id>
	    <url>http://IP:8081/nexus/content/groups/public/</url>
	    <releases>
	      <enabled>true</enabled>
	    </releases>
	    <snapshots>
	      <enabled>true</enabled>
	    </snapshots>
	  </pluginRepository>
      </pluginRepositories>
      <distributionManagement>
	  <snapshotRepository>
	      <id>snapshots</id>
	      <url>http://IP:8081/nexus/content/repositories/snapshots/</url>
	  </snapshotRepository>
	  <repository>
	      <id>releases</id>
	      <url>http://IP:8081/nexus/content/repositories/releases/</url>
	  </repository>
      </distributionManagement>


      <!-- 项目属性 -->
      <properties>
	  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	  <jdk.version>1.8</jdk.version>
	  <jdbc.driver.groupId>mysql</jdbc.driver.groupId>
	  <jdbc.driver.artifactId>mysql-connector-java</jdbc.driver.artifactId>
	  <jdbc.driver.version>${mysql-connector-java.version}</jdbc.driver.version>

	  <spring-framework-bom.version>4.2.1.RELEASE</spring-framework-bom.version>
	    。。。
	  <freemarker.version>2.3.23</freemarker.version>
      </properties>

      <!-- build -->
      <build>
	  <pluginManagement>
	      <!-- define plubins's version and common config which will be shared with
		  all subprojects -->
	      <plugins>

		  <!--  based on and is a replacement for the maven-release-plugin enabling support for git-flow style releases via maven. -->
		  <plugin>
		      <groupId>external.atlassian.jgitflow</groupId>
		      <artifactId>jgitflow-maven-plugin</artifactId>
		      <version>1.0-m5.1</version>
		      <configuration>
			  <flowInitContext>
			      <masterBranchName>master</masterBranchName>
			      <developBranchName>develop</developBranchName>
			      <featureBranchPrefix>feature-</featureBranchPrefix>
			      <releaseBranchPrefix>release-</releaseBranchPrefix>
			      <hotfixBranchPrefix>hotfix-</hotfixBranchPrefix>
			      <versionTagPrefix>giveme5-</versionTagPrefix>
			  </flowInitContext>
		      </configuration>
		  </plugin>

		  <!-- compiler插件, 设定JDK版本 -->
		  <plugin>
		      <groupId>org.apache.maven.plugins</groupId>
		      <artifactId>maven-compiler-plugin</artifactId>
		      <version>3.1</version>
		      <configuration>
			  <source>${jdk.version}</source>
			  <target>${jdk.version}</target>
			  <showWarnings>true</showWarnings>
		      </configuration>
		  </plugin>
		  
		  
		  ...

	      </plugins>

	  </pluginManagement>
      </build>

      <!-- dependencyManagement -->
      <dependencyManagement>
	  <dependencies>


	      <!-- spring -->
	      <dependency>
		  <groupId>org.springframework</groupId>
		  <artifactId>spring-framework-bom</artifactId>
		  <version>${spring-framework-bom.version}</version>
		  <type>pom</type>
		  <scope>import</scope>
	      </dependency>

	      ...
	      
      </dependencyManagement>

  </project>
```



## 插件
* [maven-war-plugin](/2015/10/30/devEnv/maven/maven/#maven-war-plugin)



## maven-war-plugin
此插件多用于web工程打包时使用，多工程依赖时很好用，很好用，很好用！[官网](https://maven.apache.org/plugins/maven-war-plugin/)


```
root 工程 pom
  <!-- war打包插件, 设定war包名称不带版本号 -->
  <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-war-plugin</artifactId>
      <version>2.6</version>
      <configuration>
	  <failOnMissingWebXml>false</failOnMissingWebXml>
	  <warName>${project.artifactId}</warName>
      </configuration>
  </plugin>
```

被依赖的工程 pom
```
    <artifactId>gm5-base-common-web</artifactId>

    <packaging>war</packaging>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                    <!--  把class额外打包jar,默认：artifactId-classes.jar-->
                    <attachClasses>true</attachClasses>

                    <!--  把class打包jar,jar包放入WEB-INF/lib下，WEB-INF/classes下将无类文件  jrebel使用是注释掉-->
                    <!--<archiveClasses>true</archiveClasses>-->

                    <!--<packagingExcludes>WEB-INF/classes/conf/account.properties</packagingExcludes>-->
                    <!--<packagingExcludes>WEB-INF/lib/</packagingExcludes>-->
                    <!--<warSourceExcludes>**/conf/account.properties</warSourceExcludes>-->

                </configuration>
            </plugin>
        </plugins>
    </build>
```

最终工程 pom
```
    <artifactId>gm5-base-security</artifactId>
    <packaging>war</packaging>

    <build>
        <!--<finalName>gm5-base-security</finalName>-->
        <plugins>
            <!-- 合并多个war -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <useCache>false</useCache>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                    <overlays>
                        <overlay>
                            <groupId>top.giveme5.base</groupId>
                            <artifactId>gm5-base-common-web</artifactId>
                        </overlay>
                    </overlays>
                    <dependentWarExcludes>WEB-INF/lib/*</dependentWarExcludes>
                    <packagingExcludes>WEB-INF/lib/gm5-base-common-web-*-classes.jar</packagingExcludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### configuration 说明
* failOnMissingWebXml ： 校验是否有web.xml ，默认true，没发现web.xml的话，打包失败。servlet 3.0以后 可以没有web.xml, 我使用这个特性，所以设置为false。
* archiveClasses ： 把class打包jar,jar包放入WEB-INF/lib下，WEB-INF/classes下将无类文件
* attachClasses  ： 把class额外打包jar,默认：artifactId-classes.jar，WEB-INF/classes下仍然有类文件
* packagingExcludes ： 复制webapp目录完成后打包时忽略target/artifactId-version 制定 文件夹的文件， 注意 文件夹必须以 `/` 结束
* warSourceExcludes ： 在编译周期进行完成后从src/main/webapp目录复制文件时忽略
* overlays/overlay ： 能够在目标WAR本身覆盖除了原生WAR构件以外的所有文件，并在WEB-INF/lib目录下收集原生WAR项目的依赖， 目标war中已经存在的文件不会被覆盖。Overlays采用第一直达者优先的策略(因此，如果一个文件被某一个副本覆盖过，则它不会被另一个副本继续覆盖)。(详细)[http://jdonee.iteye.com/blog/794226]


### 多依赖使用
* 最终目标工程编译不需要被依赖项目
只需要在目标工程 pom 加入依赖
```
  <dependency>
      <groupId>top.giveme5.base</groupId>
      <artifactId>gm5-base-common-web</artifactId>
      <version>${project.version}</version>
      <type>war</type>
  </dependency>
```

* 最终目标工程编译需要被依赖项目
为避免 `overlays` 同时使用，重复引入被依赖项目的类文件，如下设置：

目标工程 pom 

```
    <artifactId>gm5-base-security</artifactId>
    <packaging>war</packaging>

    <build>
        <!--<finalName>gm5-base-security</finalName>-->
        <plugins>
            <!-- 合并多个war -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <useCache>false</useCache>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                    <overlays>
                        <overlay>
                            <groupId>top.giveme5.base</groupId>
                            <artifactId>gm5-base-common-web</artifactId>
                        </overlay>
                    </overlays>
                    <dependentWarExcludes>WEB-INF/lib/*</dependentWarExcludes>

                    <!--避免overlays重复引入-->
                    <packagingExcludes>WEB-INF/lib/gm5-base-common-web-*-classes.jar</packagingExcludes>
                </configuration>
            </plugin>
        </plugins>
    </build>


    <dependencies>

        <!-- give me five -->
        <dependency>
            <groupId>top.giveme5.base</groupId>
            <artifactId>gm5-base-common-core</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>top.giveme5.base</groupId>
            <artifactId>gm5-base-common-web</artifactId>
            <version>${project.version}</version>
            <type>war</type>
        </dependency>

        <!-- compile 时依赖， 最终 package 时 使用 overlays -->
        <dependency>
            <groupId>top.giveme5.base</groupId>
            <artifactId>gm5-base-common-web</artifactId>
            <version>${project.version}</version>
            <type>jar</type>
            <classifier>classes</classifier>
        </dependency>


        。。。
```


被依赖工程 pom
```
```
    <artifactId>gm5-base-common-web</artifactId>

    <packaging>war</packaging>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                    <!--  把class额外打包jar,默认：artifactId-classes.jar-->
                    <attachClasses>true</attachClasses>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

以上是为了 `overlays`， 如果不需要 `overlays`， 可以使用 archiveClasses 简单处理。
