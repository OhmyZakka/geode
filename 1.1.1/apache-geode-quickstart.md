# Apache Geode Quick Start
## 相关资料
下载地址：http://geode.apache.org/releases/    
官方文档：http://geode.apache.org/docs/guide/about_geode.html    
Github：https://github.com/apache/geode    
## 环境需求
1. JDK8 或最新版本
2. 正确的时间和时间同步服务
3. hostname和/etc/hosts文件必须配置正确，否则会影响gfsh命令功能

下载java rpm或tar.gz

    $ wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm"   
    
    $ wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz"    

## 安装
#### 1. 设置JAVA_HOME
    
    $ JAVA_HOME=/usr/java/jdk.8.0_131
    $ export JAVA_HOME   
#### 2. 源码安装
在源码解压后的目录中，以非测试方式编译    
    
    $ ./gradlew build -Dskip.tests=true   
#### 3. 解压安装    
下载二进制格式的.zip或是.tar文件解压即可   
    
    $ cd /tmp/
    $ tar zxvf /tmp/apache-geode-1.1.1.tar.gz -C /usr/local/
    $ ln -s /usr/local/apache-geode-1.1.1 /usr/local/geode

#### 4. 检查安装
编译或解压后进入bin目录

    $ gfsh version
    
#### 5. 配置环境变量    

    $ vim /etc/profile
    export JAVA_HOME=/usr/java/jdk1.8.0_131    
    export GEODE_HOME=/usr/local/geode    
    export PATH=$JAVA_HOME/bin:$GEODE_HOME/bin:$PATH    
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$GEODE_HOME/lib/geode-dependencies.jar        
    
    $ ./etc/profile    
    
## 快速入门

#### 1. 创建一个geode用户

    $ groupadd geode
    $ useradd -g geode geode

#### 2. 创建一个工作目录

    $ cd /home/geode
    
#### 3. 开启 gfsh

    $ gfsh    
```
        _________________________     __
   / _____/ ______/ ______/ /____/ /
  / /  __/ /___  /_____  / _____  / 
 / /__/ / ____/  _____/ / /    / /  
/______/_/      /______/_/    /_/    1.1.1

Monitor and Manage Apache Geode
```
    
#### 4. 启动一个 GemFire Locator   

    gfsh> start locator --name=locator1    
```
Starting a Geode Locator in /home/geode/locator1...
........
Locator in /home/geode/locator1 on db1[10334] as locator1 is currently online.
Process ID: 26983
Uptime: 5 seconds
Geode Version: 1.1.1
Java Version: 1.8.0_131
Log File: /home/geode/locator1/locator1.log
JVM Arguments: -Dgemfire.enable-cluster-configuration=true -Dgemfire.load-cluster-configuration-from-dir=false -Dgemfire.launcher.registerSignalHandlers=true -Djava.awt.headless=true -Dsun.rmi.dgc.server.gcInterval=9223372036854775806
Class-Path: /usr/local/geode/lib/geode-core-1.1.1.jar:/usr/local/geode/lib/geode-dependencies.jar

Successfully connected to: JMX Manager [host=db1, port=1099]

Cluster configuration service is up and running.
```

    
#### 5. 启动一个缓存服务器

    gfsh> start server --name=server1 --server-port=40411   
```
Starting a Geode Server in /home/geode/server1...
.........
Server in /home/geode/server1 on db1[40411] as server1 is currently online.
Process ID: 27106
Uptime: 5 seconds
Geode Version: 1.1.1
Java Version: 1.8.0_131
Log File: /home/geode/server1/server1.log
JVM Arguments: -Dgemfire.default.locators=192.168.0.116[10334] -Dgemfire.use-cluster-configuration=true -Dgemfire.start-dev-rest-api=false -XX:OnOutOfMemoryError=kill -KILL %p -Dgemfire.launcher.registerSignalHandlers=true -Djava.awt.headless=true -Dsun.rmi.dgc.server.gcInterval=9223372036854775806
Class-Path: /usr/local/geode/lib/geode-core-1.1.1.jar:/usr/local/geode/lib/geode-dependencies.jar
```

#### 6. 创建一个region

    gfsh> create region --name=region1 --type=REPLICATE_PERSISTENT  

#### 7. 查看集群上的 region列表

    gfsh> list regions
    
#### 8. 查看集群成员列表。启动的locator,server都在该列表中。

    gfsh> list members  
    
#### 9. describe region   
    
    gfsh> describe region --name=region1
    
#### 10. put 数据    

    gfsh>put --region=region1 --key="1" --value="one"

#### 11. query

    gfsh> query --query="select * from /region1"

#### 12. 启动第二个server   

    gfsh> start server --name=server2 --server-port=40412    

当gfsh启动服务器时，它会从群集配置服务请求配置，集群配置服务会将共享配置分发到加入群集的任何新服务器。 
当stop server —name=server1后region1所有数据仍可用。   

#### 13. 关闭server（数据会持久化到磁盘，再次使用仍将被写入缓存中）    

    gfsh> stop server --name=server1    
#### 14. 关闭集群   

    gfsh> shutdown --include-locators=true    
#### 15. 开启Pulse   
打开Pulse，一个Web应用程序，它提供了一个图形仪表板，用于监控Geode集群，成员和regions的重要的实时健康和性能。 

http://localhost:7070/pulse/clusterDetail.html 

## 一个示例 Java Application   
#### 1. 创建HelloWold.java    
```
import java.util.Map;
import org.apache.geode.cache.Region;
import org.apache.geode.cache.client.*;

public class HelloWorld {
  public static void main(String[] args) throws Exception {
    ClientCache cache = new ClientCacheFactory()
      .addPoolLocator("localhost", 10334)
      .create();
    Region<String, String> region = cache
      .<String, String>createClientRegionFactory(ClientRegionShortcut.CACHING_PROXY)
      .create("region1");

    region.put("1", "Hello");
    region.put("2", "World");

    for (Map.Entry<String, String>  entry : region.entrySet()) {
      System.out.format("key = %s, value = %s\n", entry.getKey(), entry.getValue());
    }
    cache.close();
  }
}
```
#### 2. 编译运行   

    javac -cp /some/path/geode/geode-assembly/build/install/apache-geode/lib/geode-dependencies.jar HelloWorld.java 
    java -cp .:/some/path/geode/geode-assembly/build/install/apache-geode/lib/geode-dependencies.jar HelloWorld    

因为前面配置了CLASSPATH,可以直接用:     
javac HelloWorld.java    
Java HelloWorld   
