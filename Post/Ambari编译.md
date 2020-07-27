# Ambari编译

## 环境搭建

```bash
jdk1.8.0_161
apache-maven-3.5.3
CentOS 7.4.1708
```

## JDK、Maven安装

```shell
[root@localhost java]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core)
[root@localhost java]# java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
[root@localhost java]# mvn -v
Apache Maven 3.5.3 (3383c37e1f9e9b3bc3df5050c29c8aff9f295297; 2018-02-24T14:49:05-05:00)
Maven home: /usr/java/apache-maven-3.5.3
Java version: 1.8.0_161, vendor: Oracle Corporation
Java home: /usr/java/jdk1.8.0_161/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-693.el7.x86_64", arch: "amd64", family: "unix"
[root@localhost java]#
```

备注：安装完Maven之后，最好将镜像仓库改成阿里的镜像仓库，速度较快！

推荐Maven的settings镜像配置：

```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
<mirror>
    <id>ui</id>
    <name>Mirror from UK</name>
    <url>http://uk.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
<mirror>
    <id>jboss-public-repository-group</id>
    <name>JBoss Public Repository Group</name>
    <url>http://repository.jboss.org/nexus/content/groups/public</url>
    <mirrorOf>central</mirrorOf>
</mirror>
<mirror>
    <id>repo2</id>
    <name>Mirror from Maven Repo2</name>
    <url>http://repo2.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

## 安装rpm-build

```shell
yum install rpm-build
```

## 安装gcc&gcc-c++

```shell
yum install gcc gcc-c++
```

## 安装Git

```shell
yum install git
```

## 安装NodeJS

```shell
# 下载、编译安装、验证安装
$ wget http://nodejs.org/dist/v0.10.44/node-v0.10.44.tar.gz
$ tar zxf node-v0.10.44.tar.gz
$ cd node-v0.10.44
$ ./configure && make && sudo make install
$ node -v
# 安装插件
$ npm install -g brunch@1.7.20
$ npm install -g phantomjs@1.9.20
$ npm install -g bower
$ npm install -g gulp
```

## 安装Python2.6

```shell
$ wget https://www.python.org/ftp/python/2.6.9/Python-2.6.9.tar.xz
$ tar -Jvf Python-2.6.9.tar.xz
$ cd Python-2.6.9
$ ./configure
$ make
$ make install
$ ln -s /usr/local/bin/python2.6 /usr/bin/python2.6
$ ln -s /usr/local/bin/python2.6-config /usr/bin/python2.6-config
```

## 安装python-devel

```shell
yum install python-devel
```

## 安装python setuptools

```shell
$ wget https://pypi.python.org/packages/25/5d/cc55d39ac39383dd6e04ae80501b9af3cc455be64740ad68a4e12ec81b00/setuptools-0.6c11-py2.7.egg#md5=fe1f997bc722265116870bc7919059ea
$ wget http://pypi.python.org/packages/2.6/s/setuptools/setuptools-0.6c11-py2.6.egg#md5=bfa92100bd772d5a213eedd356d64086
$ sh setuptools-0.6c11-py2.7.egg
$ sh setuptools-0.6c11-py2.6.egg
```

# 下载大文件，修改pom.xml

有些包比较大，或者编译时下载时间较长，可以提前下载到本地目录，再修改pom.xml文件指定到本地目录，比如在Ambari2.7.3中，我提前下载好的文件为：

```xml
grafana-2.6.0.linux-x64.tar.gz
hadoop-3.1.0.3.0.0.0-1634.tar.gz
hbase-2.0.0.3.0.0.0-1634-bin.tar.gz
phoenix-5.0.0.3.0.0.0-1634.tar.gz
```

关于这几个文件的下载地址，可以在pom.xml中查看！！！

修改ambari-metrics下的pom.xml内容为：

```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <python.ver>python &gt;= 2.6</python.ver>
    <deb.python.ver>python (&gt;= 2.6)</deb.python.ver>
    <!-- 将相关内容替换为本地的地址 -->
    <hbase.tar>file:///opt/ambari-download/hbase-2.0.0.3.0.0.0-1634-bin.tar.gz</hbase.tar>
    <hbase.folder>hbase-2.0.0.3.0.0.0-1634</hbase.folder>
    <hadoop.tar>file:///opt/ambari-download/hadoop-3.1.0.3.0.0.0-1634.tar.gz</hadoop.tar>
    <hadoop.folder>hadoop-3.1.0.3.0.0.0-1634</hadoop.folder>
    <grafana.folder>grafana-2.6.0</grafana.folder>
    <grafana.tar>file:///opt/ambari-download/grafana-2.6.0.linux-x64.tar.gz</grafana.tar>
    <phoenix.tar>file:///opt/ambari-download/phoenix-5.0.0.3.0.0.0-1634.tar.gz</phoenix.tar>
    <phoenix.folder>phoenix-5.0.0.3.0.0.0-1634</phoenix.folder>
    <!-- 从下面这句可以看出编译过程中应该需要Python2.6 -->
    <resmonitor.install.dir>/usr/lib/python2.6/site-packages/resource_monitoring</resmonitor.install.dir>
    <powermock.version>1.6.2</powermock.version>
    <distMgmtSnapshotsId>apache.snapshots.https</distMgmtSnapshotsId>
    <distMgmtSnapshotsName>Apache Development Snapshot Repository</distMgmtSnapshotsName>
    <distMgmtSnapshotsUrl>https://repository.apache.org/content/repositories/snapshots</distMgmtSnapshotsUrl>
    <distMgmtStagingId>apache.staging.https</distMgmtStagingId>
    <distMgmtStagingName>Apache Release Distribution Repository</distMgmtStagingName>
    <distMgmtStagingUrl>https://repository.apache.org/service/local/staging/deploy/maven2</distMgmtStagingUrl>
    <fasterxml.jackson.version>2.9.5</fasterxml.jackson.version>
  </properties>
