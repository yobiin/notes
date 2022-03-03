#### 1 shutdown管理端口

原`server.xml`文件的`shutdown`端口及指令配置如下：

~~~xml
<Server port="8005" shutdown="SHUTDOWN">
~~~

将其修改为

~~~xml
<Server port="21005" shutdown="f6c395f7d95e49cf8b8b554b9a91588f">
~~~

#### 2 AJP端口

原`server.xml`文件的AJP端口配置如下：

~~~xml
<Connector protocol="AJP/1.3"
               address="::1"
               port="8009"
               redirectPort="8443" />
~~~

将其注释掉

~~~
<!--
    <Connector protocol="AJP/1.3"
               address="::1"
               port="8009"
               redirectPort="8443" />
    -->
~~~

#### 3 关闭自动解压自动部署功能

原`server.xml`文件的自动解压及自动部署配置如下：

~~~xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->
        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
~~~

修改`unpackWARs`等于`false`，`autoDeploy`等于`false`然后保存。

#### 4 删除tomcat/webapps目录下除自己应用外的所有目录

如下所示，其中`test`是我要部署的项目：

~~~shell
$TOMCAT_HOME/webapps
[root@192 webapps]# ll
total 16
drwxr-x---. 15 root root 4096 Nov 25 17:33 docs
drwxr-x---.  7 root root   93 Nov 25 17:33 examples
drwxr-x---.  6 root root   74 Nov 25 17:33 host-manager
drwxr-x---.  6 root root 4096 Nov 25 17:33 manager
drwxr-x---.  3 root root 4096 Nov 25 17:33 ROOT
drwxr-x---.  3 root root 4096 Nov 25 16:45 test
~~~

删除`docs`、`examples`、`host-manager`、`manager`、`ROOT`目录，然后修改`server.xml`文件，在`Host`标签内添加`Context`标签，如下所示：

~~~xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
    <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
    <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->
    <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
           prefix="localhost_access_log" suffix=".txt"
           pattern="%h %l %u %t &quot;%r&quot; %s %b" />
    <Context path="" docBase="test" debug="0" reloadable="false" ></Context>
</Host>
~~~

#### 5 删除tomcat版本信息

进入`$TOMCAT_HOME/lib`目录，找到`catalina.jar`包：

~~~shell
# cd $TOMCAT_HOME/lib
// 执行如下命令解压catalina.jar包
# $JAVA_HOME/bin/jar xf catalina.jar
~~~

解压后得到`META-INF`、`org`两个目录和一个文件`module-info.class`，找到`org/apache/catalina/util/ServerInfo.properties`文件，打开如下：

~~~shell
server.info=Apache Tomcat/9.0.55
server.number=9.0.55.0
server.built=Nov 10 2021 08:26:45 UTC
~~~

删除版本信息，如下所示

~~~shell
server.info=
server.number=
server.built=
~~~

保存更改后的`ServerInfo.properties`文件，然后重新将`META-INF`、`org`、`module-info.class`打包成`catalina.jar`，具体操作如下：

~~~shell
$JAVA_HOME/bin/jar cf catalina.jar META-INF module-info.class org
~~~

#### 6 非root用户启动tomcat

创建tomcat运行用户及组

~~~shell

~~~



