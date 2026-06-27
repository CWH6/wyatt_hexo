---
title: 【SVN】安装使用与IDEA集成
date: 2023-05-22 11:35:48
tags:
  - SVN
category: 
  - 其他
---



## SVN介绍

### 简介

 SVN全称Subversion，是一个开放源代码的版本控制系统，Subversion 在 2000 年由 CollabNet Inc 开发现在发展成为 Apache 软件基金会的一个项目，同样是一个丰富的开发者和用户社区的一部分。

SVN是一个开放源代码的**版本控制系统**，管理着随时间改变的数据。这些数据放置在一个中央资料档案库(repository)中。这个档案库很像一个普通的文件服务器,不过它会记住每一次文件的变动。这样你就可以把档案恢复到旧的版本,或是浏览文件的变动历史。说得简单一点SVN就是**用于多个人共同开发同一个项目，共用资源的目的**。

### 基本概念

- Repository(源代码库): 源代码统一存放的地方
- Checkout (提取): 当你手上没有源代码的时候，你需要从repository checkout一份。
- Commit (提交): 当你已经修改了代码，你就需要Commit到repository
- Update(更新): 当你已经Checkout了一份源代码，Update后就可以和Repository上的源代码同步

### 工作流程

开始新一天的工作

1、从服务器**下载项目组最新代码**。(Checkout)

2、如果已经Checkout并且有人已Commit了代码，你可以**更新以获得最新代码**。 (Update)

3、**进入自己的分支，进行工作，每隔一个小时向服务器自己的分支提交一次代码**(很多人都有这个习惯。因为有时候自己对代码改来改去，最后又想还原到前一个小时的版本，或者看看前一个小时自己修改了哪些代码，就需要这样做了)(Commit)

4、**下班时间快到了，把自己的分支合并到服务器主分支上，一天的工作完成，并反映给服务器**(Commit)注意:如果两个程序员同时修改了同一个文件，SVN可以合并这两个程序员的改动，实际上SVN管理源代码是以行为单位的，就是说两个程序员只要不是修改了同一行程序，SVN都会自动合并两种修改。**如果是同一行，SVN会提示文件Confict冲突，需要手动确认**。



## 安装配置

### 服务端

#### 下载

https://www.visualsvn.com/downloads/

