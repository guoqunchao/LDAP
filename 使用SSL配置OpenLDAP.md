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
[root@base-ldap-master certs]# openssl genrsa -out ldap.key 2048
[root@base-ldap-master certs]# openssl req -new -subj "/C=CN/ST=BeiJing/L=BeiJing/O=ldap/OU=zichan360/CN=ldap-master.zichan360.com" -key ldap.key -out ldap.csr

'''签发ldap证书'''
[root@base-ldap-master certs]# openssl x509 -req -in ldap.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out ldap.crt -days 3650 -sha256
Signature ok
subject=/C=CN/ST=BeiJing/L=BeiJing/O=ldap/OU=zichan360/CN=ldap-master.zichan360.com
Getting CA Private Key
[root@base-ldap-master certs]# chown -R ldap:ldap /etc/openldap/certs/

'''创建certs.ldif文件以配置LDAP使用自签名证书进行安全通信'''
[root@base-ldap-master ~]# cat >certs.ldif<<EOF
dn: cn=config
changetype: modify
replace: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/rootCA.pem

dn: cn=config
changetype: modify
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/ldap.crt

dn: cn=config
changetype: modify
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/ldap.key
EOF
[root@base-ldap-master ~]# ldapmodify -Y EXTERNAL  -H ldapi:/// -f certs.ldif


'''测试配置'''
[root@base-ldap-master ~]# slaptest -u
config file testing succeeded
```

#### 2) 配置OpenLDAP开启SSL
```shell
[root@base-ldap-master ~]# vim /etc/sysconfig/slapd
SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"
[root@base-ldap-master ~]# systemctl restart slapd
```
