# gitlab+jenkins

```
#gitlab部署
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
yum install gitlab-ce-13.2.1-ce.0.el7.x86_64 -y

# 修改配置文件
vi /etc/gitlab/gitlab.rb
external_url 'http://192.168.0.71'
```

```
#jenkins部署
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins

#修改配置文件
vim /etc/sysconfig/jenkins
JENKINS_HOME="/var/lib/jenkins" ### 如果修改jenkins的工作路径，只需要修改此处就好 ，默认的工作目录是 /var/lib/jenkins
JENKINS_JAVA_CMD=""
JENKINS_USER="root" ### 默认的jenkins用户是jenkins，修改jenkins的用户名为root
JENKINS_PORT="8080" ### 修改jenkins的默认端口 
```