[![image-20230522204354906](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522204354906.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522204354906.png)

[![image-20230522204917293](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522204917293.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522204917293.png)image-20230522204917293

#### 安装

1.双击安装程序 VisualSVN-Server-5.1.4-x64.msi

[![image-20230522205101089](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522205101089.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522205101089.png)

2、勾选复选框选择同意，然后选择 Next，选择 Upgrade

[![image-20230522205353582](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522205353582.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522205353582.png)

3、选择默认配置，选择Next

[![image-20230522205452579](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522205452579.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522205452579.png)

[![image-20230522210245679](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210245679.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210245679.png)

[![image-20230522210351823](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210351823.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210351823.png)

[![image-20230522210638368](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210638368.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210638368.png)

[![image-20230522210757861](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210757861.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210757861.png)

安装完成如下图：

[![image-20230522210946358](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210946358.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522210946358.png)

#### 配置

说明:服务器端需要提供IP、端口、帐号、密码供客户端使用。即有如下配置

##### 设置IP和端口

1.打开服务器，点击 VisualSVN Server，选择 Configure authentication options...

[![image-20230522212218490](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522212218490.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522212218490.png)

2.设置Server name，建议使用当前IP

[![image-20230522212334979](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522212334979.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522212334979.png)

注意

> Server name的值可以设置为: 1、127.0.0.1 (只能本地自己访问)
>
> 2、电脑用户名 (只能本地自己访问)
>
> 3、当前IP(能够拼通IP的用户均可访问,IPV4) Server Port使用默认值即可
>
> 查看当前IP: 打开dos窗口(windows+R键)，输入ipconfig，按回车

##### 新建账号密码

1、右键左侧菜单 User , 选择Create User

[![image-20230522212811999](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522212811999.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522212811999.png)

2、设置用户的账户和密码

[![image-20230522212919581](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522212919581.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522212919581.png)

##### 新建分组

1.选择 Group 右键，选择 Create Group...

[![image-20230522213041559](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213041559.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213041559.png)

2.设置分组名称，及为分组添加用户

[![image-20230522213135572](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213135572.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213135572.png)image-20230522213135572

##### 查看资源

[![image-20230522213315347](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213315347.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213315347.png)

#### 使用

##### 新建版本库

1.选择 Repositories 右键，选择 Create New Repository...

[![image-20230522213459330](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213459330.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213459330.png)

2.选择默认设置，选择下一步，设置仓库名称

[![image-20230522213629863](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213629863.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213629863.png)

[![image-20230522213706347](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213706347.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213706347.png)

3.设置仓库目录(选择任意一个选项都可)

[![image-20230522213752238](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213752238.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213752238.png)

4.设置仓库的访问权限(速里设置所有svn用户都有读/写权限)

[![image-20230522213848218](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213848218.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213848218.png)

5.仓库创建完成

[![image-20230522213955491](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213955491.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522213955491.png)

##### 签入项目到SVN（import）

1、拷贝远程仓库的地址

[![image-20230522214852649](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522214852649.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522214852649.png)

2、选择任意项目，右键选择TortoiseSVN，选择import

[![image-20230522214957475](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522214957475.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522214957475.png)image-20230522214957475

3.将上一步拷贝的仓库地址粘贴到地址栏

[![image-20230522215033161](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215033161.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215033161.png)

 4.选择永久接受

[![image-20230522215137657](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215137657.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215137657.png)

```
5. 输入用户账号和密码
```

[![image-20230522215237762](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215237762.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215237762.png)image-20230522215237762

6、导入成功

[![image-20230522215305062](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215305062.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215305062.png)

7.仓库右键，选择刷新，在服务器中看到的效果

[![image-20230522215353956](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215353956.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215353956.png)

### 客户端

#### 下载

https://tortoisesvn.net/downloads.html

[![image-20230522204719713](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522204719713.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522204719713.png)

[![image-20230522205019891](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522205019891.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522205019891.png)image-20230522205019891

#### 安装

1、双击安装程序 TortoiseSVN-1.13.1.28686-x64-svn-1.13.0.msi

[![image-20230522211114120](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211114120.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211114120.png)

2、选择Next，然后勾选 command line client tools，选择 Next

[![image-20230522211303818](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211303818.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211303818.png)

3.设置TortoiseSVN的安装路径，勾选command line client tools ， **IDEA集成SVN需要这个工具**

[![image-20230522211613991](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211613991.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211613991.png)

4进入到TortoiseSVN软件使用协议界面，直接选择Install 进行安装

[![image-20230522211703563](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211703563.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211703563.png)

5.安装完成之后，直接选择 Finish 即可

[![image-20230522211803986](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211803986.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211803986.png)

6.在任意空白地方，右键，出现如下内容，则表示安装成功

[![image-20230522211854341](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211854341.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522211854341.png)

#### 使用

##### 检索项目（check out）

1.复制要下载的项目的远程地址

[![image-20230522215758792](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215758792.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522215758792.png)

2.在需要检索项目的目录中，右键选择 SVN Checkout..

[![image-20230522220230087](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220230087.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220230087.png)image-20230522220230087

3.输入远程地址，设置项目的存放位置

[![image-20230522220328062](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220328062.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220328062.png)

4、检索完成

[![image-20230522220412158](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220412158.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220412158.png)

##### 提交代码 (commit)

1.新建文件，右键选择 TortoiseSVN，选择 Add，将文件添加到版本库列表

[![image-20230522220536791](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220536791.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220536791.png)

2.再次点击文件，右键，会出现 SVN Commit...

[![image-20230522220831071](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220831071.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220831071.png)

3.提交成功

[![image-20230522220913444](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220913444.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522220913444.png)

##### 更新代码（update）

1.如果当前资源不是最新版本，则可在项目中空白地方右键，选择SVN Update

[![image-20230522221111551](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522221111551.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522221111551.png)

2、更新成功

[![image-20230522221153937](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522221153937.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522221153937.png)

##### 版本冲突问题

**原因**

假设A、B两个用户都在版本号为100的时候，更新了kingtuns.txt这个文件，A用户在修改完成之后提交kingtuns.txt到服务器，这个时候提交成功，这个时候kingtuns.txt文件的版本号已经变成101了。同时B用户在版本号为100的kingtuns.txt文件上作修改，修改完成之后提交到服务器时，由于不是在当前最新的101版本上作的修改，所以导致提交失败。此时用户B去更新文件，**如果B用户和A用户修改了文件的同一行代码，就会出现冲突**

**版本冲突现象**

冲突发生时，subversion会在当前工作目录中保存所有的目标文件版本[上次更新版本、当前获取的版本(即别 人提交的版本)、自己更新的版本、目标文件]。

假设文件名是kingtuns.txt 对应的文件名分别是: kingtuns.txt.r101 kingtuns.txt.r102 kingtuns.txt.mine kingtuns.txt 同时在目标文件中标记来自不同用户的更改

**版本冲突解决**

场景

1.现在A、B两个用户都更新项目文件到本地

用户A

[![image-20230522221739948](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522221739948.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522221739948.png)

用户B

[![image-20230522221901159](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522221901159.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522221901159.png)

2.项目中的hello.txt 文件原始内容为

[![image-20230522222142644](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222142644.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222142644.png)image-20230522222142644

3.A用户修改文件，添加内容“A用户修改内容”，完成后提交到服务器

[![image-20230522222213959](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222213959.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222213959.png)image-20230522222213959

[![image-20230522222817620](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222817620.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222817620.png)image-20230522222817620

4、B用户修改文件，添加内容“B用户修改内容”，完成后提交到服务器

[![image-20230522222845428](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222845428.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222845428.png)image-20230522222845428

5、B用户提交更新至服务器时提示如下

[![image-20230522222915691](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222915691.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522222915691.png)image-20230522222915691

6、B用户将文件提交至服务器时，提示版本过期:首先应该从版本库更新版本，然后去解决冲突，冲突解决后要执行svn resolved(解决)，然后在签入到版本库。在冲突解决之后，需要使用svn resolved (解来告诉subversion冲突解决，这样才能提交更新。

**解决冲突的三种选择**

1.**放弃自己的更新**，使用svn revert (回滚)，然后提交。在这种方式下不需要使用svn resolved (解决)

2.**放奔自己的更新**，使用别人的更新。使用最新获取的版本覆盖目标文件，执行resolved filename并提交(选择 文件一右键一解决)

3.**手动解决**:冲突发生时，通过和其他用户沟通之后，手动更新目标文件。然后执行resolved filename来解阶 冲突，最后提交。

**解决冲突实践**

1.在B用户当前目录下，右键选择”SVN Update”，执行“update”(更新)操作

[![image-20230522225305080](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522225305080.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522225305080.png)

2.B用户中的 Hello.txt 文件出现冲突

[![image-20230522225354112](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522225354112.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522225354112.png)

3、在冲突的文件上（选中文件---右键菜单-TortoiseSVN--Edit conflicts（解决冲突））

[![image-20230522225653364](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522225653364.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522225653364.png)

4、打开编辑冲突的窗口

[![image-20230522225754028](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522225754028.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522225754028.png)

```
Theirs窗口为服务器上当前最新版本
Mine窗口为本地修改后的版本
Merged窗口为合并后的文件内容显示
```

5.如果要使用服务器版本，在Theirs窗口选中差异内容，右键，选择Use this text block (使用这段文本块) 同理如果要使用本地版本，在协商后，在Mine窗口右键，选择Use this text block (使用这段文本块)。

[![image-20230522230024832](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230024832.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230024832.png)image-20230522230024832

6.修改完成后，选择"Markas resolved”(标记为解决)，然后选择Save”(保存文件)，关闭窗口即可

[![image-20230522230105301](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230105301.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230105301.png)image-20230522230105301

7.此时，当前冲突已解决，可再次选择“SVN Commit”提交文件

[![image-20230522230136137](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230136137.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230136137.png)image-20230522230136137

> 注:也可先不标记为解决，直接保存文件后,在B用户的冲突目录下，选中文件一右键菜单-TortoiseSVN-Resolved (解决)。然后再提交文件。

**如何降低冲突解决的复杂度** 1.当文档编辑完成后，尽快提交，频繁的提交/更新可以降低在冲突发生的概率，以及发生时解决冲突的复杂度。

2.在提交时，写上明确的message，方便以后查找用户更新的原因，毕竟随着时间的推移，对当初更新的原因有可能会遗忘

3.养成良好的使用习惯，使用SVN时每次都是先提交，后更新。每天早上打开后，首先要从版本库获取最新版本。每天下班前必须将已经编辑过的文档都提交到版本库。

## IDEA 集成使用SVN

### 配置SVN环境

1、File—>Ohter Setting (全局配置；Setting是局部配置)—> Version Control —>Subversion

[![image-20230522230447902](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230447902.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230447902.png)image-20230522230447902

2、配置svn

[![image-20230522230554226](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230554226.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522230554226.png)image-20230522230554226

> 找不到svn.exe文件，TortoisesVN的bin目录下面没有svdl exe 之所以没有是因为安装TortoiseSVN的时候没有选指定安装项，添加command line client tools

3、重启IDEA

### IDEA Git转SVN

选择setting-> Version Control，新增或者编辑，选择目录，使用git或者svn管理

[![image-20230522231534938](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522231534938.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522231534938.png)image-20230522231534938

#### 检索项目

1.选择VCS -> Checkout from Version Control-> Subversion

[![image-20230522233637638](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233637638.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233637638.png)image-20230522233637638

2.添加远程仓库中项目的URL

[![image-20230522233740071](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233740071.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233740071.png)image-20230522233740071

3.点击添加的URL，选择 Checkout

[![image-20230522233808432](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233808432.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233808432.png)image-20230522233808432

4.选择检索的项目的存放位置

[![image-20230522233846111](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233846111.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233846111.png)image-20230522233846111

5.选择Destination，根据自己的偏好选择，其他配置默认，单击 OK

[![image-20230522233917799](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233917799.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233917799.png)image-20230522233917799

6.选择1.8 Format，点击OK

[![image-20230522233943402](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233943402.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522233943402.png)image-20230522233943402

7.已经check out一个项目，是否要打开，选择Yes

[![image-20230522234008123](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234008123.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234008123.png)image-20230522234008123

8.选择Add

[![image-20230522234041413](file:///C:/Users/27477/AppData/Roaming/Typora/typora-user-images/image-20230522234041413.png)](file:///C:/Users/27477/AppData/Roaming/Typora/typora-user-images/image-20230522234041413.png)image-20230522234041413

9.此时，项目就可以与远程仓库关联

[![image-20230522234111205](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234111205.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234111205.png)image-20230522234111205

#### 提交代码

1.选择VCS->Commit..

[![image-20230522234143179](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234143179.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234143179.png)image-20230522234143179

2.选择需要提交的文件，填写提交信息，选择Commit

[![image-20230522234226355](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234226355.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234226355.png)

3.提交成功后，会在ldea最下面显示提交状态

[![image-20230522234301427](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234301427.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234301427.png)image-20230522234301427

> 注:项目提交前，最好先更新，然后再提交

#### 更新代码

1.选择VCS -> Update Project...

[![image-20230522234420748](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234420748.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234420748.png)image-20230522234420748

2、默认即可，直接选择OK

[![image-20230522234451997](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234451997.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234451997.png)image-20230522234451997

3.更新成功的提示信息

[![image-20230522234520710](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234520710.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234520710.png)image-20230522234520710

#### 导入项目

1.选择VCS->Import into Version Control ->lmport into Subversion

[![image-20230522234557749](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234557749.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234557749.png)image-20230522234557749

2.选择“+"添加项目导入的地址(可手动添加一个文件夹，项目中的文件会放置在该文件夹中，文件名自定义

[![image-20230522234629943](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234629943.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234629943.png)image-20230522234629943

3选择要导入的路径，选择Import

[![image-20230522234709081](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234709081.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234709081.png)image-20230522234709081

4.选择要导入的项目，点击OK

[![image-20230522234755490](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234755490.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234755490.png)image-20230522234755490

5.检查导入的路径，填写导入信息，选择OK

[![image-20230522234830810](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234830810.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522234830810.png)image-20230522234830810

6.在远程仓库中检查是否导入成功即可

#### 版本冲突问题

1.如果未更新，就提交资源(有其他用户也提交过资源)，则提交失败

[![image-20230522235208687](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235208687.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235208687.png)image-20230522235208687

2.此时，执行更新操作，将其他人提交过的资源更新到本地，会提示冲突

[![image-20230522235240406](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235240406.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235240406.png)image-20230522235240406

3.通常选择合并，再选择需要保留的代码，选择好之后选择 Apply

[![image-20230522235344016](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235344016.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235344016.png)

4.提示更新成功

[![image-20230522235410787](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235410787.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235410787.png)image-20230522235410787

5.再次选择提交，成功解决冲突

[![image-20230522235429863](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235429863.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230522235429863.png)image-20230522235429863
