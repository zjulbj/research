#说明：
使用过程中遇到任何问题，意见或者经验，请到评论区吐槽，以便完善进一步完善。

#客户端安装和使用
##快速入门
我们的工作环境一般是windows，这里只介绍windows下需要装的东西。

###安装gnu diff
rbt不带diff，所以需要安装一个gnu diff。下载安装包，跟着步骤走就可以了。
[http://gnuwin32.sourceforge.net/packages/diffutils.htm][1]
装完之后将bin目录添加到环境变量里面(如，`D:\System\GnuWin32\bin`）
打开cmd,输入`diff -version`检查是否可用:
```
C:\Users\Administrator>diff -version
diff (GNU diffutils) 2.8.7
Written by Paul Eggert, Mike Haertel, David Hayes,
Richard Stallman, and Len Tower.

Copyright (C) 2004 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
### 安装TortoiseSVN
下载地址：[https://tortoisesvn.net/downloads.html][2]，安装时勾选命令行工具，并将bin目录配置到path。
 
### 安装rbtools
也是一个安装包，跟着步骤走就可以了。
[https://www.reviewboard.org/downloads/rbtools/][3]
检查是否装好:
```
C:\Users\Administrator>rbt -v
RBTools 0.7.5 alpha 0 (dev)
```
### 配置仓库
客户端装好了，需要让工具知道用谁的帐号，去连哪个repo。
所以，先配置repo:
```
E:\svr\vivacommon\trunk>rbt setup-repo
Enter the Review Board server URL: 192.168.1.10
Use the Subversion repository "server" (https://192.168.1.6/svn/svr/)? [Yes/No]:
 Yes
Create "E:\svr\vivacommon\trunk\.reviewboardrc" with the following?

REVIEWBOARD_URL = "192.168.1.10"
REPOSITORY = "server"

 [Yes/No]: Yes
Config written to E:\svr\vivacommon\trunk\.reviewboardrc
```
在`trunk`下面新增一个`.reviewboardrc`文件
```
REVIEWBOARD_URL = "192.168.1.10"
REPOSITORY = "server"
```
这样rbt就知道向哪里提交了。

###登录
rbt需要以什么身份提交，就得先登录。
```
E:\svr\vivacommon\trunk>rbt login --username=USER_NAME --password=PASS_WORD
```
这里的是ReviewBoard注册的帐号和密码。一般登录一次之后就不用再登录了，切换帐号直接登录其它帐号就可以了。

###提交
修改trunk下面某个文件，然后向ReviewBoard发起提交申请
```
E:\svr\vivacommon\trunk>rbt post --summary="register add comment"
Review request #14 posted.

http://192.168.1.10/r/14/
http://192.168.1.10/r/14/diff/
```
这样ReviewBoard就新增了一次提交申请:
```
E:\svr\vivacommon\trunk>rbt status
 * r/14 - register add comment
   r/11 - English comment
   r/10 - this time good
   r/6 - first test
```
如果发现需要重新提交，可以`-r REQUEST_ID`更新id为REQUEST_ID的request。
```
E:\svr\vivacommon\trunk>rbt post -r 14
Review request #14 posted.

http://192.168.1.10/r/14/
http://192.168.1.10/r/14/diff/
```
这样就更新了request#14。
实际使用中，只想提交某个文件或者目录下面的修改，可以用-I指定参数。
这个参数可以指定多次来指定不同的文件或者目录
```
E:\svr\vivacommon\trunk>rbt post -I app/ -I pom.xml
```
排除某些文件可以用-X指定
```
E:\svr\vivacommon\trunk>rbt post -X *.txt
```
更多的用法，可以用`rbt help post`查看

###填写提交说明
上面的步骤只是向reviewboard提交了一个草稿，需要到reviewboard补充完整的说明。
![替代文字](https://wt-prj.oss.aliyuncs.com/317d90ca616448c3a802326948b584f0/73a44ab7-de7d-4b32-b7ed-b232e163652a.png)
上面红框部分是必须填写的。图中可以看出提交还是draft状态，其他人还看不到，在填写完提交说明之后，点击Publish会自动发送review邮件给group和people。

##Review流程
###review界面
收到review邮件后，打开review页面。
![替代文字](https://wt-prj.oss.aliyuncs.com/317d90ca616448c3a802326948b584f0/57ef4a57-503a-41ba-9644-d0a52a09e579.png)

- 1 点开diff 的tab，对代码review。
- 2 如果某行有问题，点击行号，对弹出评论框可以评论。
![替代文字](https://wt-prj.oss.aliyuncs.com/317d90ca616448c3a802326948b584f0/be4dad56-a33c-4d27-acf8-f27630de30ca.png)
- 3 勾选Open an Issue会创建一个issue。
- 4 点击Save,然后点击Publish会发布这个评论。
- 5 如果代码没什么问题了，就点击Ship it表示这个请求在你这里通过了。
- 6 如果reviewer都ship了，就可以提交到svn了
- 7 提交之后，别忘了修改review的状态(Close->Submitted)
![替代文字](https://wt-prj.oss.aliyuncs.com/317d90ca616448c3a802326948b584f0/73a38434-5887-452c-bfc4-6d4616ebcb3c.png)

##Pre-commit和PostCommit
review board和svn协作一般有下面两种工作流：pre-commit和post-commit。
pre-commit较为严格，要通过审查才能提交，post-commit较为宽松，提交后告诉reviewer。
![替代文字](https://wt-prj.oss.aliyuncs.com/317d90ca616448c3a802326948b584f0/4e5bb8ac-e991-4a17-8a87-dafe4dff4e2b.png)

##详细文档
更详细的使用方法，请参阅相关文档。

- rbtools：[https://www.reviewboard.org/docs/rbtools/0.7/][4]
- 使用文档：[https://www.reviewboard.org/docs/manual/2.5/users/][5]

##常见问题
###gnu diff 之后和svn diff命令冲突： 
由于之前安装tortoisesvn是安装了svn命令行，导致安装gnu diff 之后和svn diff命令冲突。 
解决方案：将tortoisesvn卸载之后重新安装就好了。 
###2.3.2 rbt post返回UnicodeDecodeError 
解决方案：在python安装目录修改Script/rbt-script.py文件，在代码中添加
```
if sys.getdefaultencoding() != 'gb18030':
    reload(sys)
    sys.setdefaultencoding('gb18030')
```
可以参考[http://www.111cn.net/phper/python/64627.htm][6]
###2.3.3 The current directory does not contain a checkout 
![替代文字](https://wt-prj.oss.aliyuncs.com/317d90ca616448c3a802326948b584f0/c1a532c6-8041-48e7-8d1c-3a54eb6c145c.png)
这个原因是rbt识别不出这个目录的svn 信息，解决方案：

- 1  确认是否安装svn客户端(windows下推荐TortoiseSVN),且配置到了path：
```
E:\svr\vivasam\trunk>svn
Type 'svn help' for usage.
```
- 2 确认当前仓库的状态是否正常，不正常用svn cleanup恢复正常
```
E:\svr\vivasam\trunk>svn status
?       app\accountapi\a.diff
?       app\demoapi
?       doc\test_stress\out
?       rpc\gen.bat
```
###2.3.4 "review.xiaoying.co"访问不了
这是因为我们的review borad部署在内网机器，没有绑定域名。
修改host文件，将其指向"192.168.1.10"就可以了

#3 相关文档
[https://www.reviewboard.org/docs/][7]


  [1]: http://gnuwin32.sourceforge.net/packages/diffutils.htm
  [2]: https://tortoisesvn.net/downloads.html
  [3]: https://www.reviewboard.org/downloads/rbtools/
  [4]: https://www.reviewboard.org/docs/rbtools/0.7/
  [5]: https://www.reviewboard.org/docs/manual/2.5/users/
  [6]: http://www.111cn.net/phper/python/64627.htm
  [7]: https://www.reviewboard.org/docs/
