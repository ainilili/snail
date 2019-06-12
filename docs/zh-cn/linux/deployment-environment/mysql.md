## 下载yum源
```powershell
wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
```
## 安装yum源
```powershell
yum localinstall mysql57-community-release-el7-8.noarch.rpm
```
## 检测
```powershell
yum repolist enabled | grep "mysql.*-community.*"
```
## 安装
```powershell
yum install -y mysql-community-server
```
## 启动
```powershell
systemctl start mysqld
```
