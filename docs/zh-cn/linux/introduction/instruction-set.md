## 基本操作
 - ``cd /dir`` 进入dir目录、
 - ``cd ..`` 返回上一级
 - ``rm demo`` 删除demo文件
 - ``rm -r /dir`` 删除dir目录下所有文件
 - ``mkdir dir`` 创建dir目录
 - ``touch demo`` 创建demo文件
 - ``vim demo`` 编辑demo文件
 - ``cat demo`` 打印demo内容
 - ``tail -f catalina.out`` 持续打印catalina.out文件尾行
 - ``tail -n 100 catalina.out`` 打印catalina.out文件后100行
 - ``service nginx start`` 启动nginx服务
 - ``service nginx restart`` 重启nginx服务
 - ``service nginx stop`` 停止nginx服务
 - ``tar -zxf xxx.tar.gz`` 解压tar.gz文件

## 安装组件
 - ``yum list`` 查看yum安装包列表
 - ``yum install nginx`` 安装nginx
 - ``yum remove nginx`` 移除nginx
 - ``yum uninstall nginx`` 卸载nginx

## 系统管理
 - ``htop`` 查看当前cpu、内存等信息
 - ``iftop`` 查看当前网络情况
 - ``lsof -i tcp:80`` 查看80端口占用情况
 - ``netstat -ntlp`` 列出所有端口
 - ``netstat -lnp|grep 88`` 检查端口被哪个进程占用
 - ``kill -9 1777`` 杀掉编号为1777的进程

## 用户操作
 - ``adduser test`` 添加用户 test
 - ``passwd test`` 修改test密码
 - ``userdel test`` 删除用户test
 - ``userdel -r test`` 删除用户以及用户目录

## GIT操作
 - ``git clone https://github.com/ainilili/snail.git`` 将远程仓库的代码clone到本地
 - ``git status`` 查看当前仓库状态
 - ``git add test`` 添加test文件到仓库
 - ``git add * `` 添加所有文件到仓库
 - ``git commit -m 'first commit'`` 将add的文件添加注释并提交到仓库
 - ``git log`` 查看提交信息
 - ``git shortlog`` 将开发者操作按照姓名分组
 - ``git commit –amend -m 'modify first commit'`` 这里是追加的注释，会覆盖上次的注释
 - ``git remote add`` 将本地代码库提交到远程仓库
 - ``git push -u origin master`` 将本地master分支提交到远程的master分支，并关联起来
 - ``git pull –rebase`` 如果Apush修改前，B push了修改，A push的时候需要先从远程获取最新修改。这个指令不会产生过多的merge历史。
 - ``git checkout -b first`` 创建新分支，并且切换到该分支
 - ``git brach first`` 创建分支first
 - ``git checkout first`` 切换分支first
 - ``git branch`` 查看分支，-r显示所有远程分支，-a显示所有本地分支和远程分支
 - ``git merge first`` 合并first分支到master分支
 - ``git branch -d first`` 删除分支first，``-D``是强制删除
 - ``git remote origin`` 查看远程分支
 - ``git reset –hard HEAD~1`` 回退一个版本
 - ``git reset –hard HEAD~5`` 回退五个版本
 - ``git reset HEAD ReadMe.txt`` 文件从暂存区回退到工作区
