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
root@b09330660342:/# cat ldapuser.ldif 
# replace to your own domain name for "dc=***,dc=***" section
dn: uid=inkwhite,ou=staff,dc=zichan360-external,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: 墨白
sn: 墨
userPassword:123456
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/inkwhite
mail: mobai@zichan360.com
telephoneNumber: 12345678911
title: 首席技术官


cn: 运维组
description: 运维组
dn: cn=运维组,ou=Group,dc=zichan360-external,dc=com
objectClass: posixGroup
gidNumber: 1000


ldapadd -x -D cn=admin,dc=zichan360-external,dc=com -W  -f ldapuser.ldif
```
