# Linux-java

```
#java安装
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie"  http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz

tar xf jdk-8u131-linux-x64.tar.gz -C /usr/local/

mv /usr/local/jdk1.8.0_131/ /usr/local/java

vim /etc/profile
JAVA_HOME=/usr/local/java
PATH=$PATH:$JAVA_HOME/bin

ln -s /usr/local/java/bin/java /usr/bin/
```
