# 使用SSL配置OpenLDAP

1) 使用SSL 配置OpenLDAP以进行安全通信。在此设置中，LDAP客户端通信通过安全端口636而不是非安全端口389进行。    
2) OpenLDAP安装部分见 https://github.com/guoqunchao/LDAP       

#### 1) 创建LDAP自签名证书
```shell
'''生成ca证书'''
[root@base-ldap-master ~]# cd /etc/openldap/certs/
[root@base-ldap-master certs]# openssl genrsa -out rootCA.key 2048
[root@base-ldap-master certs]# openssl req -x509 -new -nodes -subj "/C=CN/ST=ShangHai/L=ShangHai/O=ldap/OU=lework/CN=ldap-ca"  -key rootCA.key -sha256 -days 1024 -out rootCA.pem

'''生成ldap证书请求'''
'''签发ldap证书'''
'''创建certs.ldif文件以配置LDAP使用自签名证书进行安全通信'''
'''测试配置'''
```

#### 2) 配置OpenLDAP开启SSL
