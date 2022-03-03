# 1 keytool命令说明

## 1.1 keytool是密钥和证书管理工具

命令:

 -certreq            生成证书：请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令

使用 "keytool -command_name -help" 获取 command_name 的用法

## 1.2 -genkeypair 生成秘钥对命令

keytool -genkeypair [OPTION]...

生成密钥对

选项:

 -alias <alias>                  要处理的条目的别名
 -keyalg <keyalg>                密钥算法名称
 -keysize <keysize>              密钥位大小
 -sigalg <sigalg>                签名算法名称
 -destalias <destalias>          目标别名
 -dname <dname>                  唯一判别名，例如：  "CN=名字与姓氏,OU=组织单位名称,O=组织名称,L=城市或区域名称,ST=州或省份名称,C=单位的两字母国家代码"
 -startdate <startdate>          证书有效期开始日期/时间
 -ext <value>                    X.509 扩展
 -validity <valDays>             有效天数
 -keypass <arg>                  密钥口令
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

# 2 生成秘钥库文件

例1：生成秘钥库casserver.jks

~~~powershell
PS E:\etc\cas> keytool -genkey -validity 36500 -keysize 1024 -alias cas -keyalg RSA -keystore .\casserver.jks -dname "CN=cas.example.org,OU=xzops,O=xzops,L=GZ,ST=GD,C=CN" -storepass root.501 -keypass root.501 -v
正在为以下对象生成 1,024 位RSA密钥对和自签名证书 (SHA256withRSA) (有效期为 36,500 天):
         CN=cas.example.org, OU=xzops, O=xzops, L=GZ, ST=GD, C=CN
[正在存储.\casserver.jks]

Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore .\casserver.jks -destkeystore .\casserver.jks -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。
~~~

执行上方命令时打印出一个告警信息，建议使用行业标准格式PKCS12，下面我们可直接指定秘钥库格式-storetype pkcs12（注意：再次执行前需要删除上方生成的casserver.jks文件，否则会报错）

~~~powershell
PS E:\etc\cas> keytool -genkey -validity 36500 -keysize 1024 -alias cas -keyalg RSA -storetype pkcs12 -keystore .\casserver.jks -dname "CN=cas.example.org,OU=xzops,O=xzops,L=GZ,ST=GD,C=CN" -storepass root.501 -keypass root.501 -v
正在为以下对象生成 1,024 位RSA密钥对和自签名证书 (SHA256withRSA) (有效期为 36,500 天):
         CN=cas.example.org, OU=xzops, O=xzops, L=GZ, ST=GD, C=CN
[正在存储.\casserver.jks]
~~~

例2：生成信任IP而非域名方式的秘钥库

~~~shell
[root@localhost ~]# keytool -genkey -validity 36500 -keysize 1024 -alias server31 -keyalg RSA -storetype pkcs12 -keystore server31.jks -dname "CN=192.168.1.31,OU=xzops,O=xzops,L=GZ,ST=GD,C=CN" -ext san=ip:192.168.1.31 -storepass root.501 -keypass root.501 -v
Generating 1,024 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 36,500 days
        for: CN=192.168.1.31, OU=xzops, O=xzops, L=GZ, ST=GD, C=CN
[Storing server31.jks]
~~~

# 3 导出证书文件

例：根据上方生成的秘钥库文件casserver.jks导出证书文件casserver.cer

~~~powershell
PS E:\etc\cas> keytool -exportcert -keystore .\casserver.jks -alias cas -storepass root.501 -file .\casserver.cer
存储在文件 <.\casserver.cer> 中的证书
~~~

# 4 生成证书信任库

例：根据上方导出的证书文件casserver.cer生成证书信任库casserver.jts

~~~powershell
PS E:\etc\cas> keytool -importcert -file .\casserver.cer -alias cas -keypass root.501 -keystore .\casserver.jts -storetype pkcs12 -storepass root.501
所有者: CN=cas.example.org, OU=xzops, O=xzops, L=GZ, ST=GD, C=CN
发布者: CN=cas.example.org, OU=xzops, O=xzops, L=GZ, ST=GD, C=CN
序列号: 58157f6e
有效期为 Mon Mar 08 15:47:59 CST 2021 至 Wed Feb 12 15:47:59 CST 2121
证书指纹:
         MD5:  8B:DB:06:0E:FF:82:AB:C5:17:12:0A:9A:28:29:E9:4F
         SHA1: 41:C5:48:00:78:F1:98:1A:9E:E9:5A:66:F6:DC:7B:98:6C:A4:07:1B
         SHA256: 8C:3C:FD:7B:29:25:8F:89:21:01:37:AB:37:2E:9D:22:F2:CE:F1:8A:C9:5D:40:DC:0F:AD:4A:7F:54:16:7D:51
签名算法名称: SHA256withRSA
主体公共密钥算法: 1024 位 RSA 密钥
版本: 3

扩展:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 5D AF 28 88 3F 0E FE C6   E4 8D C7 D3 64 F4 32 E5  ].(.?.......d.2.
0010: 4C 20 AC 09                                        L ..
]
]

是否信任此证书? [否]:  y
证书已添加到密钥库中
~~~

# 5 导入证书到JDK证书库

1)、Java证书库位置：$JAVA_HOME/jre/lib/security/cacerts

2)、查看证书库的证书
keytool -list -keystore $JAVA_HOME/jre/lib/security/cacerts

输入密码：changeit

3)、导入证书
keytool -import -alias cas -keystore $JAVA_HOME/jre/lib/security/cacerts -file cas.cer

4)、删除证书
keytool -delete -alias cas -keystore $JAVA_HOME/jre/lib/security/cacerts

