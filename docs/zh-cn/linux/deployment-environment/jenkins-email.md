## 背景
Jenkins如何在构建之后将构建状况发送到指定邮箱？
## 实现
首先，Jenkins要安装插件``Email Extension Plugin``

安装完成之后，进入``系统管理``->``系统设置``，开始设置！
### Jenkins Location
设置Jenkins Localtion：
![](https://github.com/ainilili/snail/blob/master/images/jenkins-email-1.jpg?raw=true)
 - **Jenkins URL**：Jenkins的地址
 - **System Admin e-mail address**：发送者的email

### Extended E-mail Notification
这里设置发送邮件相关的配置，如SMTP server和发送类型等。

首先点击``Advanced``显示高级配置：
![](https://github.com/ainilili/snail/blob/master/images/jenkins-email-2.jpg?raw=true)
然后填入必填项：
![](https://github.com/ainilili/snail/blob/master/images/jenkins-email-3.jpg?raw=true)
 - **SMTP server**：SMTP服务器地址
 - **Default user E-mail suffix**：默认的邮箱前缀
 - **User Name**：发送者邮箱
 - **Password**：发送者邮箱 ``POP3/STMP`` 授权密码（非真实密码）
 - **SMTP port**：SMTP服务器SSL端口
 - **Default Content Type**：默认内容类型
 - **Default Recipients**：默认收件人

配置完毕点保存
### 工程中运用
新建一个Jenkins Project，其他操作跟以往一样，可以附加邮件发送功能：
![](https://github.com/ainilili/snail/blob/master/images/jenkins-email-4.jpg?raw=true)
打开之后，很多选项都是使用的默认项，也就是我们在系统设置中配置的选项，在此基础上我们根据自己的需求改动即可：
![](https://github.com/ainilili/snail/blob/master/images/jenkins-email-5.jpg?raw=true)

## 附加
一个不错的``Default Content``模板：
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
</head>

<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"
    offset="0">
    <table width="95%" cellpadding="0" cellspacing="0"
        style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
        <tr>
            <td><br />
            <b><font color="#0B610B">构建信息</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>
        <tr>
            <td>
                <ul>
                    <li>项目名称 ： ${PROJECT_NAME}</li>
                    <li>构建编号 ： 第${BUILD_NUMBER}次构建</li>
                    <li>SVN 版本： ${SVN_REVISION}</li>
                    <li>触发原因： ${CAUSE}</li>
                    <li>构建日志： <a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                    <li>构建  Url ： <a href="${BUILD_URL}">${BUILD_URL}</a></li>
                    <li>工作目录 ： <a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                    <li>项目  Url ： <a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td><b><font color="#0B610B">变更集</font></b>
            <hr size="2" width="100%" align="center" /></td>
        </tr>

        <tr>
            <td>${JELLY_SCRIPT,template="html"}<br/>
            <hr size="2" width="100%" align="center" /></td>
        </tr>


    </table>
</body>
</html>
```
