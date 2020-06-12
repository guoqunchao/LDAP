# OpenLDAP主从搭建与配置

#### 1)安装openldap软件
```shell
[root@base-ldap-master ~]# yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel migrationtools
[root@base-ldap-master ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@base-ldap-master ~]# chown ldap:ldap -R /var/lib/ldap
[root@base-ldap-master ~]# chmod 700 -R /var/lib/ldap
[root@base-ldap-master ~]# systemctl start slapd && systemctl status slapd
[root@base-ldap-master ~]# systemctl enable slapd
```

#### 2) 配置openldap管理员密码
```shell
[root@base-ldap-master ~]# HRSpHDcUi0NL4ZSo
[root@Devops-gate ~]# slappasswd 
New password: HRSpHDcUi0NL4ZSo
Re-enter new password: HRSpHDcUi0NL4ZSo
{SSHA}8PUBzRFMY4BvoL1j1A5xps0dMVnVBxYi
[root@base-ldap-master ~]# cat >/root/chrootpw.ldif<<EOF
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}8PUBzRFMY4BvoL1j1A5xps0dMVnVBxYi
EOF
[root@base-ldap-master ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /root/chrootpw.ldif
```

#### 3) 导入相关openldap属性
```shell
[root@base-ldap-master ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
[root@base-ldap-master ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
[root@base-ldap-master ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

#### 4) 修改openldap的基本配置
```shell
[root@base-ldap-master ~]# vim /root/chdomain.ldif 
#replace to your own domain name for “dc=***,dc=***” section
#specify the password generated above for “olcRootPW” section
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=root,dc=zichan360,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=zichan360,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=root,dc=zichan360,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}8PUBzRFMY4BvoL1j1A5xps0dMVnVBxYi

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by dn="cn=root,dc=zichan360,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=root,dc=zichan360,dc=com" write by * read
[root@base-ldap-master ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/chdomain.ldif
```

#### 5) 导入基础数据库
```shell
[root@base-ldap-master ~]# vim basedomain.ldif 
#replace to your own domain name for “dc=***,dc=***” section
dn: dc=zichan360,dc=com
objectClass: top
objectClass: dcObject
objectclass: organization
o: Server cn
dc: zichan360

dn: cn=root,dc=zichan360,dc=com
objectClass: organizationalRole
cn: root
description: Directory root

dn: ou=People,dc=zichan360,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=zichan360,dc=com
objectClass: organizationalUnit
ou: Group


[root@base-ldap-master ~]# ldapadd -x -D cn=root,dc=zichan360,dc=com -w "HRSpHDcUi0NL4ZSo" -f /root/basedomain.ldif
adding new entry "dc=zichan360,dc=com"
adding new entry "cn=root,dc=zichan360,dc=com"
adding new entry "ou=People,dc=zichan360,dc=com"
adding new entry "ou=Group,dc=zichan360,dc=com"
```

#### 6) 导入用户
```shell
[root@base-ldap-master ~]# cat users.ldif 
dn: uid=ldapuser1,ou=People,dc=zichan360,dc=com
uid: ldapuser1
cn: 测试用户1
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword: {crypt}$6$pmVuchTg$kLzWnW0J1CS3LTWrzMu4PVnjROjXaoVUlr8Em3HzIH6wAK74Gzor7yiuRbrOoYCRGHmSNhAGBxMTNEcTkfpUt1
shadowLastChange: 17642
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/ldapuser1

dn: uid=ldapuser2,ou=People,dc=zichan360,dc=com
uid: ldapuser2
cn: 测试用户2
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword: {crypt}$6$NC7BvWQW$b.ceEn5zl7tOf0upfR3E5057um5ovIDo4Xf5sCOZVhwrr01nOfPmqXB0pNBtQCjzahP1lW3DLW5WKBp.qddeT0
shadowLastChange: 17642
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1001
gidNumber: 1001
homeDirectory: /home/ldapuser2


'''
上述命令中，有关ldap用户密码的部分，我们可以是明文的形式存在。也可以是通过slappasswd命令生成的加密后的密码。
生成SSHA方案密码。
[root@linuxlz.com~]# slappasswd

生成随机密码
[root@linuxlz.com~]# slappasswd -g

生成哈希密码
[root@linuxlz.com~]# slappasswd -s redhat

生成CRYPT方案密码
[root@linuxlz.com~]# slappasswd -c crypt-salt-format

生成MD5方案密码
[root@linuxlz.com~]# slappasswd -h {MD5}
'''

[root@base-ldap-master ~]# ldapadd -x -w "HRSpHDcUi0NL4ZSo" -D "cn=root,dc=zichan360,dc=com" -f /root/users.ldif 
adding new entry "uid=ldapuser1,ou=People,dc=zichan360,dc=com"
adding new entry "uid=ldapuser2,ou=People,dc=zichan360,dc=com"
```

#### 7) 导入用户组
```shell
[root@base-ldap-master ~]# cat /root/groups.ldif 
dn: cn=ldapgroup1,ou=Group,dc=zichan360,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapgroup1
userPassword: {crypt}x
gidNumber: 1000
dn: cn=ldapgroup2,ou=Group,dc=zichan360,dc=com
objectClass: posixGroup
objectClass: top
cn: ldapgroup2
userPassword: {crypt}x
gidNumber: 1001

[root@base-ldap-master ~]# ldapadd -x -w "HRSpHDcUi0NL4ZSo" -D "cn=root,dc=zichan360,dc=com" -f /root/groups.ldif 
adding new entry "cn=ldapgroup1,ou=Group,dc=zichan360,dc=com"
adding new entry "cn=ldapgroup2,ou=Group,dc=zichan360,dc=com"
```

#### 8) 把用户加入到用户组
```shell
[root@base-ldap-master ~]# vim /root/add_user_to_groups.ldif 
dn: cn=ldapgroup1,ou=Group,dc=zichan360,dc=com
changetype: modify
add: memberuid
memberuid: ldapuser1

dn: cn=ldapgroup2,ou=Group,dc=zichan360,dc=com
changetype: modify
add: memberuid
memberuid: ldapuser2

[root@base-ldap-master ~]# ldapadd -x -w "HRSpHDcUi0NL4ZSo" -D "cn=root,dc=zichan360,dc=com" -f /root/add_user_to_groups.ldif
modifying entry "cn=ldapgroup1,ou=Group,dc=zichan360,dc=com"
modifying entry "cn=ldapgroup2,ou=Group,dc=zichan360,dc=com"
```

#### 9) 开启openldap日志功能
```shell
[root@base-ldap-master ~]# vim /root/loglevel.ldif 
dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: stats
[root@base-ldap-master ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/loglevel.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"
[root@base-ldap-master ~]# systemctl restart slapd
[root@base-ldap-master ~]# cat >> /etc/rsyslog.conf << EOF
local4.* /var/log/slapd.log
EOF
[root@base-ldap-master ~]# systemctl restart rsyslog
[root@base-ldap-master ~]# tail -f /var/log/slapd.log
Jun 11 21:04:58 iZ8vb292x9xrlgf6slqdm2Z slapd[12873]: daemon: shutdown requested and initiated.
Jun 11 21:04:58 iZ8vb292x9xrlgf6slqdm2Z slapd[12873]: slapd shutdown: waiting for 0 operations/tasks to finish
Jun 11 21:04:58 iZ8vb292x9xrlgf6slqdm2Z slapd[12873]: slapd stopped.
Jun 11 21:04:58 iZ8vb292x9xrlgf6slqdm2Z slapd[12958]: @(#) $OpenLDAP: slapd 2.4.44 (Jan 29 2019 17:42:45) $#012#011mockbuild@x86-01.bsys.centos.org:/builddir/build/BUILD/openldap-2.4.44/openldap-2.4.44/servers/slapd
Jun 11 21:04:58 iZ8vb292x9xrlgf6slqdm2Z slapd[12960]: slapd starting
```

#### 10) slave上安装openldap
```shell
'''
1.而要在slave机器上配置openldap的主从，我们也要安装openldap并进行相关配置。
2.不过对于在slave机器安装的openlap，不需要像master机器上那样进行全部的配置，我们只需要操作第一到第四章节即可。
3.其中后续的章节，比如：导入基础数据库、导入用户、导入用户组和用户加入到用户组都不需要。
'''
```

#### 11) master的上主从配置
```shell
'''在master机器，我们需要进行导入相关的属性'''
[root@base-ldap-master ~]# cat >/root/syncprov_mod.ldif<<EOF
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la
EOF

[root@base-ldap-master ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /root/syncprov_mod.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=module,cn=config"

[root@base-ldap-master ~]# cat >/root/syncprov.ldif<<EOF
dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpCheckpoint: 1 1
olcSpSessionLog: 1024
EOF

[root@base-ldap-master ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /root/syncprov.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "olcOverlay=syncprov,olcDatabase={2}hdb,cn=config"

#以上就是master机器上的配置
```

#### 12) slave的上主从配置
```shell
[root@base-ldap-slave ~]# cat >/root/rp.ldif<<EOF 
dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: rid=001
    provider=ldap://192.168.2.154:389/
    bindmethod=simple
    binddn="cn=root,dc=zichan360,dc=com"
    credentials=HRSpHDcUi0NL4ZSo
    searchbase="dc=zichan360,dc=com"
    scope=sub
    schemachecking=on
    type=refreshAndPersist
    retry="30 5 300 3"
    interval=00:00:05:00
EOF
[root@base-ldap-slave ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/rp.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
ldapmodify: invalid format (line 5) entry: "olcDatabase={2}hdb,cn=config"
#其中provider表示master的地址，其他的都是些基础信息。不过这里面需要注意的是认证用户一定要使用超级管理员，如果使用普通用户连接master的话，slave将不会同步用户的密码字段信息。
#除此之外，为了优化openldap的查询速度，我们添加了相关字段属性的索引。
```

#### 13) 验证主从正确性
'''
slave机器上配置完毕后，无需重启master机器和slave机器的slapd服务。
在slave机器上查看openldap日志，如下：
'''



