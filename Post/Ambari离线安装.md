# Ambari离线安装

Ambari的离线安装介绍文档。

## 系统版本

### 操作系统

CentOS 7.4.1780

### Ambari版本

Ambari版本是从官网下载下来的2.7.3.0版本：

[下载链接](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/ambari_repositories.html)

### HDP版本

HDP版本是从官网下载下来的3.1.0.0的版本：

[下载链接](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/hdp_31_repositories.html)

### HDP-UTILS版本

HDP-UTILS版本同样是从官网下载下来的1.1.0.22版本的。下载链接同HDP一样。

### JDK版本

安装JDK，建议版本1.8+，安装教程请参照Java中的相关篇章。

### PostgreSQL

数据库选择的是Ambari内置的数据库，当然了，也可以选择自己的MySQL作为自己的数据库。

在安装Ambari之前做的准备。

## 安装软件

### 安装JDK

在所有服务器中安装JDK，安装过程请查看Java相关篇幅，此处不再赘述。

### 安装NTP

在所有服务器中安装NTP这一时间同步软件。

1）在每台服务器上安装ntp，用于矫正每台服务器的时间。直接使用shell命令：yum -y install ntp。

2）在集群机器上运行命令：systemctl enable ntpd

3）vi /etc/ntp.conf

到server 3.centos.pool.ntp.org iburst这行下面添加以下两行内容并保存：

```bash
server 127.127.1.0
fudge 127.127.1.0 stratum 8
```

4）如果发现时间的时区不是CST，执行如下命令：

```bash
$ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ hwclock
```

5）重启ntp服务

```bash
$ systemctl restart ntpd
```

### 安装createrepo

在主服务器上安装createrepo，安装命令如下：

```bash
$ yum install createrepo -y
```

### 安装yum-utils

在主服务器上安装yum-utils如下：

```bash
$ yum install yum-utils -y
```

### 安装httpd

在主服务器上安装httpd是为了能够制作局域网通用的yum本地资源库。

```bash
$ yum install -y httpd   # 安装httpd
$ service httpd start   # 启动httpd
$ systemctl enable httpd   # 设置httpd为开机启动
```

## 环境配置部分

### 系统版本

在安装Ambari之前需要确认每一台主机的系统版本严格一致，命令如下：

```bash
[root@ambari1 ~]# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core)
```

### 主机名修改

1）为每台服务器设定一个独立的主机名，便于ambari-server识别。可在shell中输入hostname -f检查FQDN全域名。

2）如果返回的全域名为你设定过的名字，则不用修改全域名。如果返回值为例如localhost，或者localhost.localdomain，则建议修改。因为可能有其他主机使用同一个名字，安装集群时会出错！

3）设定FQDN全域名方法如下：

在每台主机shell中输入命令vi /etc/hosts，把每台服务器的ip，域名保存在该文件。

FQDN格式为：ip地址 FQDN全域名 主机名。

大致格式如下所示：

```bash
192.168.0.219 master.org.cn master
192.168.0.220 agent1.org.cn agent1
192.168.0.223 agent2.org.cn agent2
```

修改完一台服务器的FQDN之后，将/etc/hosts文件分发到其他的服务器。请记得把所有服务器的FQDN信息都要填充上，包括本服务器的，再使用hostname -f进行检查全域名是否成功设置。

### 关闭防火墙

每台服务器都需要关闭防火墙并禁用：

```bash
$ systemctl stop firewalld # 关闭防火墙
$ systemctl disable firewalld # 禁用防火墙
```

### 关闭selinux

关闭集群每一台服务器的selinux服务，让服务器之间能够互相连通。

命令：

```bash
$ vi /etc/sysconfig/selinux
```

修改内容如下：

