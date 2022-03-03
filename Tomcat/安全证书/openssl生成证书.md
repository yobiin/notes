## 1 OpenSSL不同后缀文件说明

| 文件后缀名 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| *.key      | 私有的密钥                                                   |
| *.crt      | 证书文件，certificate的缩写                                  |
| *.csr      | 证书签名请求（证书请求文件），含有公钥信息，certificate signing request的缩写 |
| *.crl      | 证书吊销列表，Certificate Revocation List的缩写              |
| *.pem      | 用于导出，导入证书时候的证书的格式，有证书开头，结尾的格式   |
| *.crt.pem  | 可导出证书                                                   |

## 2 OpenSSL创建CA证书

1 生成CA秘钥

~~~shell
# 执行已下指令并输入密码
openssl genrsa -des3 -out ca.key 2048
~~~

2 生成CA根证书

~~~shell
# 执行以下指令并输入密码
openssl req -sha256 -new -x509 -days 3650 -key ca.key -out ca.crt \
    -subj "/C=CN/ST=GD/L=GZ/O=xzops/OU=xzops/CN=xzops.com"
~~~

3 生成服务器秘钥

~~~shell
# 执行已下指令并输入密码
openssl genrsa -des3 -out server41.key 2048
~~~

4 生成服务器请求文件（公钥）

~~~shell
# 执行已下指令并输入密码
openssl req -new \
    -sha256 \
    -key server41.key \
    -subj "/C=CN/ST=GD/L=GZ/O=xzops/OU=xzops/CN=xzops.com" \
    -reqexts SAN \
    -config <(cat /etc/pki/tls/openssl.cnf \
        <(printf "[SAN]\nsubjectAltName=DNS:*.xzops.com,IP:192.168.1.41")) \
    -out server41.csr
~~~

5 CA签署服务器证书

~~~shell
# 执行如下指令并输入密码
openssl ca -in server41.csr \
    -md sha256 \
    -keyfile ca.key \
    -cert ca.crt \
    -extensions SAN \
    -config <(cat /etc/pki/tls/openssl.cnf \
        <(printf "[SAN]\nsubjectAltName=DNS:*.xzops.com,IP:192.168.1.41")) \
    -out server41.crt
~~~

可能出现的错误：

①  缺少/etc/pki/CA/index.txt文件

~~~shell
/etc/pki/CA/index.txt: No such file or directory
unable to open '/etc/pki/CA/index.txt'
139752875595664:error:02001002:system library:fopen:No such file or directory:bss_file.c:402:fopen('/etc/pki/CA/index.txt','r')
139752875595664:error:20074002:BIO routines:FILE_CTRL:system lib:bss_file.c:404:
~~~

新建一个index.txt文件即可

~~~shell
touch /etc/pki/CA/index.txt
~~~

②  缺少/etc/pki/CA/serial文件

~~~shell
error while loading serial number
140186320246672:error:02001002:system library:fopen:No such file or directory:bss_file.c:402:fopen('/etc/pki/CA/serial','r')
140186320246672:error:20074002:BIO routines:FILE_CTRL:system lib:bss_file.c:404:
~~~

新建一个serial文件，同时输入内容00

~~~shell
touch /etc/pki/CA/serial
echo 00 > /etc/pki/CA/serial
~~~

## 3 证书转换

1 ctr 转 cer

~~~shell
openssl x509 -in server41.crt -out server41.cer -outform der
~~~

