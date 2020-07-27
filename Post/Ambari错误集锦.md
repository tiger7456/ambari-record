# Ambari错误集锦

## SmartSense启动报错

在web页面上报错：

```xml
Failed to execute command: /usr/sbin/hst activity-analyzer start ; Exit code: 1; stdout: SmartSense Activity PID at: /var/run/smartsense-activity-analyzer/activity-analyzer.pid
SmartSense Activity out at: /var/log/smartsense-activity/activity-analyzer.out
SmartSense Activity log at: /var/log/smartsense-activity/activity-analyzer.log
Waiting for activity analyzer to start...................
SmartSense Activity Analyzer failed to start, with exitcode -1. Check /var/log/smartsense-activity/activity-analyzer.log for more information.
User root failed to execute command : /usr/sbin/hst activity-analyzer start;
stderr:
```

进入到相应的日志页面发现*.log没有内容，于是查看另外一个文件：

```xml
[root@ambari1 smartsense-activity]# cat activity-analyzer.out
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/hadoop/conf/Configuration
        at com.hortonworks.smartsense.activity.util.ActivityUtil.getHadoopConfig(ActivityUtil.java:435)
        at com.hortonworks.smartsense.activity.util.ActivityUtil.login(ActivityUtil.java:456)
        at com.hortonworks.smartsense.activity.ActivityAnalyzerFacade.init(ActivityAnalyzerFacade.java:193)
        at com.hortonworks.smartsense.activity.ActivityAnalyzerFacade.main(ActivityAnalyzerFacade.java:95)
Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.conf.Configuration
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:338)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        ... 4 more
[root@ambari1 smartsense-activity]# cat activity-explorer.out
ZEPPELIN_CLASSPATH: :/etc/ams-hbase/conf::/usr/hdp/share/hst/activity-explorer/lib/interpreter/*:/usr/hdp/share/hst/activity-explorer/lib/*:/usr/hdp/share/hst/activity-explorer/*:/etc/ams-hbase/conf::/etc/smartsense-activity/conf
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256m; support was removed in 8.0
```

查看日志可以发现，Hadoop相关的类缺少，于是猜测可能是因为没有安装Hadoop相关组件，于是安装HDFS，YARN+MapReduce2组件，然后重启该组件，重启成功！

当然了，最主要的原因还是因为这个服务是收费的。

## Ambari必须时间同步

Ambari必须使用ntpd实现时间同步，否则的话会报各种各样的错，比如这里举个例子：

```xml
resource_management.core.exceptions.ExecutionFailed: Execution of 'curl -sS -L -w '%{http_code}' -X PUT --data-binary @/usr/hdp/current/zeppelin-server/interpreter/spark/dep/zeppelin-spark-dependencies-0.6.0.jar -H 'Content-Type: application/octet-stream' 'http://ambari1.org.cn:50070/webhdfs/v1/apps/zeppelin/zeppelin-spark-dependencies-0.6.0.jar?op=CREATE&user.name=hdfs&overwrite=True&permission=444' 1>/tmp/tmp2DrG0X 2>/tmp/tmp7sn96J' returned 55. curl: (55) Send failure: Connection reset by peer
```

从上面的报错我们可以知道，curl上传某一个jar包到HDFS上面，报错，于是我们使用手动的方式上传，发现依旧报错，并且此时所有的组件都正常启动起来，但是Ambari的界面监控信息显示不全，于是可以断定是ntpd没有启动起来，查看状态：systemctl status ntpd.service，这时候我们发现如下信息：

