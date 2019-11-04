# Debug in idea 
1. 获取源码  git clone https://github.com/apache/incubator-druid.git
2. build 源码  mvn clean install -DskipTests
3. idea 导入项目
4. 在.idea下面新建runConfigurations文件夹， 导入各个服务xml文件, Coordinator服务示例

```xml
<component name="ProjectRunConfigurationManager">
  <configuration default="false" name="Coordinator" type="Application" factoryName="Application">
    <extension name="coverage" enabled="false" merge="false" sample_coverage="true" runner="idea" />
    <option name="MAIN_CLASS_NAME" value="org.apache.druid.cli.Main" />
    <option name="VM_PARAMETERS" value="-server -Duser.timezone=UTC -Dfile.encoding=UTF-8 -Xmx256M -Xmx256M -XX:+UseG1GC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintAdaptiveSizePolicy -XX:+PrintReferenceGC -verbose:gc -XX:+PrintFlagsFinal -Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager -Dorg.jboss.logging.provider=slf4j -Ddruid.host=localhost -Ddruid.service=coordinator -Ddruid.extensions.directory=$PROJECT_DIR$/distribution/target/extensions/ -Ddruid.extensions.loadList=[\&quot;druid-s3-extensions\&quot;,\&quot;druid-histogram\&quot;,\&quot;mysql-metadata-storage\&quot;] -Ddruid.zk.service.host=localhost -Ddruid.metadata.storage.type=mysql -Ddruid.metadata.storage.connector.connectURI=&quot;jdbc:mysql://localhost:3306/druid&quot; -Ddruid.metadata.storage.connector.user=druid -Ddruid.metadata.storage.connector.password=diurd -Ddruid.serverview.type=batch -Ddruid.emitter=logging -Ddruid.coordinator.period=PT10S -Ddruid.coordinator.startDelay=PT5S" />
    <option name="PROGRAM_PARAMETERS" value="server coordinator" />
    <option name="WORKING_DIRECTORY" value="file://$PROJECT_DIR$" />
    <option name="ALTERNATIVE_JRE_PATH_ENABLED" value="false" />
    <option name="ALTERNATIVE_JRE_PATH" value="1.8" />
    <option name="ENABLE_SWING_INSPECTOR" value="false" />
    <option name="ENV_VARIABLES" />
    <option name="PASS_PARENT_ENVS" value="true" />
    <module name="druid-services" />
    <envs />
    <method />
  </configuration>
</component>
```
