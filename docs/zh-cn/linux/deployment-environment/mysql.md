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
安装过程会有点漫长，请耐心等待。
## 启动
```powershell
systemctl start mysqld
```
## 获取初始密码
```powershell
grep 'temporary password' /var/log/mysqld.log
```
## 关闭密码校验策略（可跳过）
mysql5.7新增了密码策略，默认策略为：必须包含大小写字母、数字和特殊符号，并且长度不能少于8位，解决办法是在``/etc/my.cnf``文件中添加如下配置禁用即可关闭密码策略：``validate_password = off``
```powershell
vim /etc/my.cnf
```
## 修改密码
```powershell
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
```
## 修改编码
``/etc/my.cnf``增加:
```powershell
character_set_server=utf8
```