```shell
[root@ambari1 hdfs]# systemctl status ntpd
● ntpd.service - Network Time Service
Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled; vendor preset: disabled)
Active: inactive (dead)
10月 29 17:07:51 localhost.localdomain ntpd[786]: signal_no_reset: signal 17 had flags 4000000
10月 29 17:08:05 ambari1.org.cn ntpd[768]: Listen normally on 4 ens33 192.168.17.221 UDP 123
10月 29 17:08:05 ambari1.org.cn ntpd[768]: Listen normally on 5 ens33 fe80::20c:29ff:fe3c:45d5 UDP 123
10月 29 17:08:05 ambari1.org.cn ntpd[768]: new interface(s) found: waking up resolver
10月 29 17:08:07 ambari1.org.cn ntpd_intres[786]: DNS 0.centos.pool.ntp.org -> 37.187.100.18
10月 29 17:08:07 ambari1.org.cn ntpd_intres[786]: DNS 1.centos.pool.ntp.org -> 37.187.100.18
10月 29 17:08:07 ambari1.org.cn ntpd_intres[786]: DNS 2.centos.pool.ntp.org -> 5.103.139.163
10月 29 17:08:07 ambari1.org.cn ntpd_intres[786]: DNS 3.centos.pool.ntp.org -> 108.59.2.24
10月 29 17:12:51 ambari1.org.cn systemd[1]: Stopping Network Time Service...
10月 29 17:12:51 ambari1.org.cn systemd[1]: Stopped Network Time Service.
```

说明确实没有启动起来，重启服务，上传文件，问题可以得以解决！

## Ambari安装Hive错误

Ambari安装Hive出现错误：

```xml
Traceback (most recent call last):
  File "/usr/lib/ambari-agent/lib/resource_management/core/source.py", line 195, in get_content
    web_file = opener.open(req)
  File "/usr/lib64/python2.7/urllib2.py", line 437, in open
    response = meth(req, response)
  File "/usr/lib64/python2.7/urllib2.py", line 550, in http_response
    'http', request, response, code, msg, hdrs)
  File "/usr/lib64/python2.7/urllib2.py", line 475, in error
    return self._call_chain(*args)
  File "/usr/lib64/python2.7/urllib2.py", line 409, in _call_chain
    result = func(*args)
  File "/usr/lib64/python2.7/urllib2.py", line 558, in http_error_default
    raise HTTPError(req.get_full_url(), code, msg, hdrs, fp)
HTTPError: HTTP Error 404: Not Found
The above exception was the cause of the following exception:
Traceback (most recent call last):
  File "/var/lib/ambari-agent/cache/stacks/HDP/3.0/services/HIVE/package/scripts/hive_client.py", line 60, in <module>
    HiveClient().execute()
  File "/usr/lib/ambari-agent/lib/resource_management/libraries/script/script.py", line 352, in execute
    method(env)
  File "/var/lib/ambari-agent/cache/stacks/HDP/3.0/services/HIVE/package/scripts/hive_client.py", line 40, in install
    self.configure(env)
  File "/var/lib/ambari-agent/cache/stacks/HDP/3.0/services/HIVE/package/scripts/hive_client.py", line 48, in configure
    hive(name='client')
  File "/var/lib/ambari-agent/cache/stacks/HDP/3.0/services/HIVE/package/scripts/hive.py", line 114, in hive
    jdbc_connector(params.hive_jdbc_target, params.hive_previous_jdbc_jar)
  File "/var/lib/ambari-agent/cache/stacks/HDP/3.0/services/HIVE/package/scripts/hive.py", line 634, in jdbc_connector
    File(params.downloaded_custom_connector, content = DownloadSource(params.driver_curl_source))
  File "/usr/lib/ambari-agent/lib/resource_management/core/base.py", line 166, in __init__
    self.env.run()
  File "/usr/lib/ambari-agent/lib/resource_management/core/environment.py", line 160, in run
    self.run_action(resource, action)
  File "/usr/lib/ambari-agent/lib/resource_management/core/environment.py", line 124, in run_action
    provider_action()
  File "/usr/lib/ambari-agent/lib/resource_management/core/providers/system.py", line 123, in action_create
    content = self._get_content()
  File "/usr/lib/ambari-agent/lib/resource_management/core/providers/system.py", line 160, in _get_content
    return content()
  File "/usr/lib/ambari-agent/lib/resource_management/core/source.py", line 52, in __call__
    return self.get_content()
  File "/usr/lib/ambari-agent/lib/resource_management/core/source.py", line 197, in get_content
    raise Fail("Failed to download file from {0} due to HTTP error: {1}".format(self.url, str(ex)))
resource_management.core.exceptions.Fail: Failed to download file from http://ambari1.org.cn:8080/resources/mysql-connector-java.jar due to HTTP error: HTTP Error 404: Not Found
```

