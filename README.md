# LDAP

```shell
#安装yum-utils工具
yum install -y yum-utils

#添加docker-ce源：
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#安装docker-ce
yum install docker-ce -y

#配置服务器的时间同步
yum install ntp -y

#启动自动同步时间:
timedatectl set-ntp yes  #此处可用yes,no,1或0

#配置时区:
timedatectl set-timezone Asia/Shanghai

#配置一下镜像源加快拉取镜像
cat /etc/docker/daemon.json
        {
         "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
        }

#如果需要添加一个私有的仓库reg.docker.tb以同样的方法：
cat /etc/docker/daemon.json
    {
     "insecure-registries":["reg.docker.tb"],
     "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
    }
systemctl enable docker && systemctl start docker
docker pull osixia/openldap
docker run -p 389:389 --name openldap --restart=always --env LDAP_ORGANISATION="zichan360-external" --env LDAP_DOMAIN="zichan360-external.com" --env LDAP_ADMIN_PASSWORD="zichan360" --detach osixia/openldap

#389端口：默认ldap服务是使用389端口
#LDAP_ORGANISATION 表示ldap的机构组织
#LDAP_DOMAIN 配置LDAP域
#LDAP_ADMIN_PASSWORD 配置LDAP管理员(admin)的密码
#默认用登陆用户名admin

Base: dc=hlzxcq,dc=com
Username:cn=admin,dc=hlzxcq,dc=com
```


```shell
#用户文件
root@6ea76c7fcf24:/# cat User.ldif 
#replace to your own domain name for "dc=***,dc=***" section
#1
dn: uid=liyanling,ou=ChongQing,dc=hlzxcq,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: 燕玲
sn: 李
userPassword:123456
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/liyanling
#mail: mobai@zichan360.com
telephoneNumber: 11111111111
title: 资产处置经理

#2
dn: uid=tanlu,ou=ChongQing,dc=hlzxcq,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: 璐
sn: 谭
userPassword:123456
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/tanlu
#mail: mobai@zichan360.com
telephoneNumber: 11111111111
title: 资产处置主管


#3
dn: uid=liyan,ou=ChongQing,dc=hlzxcq,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: 艳
sn: 李
userPassword:123456
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/liyan
#mail: mobai@zichan360.com
telephoneNumber: 11111111111
title: 资产处置组长

```
```shell
#用户组文件
root@6ea76c7fcf24:/# cat Group.ldif 
dn: cn=资产处置部,ou=group,dc=hlzxcq,dc=com
objectClass: posixGroup
objectClass: top
cn: 资产处置部
memberUid: chenyang
memberUid: fanglin
gidNumber: 500


dn: cn=资产处置部2,ou=group,dc=hlzxcq,dc=com
objectClass: posixGroup
objectClass: top
cn: 资产处置部2
memberUid: tanlu
memberUid: sunsi
gidNumber: 500
```

```shell
#查询所有uid
ldapsearch -x -b "ou=ChongQing,dc=hlzxcq,dc=com" -D "cn=admin,dc=hlzxcq,dc=com" -w zichan360

#添加用户
ldapadd -x -D "cn=admin,dc=hlzxcq,dc=com" -w zichan360 -f User.ldif

删除单用户
ldapdelete -x -D "cn=admin,dc=hlzxcq,dc=com" -w zichan360 uid=sunsi,ou=ChongQing,dc=hlzxcq,dc=com

删除所有用户
ldapsearch -x -b "ou=ChongQing,dc=hlzxcq,dc=com" -D "cn=admin,dc=hlzxcq,dc=com" -w zichan360 |egrep "dn: uid"|awk '{print "ldapdelete -x -D \"cn=admin,dc=hlzxcq,dc=com\" -w zichan360 "$2}'|bash
```

