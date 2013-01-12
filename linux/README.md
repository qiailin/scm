redHat下配置svn服务器  


=============

1.准备 

  svnbook（比较详细的svn文档）http://svnbook.red-bean.com/ 

 

  安装包下载地址 http://subversion.tigris.org/downloads/subversion-1.6.6.tar.gz 

  依赖包下载地址 http://subversion.tigris.org/downloads/subversion-deps-1.6.6.tar.gz 

 

  subversion 可以用两种服务器可以使用svnserve（自带的）也可以使用apache，svnserve配置简单，速度快，所以在这里使用。apache也配置过，但没有成功，以后试验成功后再另写文档。 

 

2.解压缩： 

  把安装包和依赖包放在同一目录下，执行以下命令解压缩 

  tar –xvf subversion-1.6.6.tar.gz 

  tar –xvf subversion-deps-1.6.6.tar.gz 

  两个压缩包解压后会在同一个目录下，目录名称叫subversion-1.6.6，里边的INSTALL是安装说明文件。 

 

 

3.安装 

  a.依赖包介绍和安装 

svnserve依赖包包括libarp libapr-util sqlite libz等(其中libarp是Apache portable Run-time libraries,Apache可移植运行库)。

以上依赖包都在subversion-deps-1.6.6.tar.gz中，解压缩到安装包同一目录下，安装时自动安装，不需要单独安装。 

subversion需要openssl，下载的依赖包里没有，安装方法是打开Yast2->软件管理，勾选openssl、openssl-devel和openssl-doc，插入suse安装光盘，点击接受即可。

如果不安装openssl和openssl-devel，运行./configure会报错： 

    configure: error: We require OpenSSL; try --with-openssl 

    configure failed for serf 

  b.安装 

    由于不使用apache做服务器，所以跳过apache的安装。 

    进入subversion-1.6.6目录 

    $ ./configure 

    $ make 

    # make install  

4. svn配置 

建立版本库目录，可建多个： 

mkdir -p /opt/svn/svndata/

mkdir -p /opt/svn/svndata/ 

建立版本库: 

svnadmin create /opt/svndata/repos1 

svnadmin create /opt/svndata/repos2 

修改版本库配置文件: 

版本库1： 

vi /opt/svn/svndata/repos1/conf/svnserve.conf 

内容修改为: 

[general] 

anon-access = none 

auth-access = write 

password-db = /opt/svn/svndata/pwd.conf 

authz-db = /opt/svn/svndata/authz.conf 

realm = repos1 

版本库2: 

vi /opt/svn/svndata/repos2/conf/svnserve.conf 

内容修改为: 

[general] 

anon-access = none 

auth-access = write 

password-db = /opt/svn/svndata/pwd.conf 

authz-db = /opt/svn/svndata/authz.conf 

realm = repos2 

即除realm = repos2外，其他与版本库1配置文件完全相同。如果有更多的版本库，依此类推。 

 

配置允许访问的用户: 

vi /opt/svn/conf/pwd.conf 

为了简化配置，2个版本库共用1个用户配置文件。如有必要，也可以分开。 

注意：对用户配置文件的修改立即生效，不必重启svn。 

文件格式如下： 

[users] 

<用户1> = <密码1> 

<用户2> = <密码2> 

其中，[users]是必须的。下面列出要访问svn的用户，每个用户一行。

 

示例： 

[users] 

alan = password 

king = hello 

 

配置用户访问权限: 

vi /opt/svn/conf/authz.conf 

为了简化配置，3个版本库共用1个权限配置文件/opt/svn/conf/pwd.conf。如有必要，也可以分开。文件中定义用户组和版本库目录权限。 

 

注意： 

* 权限配置文件中出现的用户名必须已在用户配置文件中定义。 

* 对权限配置文件的修改立即生效，不必重启svn。 

 

用户组格式： 

[groups] 

<用户组名> = <用户1>,<用户2> 

其中，1个用户组可以包含1个或多个用户，用户间以逗号分隔。 

 

版本库目录格式： 

[<版本库>:/项目/目录] 

@<用户组名> = <权限> 

<用户名> = <权限> 

 

其中，方框号内部分可以有多种写法: 

/,表示根目录及以下。根目录是svnserve启动时指定的，我们指定为/opt/svndata。这样，/就是表示对全部版本库设置权限。 

repos1:/,表示对版本库1设置权限 

repos2:/occi, ,表示对版本库2中的occi项目设置权限 

repos2:/occi/aaa, ,表示对版本库2中的occi项目的aaa目录设置权限 

权限主体可以是用户组、用户或*，用户组在前面加@，*表示全部用户。权限可以是w、r、wr和空，空表示没有任何权限。

示例： 

      [groups] 

admin = alan 

[/] 

@admin = rw 

[repos1:/occi/aaa] 

king = rw 

[repos2:/pass] 

king = 

删除无用文件: 

rm /opt/svn/svndata/repos1/conf/authz 

rm /opt/svn/svndata/repos1/conf/passwd 

rm /opt/svn/svndata/repos2/conf/authz 

rm /opt/svn/svndata/repos2/conf/passwd 

 

5.启动svn： 

svnserve -d --listen-port 9999 -r /opt/svn/svndata 

其中： 

-d表示以daemon方式（后台运行）运行 

--listen-port 9999表示使用9999端口，可以换成你需要的端口。但注意，使用1024以下的端口需要root权限 

-r /opt/svn/svndata指定根目录是/opt/svn/svndata 

检查: 

ps –ef|grep svnserve 

如果显示如下，即为启动成功： 

root  6941　　 1　0 15:07 ?　　00:00:00 svnserve -d --listen-port 9999 -r /opt/svn/svndata
