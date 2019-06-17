## 背景
CentOS中yum默认的git版本为``1.7.1``，我想搞到``2.2.1``
## 前置
安装库
```powershell
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
yum install  gcc perl-ExtUtils-MakeMaker   
yum install install autoconf automake libtool
```
## 安装git
```powershell
wget https://github.com/git/git/archive/v2.2.1.tar.gz
tar zxvf v2.2.1.tar.gz
cd git-2.2.1
make configure
./configure --prefix=/usr/local/git --with-iconv=/usr/local/libiconv
make all doc
make install install-doc install-html
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
source /etc/bashrc
```
## 验证
```powershell
[root@localhost log]# git version
git version 2.2.1
```