于是访问报错中的页面：[http://ambari1.org.cn:8080/resources/](http://ambari1.org.cn:8080/resources/)，发现浏览器可以访问，但是确实没有我们想要的mysql-connector-java.jar包，于是进入服务器到相应的目录下：

```shell
[root@ambari1 centos7]# find / -name Ambari-DDL-SQLServer-DROP.sql
/var/lib/ambari-server/resources/Ambari-DDL-SQLServer-DROP.sql
[root@ambari1 centos7]# cd /var/lib/ambari-server/resources/
[root@ambari1 resources]# ll
total 604
-rwxr-xr-x  1 root root 101053 Dec  8 02:33 Ambari-DDL-AzureDB-CREATE.sql
-rwxr-xr-x  1 root root  83841 Dec  8 02:33 Ambari-DDL-MySQL-CREATE.sql
-rwxr-xr-x  1 root root   1192 Dec  8 02:33 Ambari-DDL-MySQL-DROP.sql
-rwxr-xr-x  1 root root  89531 Dec  8 02:33 Ambari-DDL-Oracle-CREATE.sql
-rwxr-xr-x  1 root root   2160 Dec  8 02:33 Ambari-DDL-Oracle-DROP.sql
-rwxr-xr-x  1 root root  83001 Dec  8 02:33 Ambari-DDL-Postgres-CREATE.sql
-rwxr-xr-x  1 root root   1337 Dec  8 02:33 Ambari-DDL-Postgres-DROP.sql
-rwxr-xr-x  1 root root   1233 Dec  8 02:33 Ambari-DDL-Postgres-EMBEDDED-CREATE.sql
-rwxr-xr-x  1 root root    827 Dec  8 02:33 Ambari-DDL-Postgres-EMBEDDED-DROP.sql
-rwxr-xr-x  1 root root  87927 Dec  8 02:33 Ambari-DDL-SQLAnywhere-CREATE.sql
-rwxr-xr-x  1 root root      0 Dec  8 02:33 Ambari-DDL-SQLAnywhere-DROP.sql
-rwxr-xr-x  1 root root   4215 Dec  8 02:33 Ambari-DDL-SQLServer-CREATELOCAL.sql
-rwxr-xr-x  1 root root  85101 Dec  8 02:33 Ambari-DDL-SQLServer-CREATE.sql
-rwxr-xr-x  1 root root   2117 Dec  8 02:33 Ambari-DDL-SQLServer-DROP.sql
-rwxr-xr-x  1 root root   6019 Dec  8 02:33 APACHE-AMBARI-MIB.txt
drwxr-xr-x 34 root root   4096 Apr  4 21:16 common-services
-rw-r--r--  1 root root  11932 Dec  8 02:33 CredentialUtil.jar
drwxr-xr-x  2 root root     43 Apr  4 21:16 custom_action_definitions
drwxr-xr-x  3 root root     53 Apr  4 21:19 custom_actions
-rw-r--r--  1 root root   1929 Dec  8 02:33 DBConnectionVerification.jar
drwxr-xr-x  2 root root    287 Apr  4 21:19 host_scripts
-rwxr-xr-x  1 root root   1991 Dec  8 02:33 kerberos.json
drwxr-xr-x  2 root root   4096 Apr  4 21:16 scripts
drwxr-xr-x  8 root root    166 Apr  4 21:19 stack-hooks
drwxr-xr-x  3 root root    266 Apr  4 21:16 stacks
drwxr-xr-x  4 root root     28 Apr  4 21:16 upgrade
-rwxr-xr-x  1 root root      8 Dec  8 02:33 version
drwxr-xr-x  3 root root    210 Apr  4 21:19 views
-rwxr-xr-x  1 root root   2558 Dec  8 02:33 widgets.json
```

然后将mysql-connector-java.jar上传，然后点击RETRY按钮，即可安装成功！

备注：此处安装选择的是新安装一个MySQL。
