# Maven 简介

Maven通过完成以下的工作， 来帮助人类减少构建软件过程中的错误。

- 下载依赖
- 将附加的jars附加到类的路径
- 编译源代码到二进制码
- 运行测试
- 打包成jar， war
- 部署



## Project Object Model

pom是maven项目的配置文件， pom定义了项目， 管理依赖， 配置插件, 下面示例是一个pom文件的基本结构.

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>...</name>
    <url>...</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
            //...
            </plugin>
        </plugins>
    </build>

```

### Project Identifiers
- *groupId* - 组织或者公司名
- *artifactId* - 项目独一无二的名字
- *version* - 项目的版本
- *packaging* - 打包方法( war / jar / zip)

### Dependencies
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>4.3.5.RELEASE</version>
</dependency>
```

maven会从远程仓库自动下载依赖


### Repositories
Maven中的存储库用于保存不同类型的构建工件和依赖项。默认本地存储库位于用户主目录下的.m2/repository文件夹中。

如果本地已存在依赖， maven会使用它， 否则maven会从[Maven central](https://search.maven.org/classic/#search|ga|1|centra)进行下载, 如果有些项目在maven库找不到的话，
可以自己提供repository url

```xml
<repositories>
    <repository>
        <id>JBoss repository</id>
        <url>http://repository.jboss.org/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

### Properties
用户可以使用 ${name} 来使用定义的属性， 比如下面的示例
```xml
<properties>
    <spring.version>4.3.5.RELEASE</spring.version>
</properties>
 
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
</dependencies>
```
以后需要升级版本的时候只需要改变``` <spring.version> ```即可


### Build
tag为build的部分，定义了编译之后的信息， 如下面的示例
```xml
<build>
    <defaultGoal>install</defaultGoal>
    <directory>${basedir}/target</directory>
    <finalName>${artifactId}-${version}</finalName>
    <filters>
      <filter>filters/filter1.properties</filter>
    </filters>
    //...
</build>
```


### Using Profiles
Maven的另一个重要特性是它对配置文件的支持。配置文件基本上是一组配置值。通过使用配置文件，您可以为不同的环境（如生产/测试/开发）自定义构建

```xml
<profiles>
    <profile>
        <id>production</id>
        <build>
            <plugins>
                <plugin>
                //...
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>development</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <build>
            <plugins>
                <plugin>
                //...
                </plugin>
            </plugins>
        </build>
     </profile>
 </profiles>
```

默认的配置文件是development， 如果想使用生产的配置文件， 可以使用下面的命令```mvn clean install -Pproduction```



## Maven Build Lifecycles
每个Maven构建都遵循指定的生命周期。您可以执行多个构建生命周期目标，包括编译项目代码，创建包以及在本地Maven依赖存储库中安装存档文件的目标。


以下列表显示了最重要的Maven生命周期阶段：

- *validate* - 检查项目的正确性
- *compile* - 将提供的源代码编译为二进制工件
- *test* - 执行单元测试
- *package* - 将已编译的代码打包到存档文件中
- *integration-test* - 执行需要打包的其他测试
- *verify* - 检查打包文件是否有效
- *install* - 在本地maven仓库安装文件
- *deploy* - 部署打包文件到远程服务器或者仓库

### Multi-Module Projects

Maven中处理多模块项目（也称为聚合器项目）的机制称为Reactor。

Reactor收集所有可用的模块进行构建，然后将项目分类到正确的构建顺序，最后逐个构建它们

pom.xml示例

```xml
<parent>
    <groupId>xxxx</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<modules>
    <module>core</module>
    <module>service</module>
    <module>webapp</module>
</modules>
```




## 参考资料
[Apache Maven Tutorial](https://www.baeldung.com/maven)

