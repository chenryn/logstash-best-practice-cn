# 安装

## 下载

目前，Logstash 分为两个包：核心包和社区贡献包。你可以从 <http://www.elasticsearch.org/overview/elkdownloads/> 下载这两个包的源代码或者二进制版本。

* 源代码方式

```
wget https://download.elasticsearch.org/logstash/logstash/logstash-1.4.2.tar.gz
wget https://download.elasticsearch.org/logstash/logstash/logstash-contrib-1.4.2.tar.gz
```

* Debian 平台

```
wget https://download.elasticsearch.org/logstash/logstash/packages/debian/logstash_1.4.2-1-2c0f5a1_all.deb
wget https://download.elasticsearch.org/logstash/logstash/packages/debian/logstash-contrib_1.4.2-1-efd53ef_all.deb
```

* Redhat 平台

```
wget https://download.elasticsearch.org/logstash/logstash/packages/centos/logstash-1.4.2-1_2c0f5a1.noarch.rpm
https://download.elasticsearch.org/logstash/logstash/packages/centos/logstash-contrib-1.4.2-1_efd53ef.noarch.rpm
```

## 安装

上面这些包，你可能更偏向使用 `rpm`，`dpkg` 等软件包管理工具来安装 Logstash，开发者在软件包里预定义了一些依赖。比如，`logstash-1.4.2-1_2c0f5a.narch` 就依赖于 `jre` 包。

另外，软件包里还包含有一些很有用的脚本程序，比如 `/etc/init.d/logstash`。

如果你必须得在一些很老的操作系统上运行 Logstash，那你只能用源代码包部署了，记住要自己提前安装好 Java：

```
yum install openjdk-jre
export JAVA_HOME=/usr/java
tar zxvf logstash-1.4.2.tar.gz
```

## 最佳实践

但是真正的建议是：如果可以，请用 Elasticsearch 官方仓库来直接安装 Logstash！

### Debian 平台

```bash
wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | apt-key add -
cat >> /etc/apt/sources.list <<EOF
deb http://packages.elasticsearch.org/logstash/1.4/debian stable main
EOF
apt-get update
apt-get install logstash
```

### Redhat 平台

```
rpm --import http://packages.elasticsearch.org/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/logstash.repo <EOF
[logstash-1.4]
name=logstash repository for 1.4.x packages
baseurl=http://packages.elasticsearch.org/logstash/1.4/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1
EOF
yum clean all
yum install logstash
```