```bash
[root@ambari1 ~]# cat /etc/sysconfig/selinux
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled # 此处修改为disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

当每一台服务器都设置好selinux之后，记得全部重启一下。

### 光盘挂载

#### 挂载

光盘挂载是需要将和系统版本一致的镜像挂载到相关的目录下：

```bash
$ mkdir /media/CentOS # 创建目录
$ mount -o loop /bigdata-soft/CentOS-7-x86_64-DVD-1708.iso  /media/CentOS # 挂载镜像
```

#### 修改repo文件

关盘挂载完了之后，需要对仓库中的文件进行配置，相关文件在目录/etc/yum.repos.d下：

1）修改CentOS-Media.repo文件内容，将enable=0修改为enable=1，保存！

2）mv CentOS-Base.repo CentOS-Base.repo.bak

3）清理，缓存：

```bash
$ yum clean all
$ yum makecache
$ yum repolist
```

### ssh免密设置

公钥用于ambari-server免钥链接各个client服务器。

1）生成每个服务器的公钥/私钥

```bash
$ ssh-keygen -t rsa
```

输入上述命令之后，会提示输入相应的信息，这里全部不输入，直接回车一直到结束为止！

2）创建authorized_keys

在安装ambari-server的服务器上执行：

```bash
$ cd /root/.ssh/
$ cat id_rsa.pub >> authorized_keys
$ chmod 600 /root/.ssh/authorized_keys
```

3）执行命令，将authorized_keys传输到每一个服务器的/root/.ssh/目录下

```bash
scp -r ~/.ssh/authorized_keys <server-ip>:/root/.ssh/
```

4）从ambari-server的服务器执行命令ssh username>@server-ip联通到每台服务器之后，在shell中输入命令chmod 600 ~/.ssh/authorized_keys

5）在shell中输入exit返回本机，再尝试ssh username>@server-ip，查看是否需要输入密码。如果不需要说明免钥联通成功。（此处配置的免密码登陆是单向的）

## 制作本地仓库

### 下载软件包

从[网站地址](https://hortonworks.com/downloads/#data-platform)下载如下安装包：

```bash
[root@master ambari2.7.3.0+hdp3.1.0.0]# ll
total 10845148
-rw-r--r--. 1 root root 1947685893 Nov  4 16:18 ambari-2.7.3.0-centos7.tar.gz
-rw-r--r--. 1 root root 9066967592 Nov  4 16:41 HDP-3.1.0.0-centos7-rpm.tar.gz
-rw-r--r--. 1 root root     162100 Nov  4 16:15 HDP-GPL-3.1.0.0-centos7-gpl.tar.gz
-rw-r--r--. 1 root root   90606616 Nov  4 16:15 HDP-UTILS-1.1.0.22-centos7.tar.gz
```

### 制作本地仓库

1）httpd默认本地源头目录在/var/www/html目录下，所以我们先创建建/var/www/html/ambari目录：

```bash
$ mkdir /var/www/html/ambari
```

2）将上述的压缩包解压到ambari目录下：

```bash
$ tar zxf HDP-UTILS-1.1.0.22-centos7.tar.gz -C /var/www/html/ambari/
$ tar zxf HDP-3.1.0.0-centos7-rpm.tar.gz -C /var/www/html/ambari/
$ tar zxf HDP-GPL-3.1.0.0-centos7-gpl.tar.gz -C /var/www/html/ambari/
$ tar zxf ambari-2.7.3.0-centos7.tar.gz -C /var/www/html/ambari/
```

注意：如果HDP-UTILS解压出来没有一个HDP-UTILS文件夹作为包装，就需要在/var/www/html/ambari下面先创建HDP-UTILS文件夹，然后再执行解压缩命令。

3）执行命令createrepo /var/www/html/ambari/

4）在/etc/yum.repos.d目录下创建资源文件：

ambari.repo内容如下：

```bash
[Updates-ambari-2.7.3.0]
name=ambari-2.7.3.0 - Updates
baseurl=http://192.168.0.219/ambari/ambari/centos7/2.7.3.0-139/
gpgcheck=1
gpgkey=http://192.168.0.219/ambari/ambari/centos7/2.7.3.0-139/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
```

HDP.repo内容如下：

```bash
[HDP-3.1.0.0]
name=HDP Version - HDP-3.1.0.0
baseurl=http://192.168.0.219/ambari/HDP/centos7/3.1.0.0-78
gpgcheck=1
gpgkey=http://192.168.0.219/ambari/HDP/centos7/3.1.0.0-78/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
```

HDP-UTILS.repo内容如下：

```bash
[HDP-UTILS-1.1.0.22]
name=HDP Utils Version - HDP-UTILS-1.1.0.22
baseurl=http://192.168.0.219/ambari/HDP-UTILS/centos7/1.1.0.22/
gpgcheck=1
gpgkey=http://192.168.0.219/ambari/HDP-UTILS/centos7/1.1.0.22/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
```

HDP-GPL.repo内容如下：

```bash
[HDP-GPL]
name=HDP-GPL
baseurl=http://192.168.0.219/ambari/HDP-GPL/centos7/3.1.0.0-78/
gpgcheck=1
gpgkey=http://192.168.0.219/ambari/HDP-GPL/centos7/3.1.0.0-78/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
```

备注：因为apache httpd默认的文件夹是/var/www/html，所以这里的baseurl目录根据/var/www/html下的来写，比如ambari.repo的baseurl为192.168.0.219/ambari/HDP/centos7/3.1.0.0-78，并且还可以将HDP.repo、HDP-GPL.repo以及HDP-UTILS.repo写成一个repo文件，比如如下形式：

```bash
[root@master yum.repos.d]# cat hdp.repo
[HDP-3.1.0.0]
name=HDP Version - HDP-3.1.0.0
baseurl=http://192.168.0.219/ambari/HDP/centos7/3.1.0.0-78
gpgcheck=1
gpgkey=http://192.168.0.219/ambari/HDP/centos7/3.1.0.0-78/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
[HDP-UTILS-1.1.0.22]
name=HDP Utils Version - HDP-UTILS-1.1.0.22
baseurl=http://192.168.0.219/ambari/HDP-UTILS/centos7/1.1.0.22/
gpgcheck=1
gpgkey=http://192.168.0.219/ambari/HDP-UTILS/centos7/1.1.0.22/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
[HDP-GPL]
name=HDP-GPL
baseurl=http://192.168.0.219/ambari/HDP-GPL/centos7/3.1.0.0-78/
gpgcheck=1
gpgkey=http://192.168.0.219/ambari/HDP-GPL/centos7/3.1.0.0-78/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
```

本地仓库制作完成后，执行如下命令：

```bash
$ yum clean all
$ yum makecache
$ yum repolist
```

注意：ambari.repo创建完成即可，hdp.repo可以不用创建。

## Ambari安装和配置

### 安装

```bash
yum install ambari-server -y
```

### 配置

```bash
[root@master ~]# ambari-server setup
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? n
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Custom JDK
==============================================================================
Enter choice (1): 2 # 这里选择第二选项，表示使用我们自己的JDK
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /usr/local/jdk1.8.0_231 # 填写JDK的安装目录
Validating JDK on Ambari Server...done.
Check JDK version for Ambari Server...
JDK version found: 8
Minimum JDK version is 8 for Ambari. Skipping to setup different JDK for Ambari Server.
Checking GPL software agreement...
GPL License for LZO: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)? n # 这是新版本的功能，这里暂时不开启
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? y # 选择配置数据库
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL / MariaDB
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
[7] - BDB
==============================================================================
Enter choice (1): 1 # 这里选择1，选择Ambari内置的数据库，数据库类型为PostgreSQL
Database admin user (postgres): # 默认回车
Database name (ambari): # 默认回车
Postgres schema (ambari): # 默认回车
Username (ambari): # 默认回车
Enter Database Password (bigdata): # 默认回车
Default properties detected. Using built-in database.
Configuring ambari database...
Checking PostgreSQL...
Running initdb: This may take up to a minute.
Initializing database ... OK


About to start PostgreSQL
Configuring local database...
Configuring PostgreSQL...
Restarting PostgreSQL
Creating schema and user...
done.
Creating tables...
done.
Extracting system views...
ambari-admin-2.7.3.0.139.jar
....
Ambari repo file doesn't contain latest json url, skipping repoinfos modification
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```

### 启动Ambari

```bash
ambari-server start
```

启动成功之后，在windows本地的hosts文件中加入集群信息即可。

默认登陆密码：admin/admin
