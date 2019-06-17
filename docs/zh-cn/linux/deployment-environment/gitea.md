## 前置
### Git安装
```powershell
yum install git
```
### Mysql安装
请跳转[mysql安装方法](mysql.md)
## CentOS二进制安装
```powershell
wget -O gitea https://dl.gitea.io/gitea/1.3.2/gitea-1.3.2-linux-amd64
chmod +x gitea
```
## 启动
```powershell
./gitea web
```
## 设置
编辑安装目录下的``cusmer/conf/app.ini``文件：
### server配置
```powershell
[server]
SSH_DOMAIN       = 192.168.1.170 //ssh ip
DOMAIN           = 192.168.1.170 //web host
HTTP_PORT        = 3000          //web port
ROOT_URL         = http://192.168.1.170:3000/ //http/https拉取项目时的url前缀
DISABLE_SSH      = false                      //关闭ssh
SSH_PORT         = 6666                       //ssh端口，默认为22
LFS_START_SERVER = true
LFS_CONTENT_PATH = /data/gitea/data/lfs
LFS_JWT_SECRET   = emR-JPpJuh0m1kQ1OsVEWNmDO_Bq8nisU_AXqZPbmYA
OFFLINE_MODE     = false
START_SSH_SERVER = true                       //开启ssh认证模式
```
请改为自己的ip
### 邮件配置
```powershell
[mailer]
ENABLED = true
HOST    = xxx
FROM    = xxx
USER    = xx
PASSWD  = xxx                                 //代理密码
```
其他配置请看官网：[https://docs.gitea.io/zh-cn/](https://docs.gitea.io/zh-cn/)