```

# 执行编译

```shell
$ cd apache-ambari-2.7.3-src
$ mvn versions:set -DnewVersion=2.7.3.0.0
$ pushd ambari-metrics
$ mvn versions:set -DnewVersion=2.7.3.0.0
$ popd
$ mvn -B clean install rpm:rpm -DnewVersion=2.7.3.0.0 -DbuildNumber=4295bb16c439cbc8fb0e7362f19768dde1477868 -DskipTests -Dpython.ver="python >= 2.6" -Drat.skip=true
```

# 编译遇到的问题集锦

## 问题一

编译到某一个地方，常常会卡住，直接终止，重新执行编译命令即可，另外，如果编译失败也可以用这种方式，如果两次三次不生效，且报错一致，那么就根据报错进行排错。

# 编译成功的标志

```shell
[INFO] + exit 0
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] Ambari Main 2.7.3.0.0 .............................. SUCCESS [  3.442 s]
[INFO] Apache Ambari Project POM .......................... SUCCESS [  0.096 s]
[INFO] Ambari Web ......................................... SUCCESS [ 51.557 s]
[INFO] Ambari Views ....................................... SUCCESS [  2.089 s]
[INFO] Ambari Admin View .................................. SUCCESS [  5.611 s]
[INFO] ambari-utility 1.0.0.0-SNAPSHOT .................... SUCCESS [  2.837 s]
[INFO] ambari-metrics ..................................... SUCCESS [  0.701 s]
[INFO] Ambari Metrics Common .............................. SUCCESS [ 10.591 s]
[INFO] Ambari Metrics Hadoop Sink ......................... SUCCESS [  5.419 s]
[INFO] Ambari Metrics Flume Sink .......................... SUCCESS [  2.393 s]
[INFO] Ambari Metrics Kafka Sink .......................... SUCCESS [  2.123 s]
[INFO] Ambari Metrics Storm Sink .......................... SUCCESS [  4.527 s]
[INFO] Ambari Metrics Storm Sink (Legacy) ................. SUCCESS [  3.932 s]
[INFO] Ambari Metrics Collector ........................... SUCCESS [02:41 min]
[INFO] Ambari Metrics Monitor ............................. SUCCESS [  1.275 s]
[INFO] Ambari Metrics Grafana 2.1.0.0.0 ................... SUCCESS [  0.939 s]
[INFO] Ambari Metrics Host Aggregator ..................... SUCCESS [  5.329 s]
[INFO] Ambari Metrics Assembly ............................ SUCCESS [01:30 min]
[INFO] Ambari Service Advisor 1.0.0.0-SNAPSHOT ............ SUCCESS [  0.486 s]
[INFO] Ambari Server ...................................... SUCCESS [30:42 min]
[INFO] Ambari Functional Tests ............................ SUCCESS [  0.994 s]
[INFO] Ambari Agent ....................................... SUCCESS [02:58 min]
[INFO] ambari-logsearch ................................... SUCCESS [  1.581 s]
[INFO] Ambari Logsearch Appender .......................... SUCCESS [ 14.171 s]
[INFO] Ambari Logsearch Config Api ........................ SUCCESS [  0.299 s]
[INFO] Ambari Logsearch Config JSON ....................... SUCCESS [  0.289 s]
[INFO] Ambari Logsearch Config Solr ....................... SUCCESS [ 14.207 s]
[INFO] Ambari Logsearch Config Zookeeper .................. SUCCESS [  1.067 s]
[INFO] Ambari Logsearch Config Local ...................... SUCCESS [  0.127 s]
[INFO] Ambari Logsearch Log Feeder Plugin Api ............. SUCCESS [ 10.531 s]
[INFO] Ambari Logsearch Log Feeder Container Registry ..... SUCCESS [  8.680 s]
[INFO] Ambari Logsearch Log Feeder ........................ SUCCESS [ 59.983 s]
[INFO] Ambari Logsearch Web ............................... SUCCESS [03:41 min]
[INFO] Ambari Logsearch Server ............................ SUCCESS [03:03 min]
[INFO] Ambari Logsearch Assembly .......................... SUCCESS [  5.146 s]
[INFO] Ambari Logsearch Integration Test .................. SUCCESS [01:11 min]
[INFO] ambari-infra ....................................... SUCCESS [ 12.123 s]
[INFO] Ambari Infra Solr Client ........................... SUCCESS [ 12.837 s]
[INFO] Ambari Infra Solr Plugin ........................... SUCCESS [02:18 min]
[INFO] Ambari Infra Manager ............................... SUCCESS [01:51 min]
[INFO] Ambari Infra Assembly .............................. SUCCESS [ 11.453 s]
[INFO] Ambari Infra Manager Integration Tests 2.7.3.0.0 ... SUCCESS [ 16.413 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 54:34 min
[INFO] Finished at: 2019-04-16T22:54:37-04:00
[INFO] ------------------------------------------------------------------------
```
