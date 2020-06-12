# OpenLDAP双主同步

#### 启用 syncprov 模块
```shell
#syncprov 是 OpenLDAP 用来进行数据同步的模块，这里我们首先将其启用。
#在两台服务器上分别创建一个名为 syncprov_mod.ldif 文件。
[root@ldap-server1 ~]# cat >syncprov_mod.ldif<<EOF
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la
EOF

#然后在两台服务器上分别执行该文件。
[root@ldap-server1 ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f syncprov_mod.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=module,cn=config"
```

#### 启用OpenLDAP的双主同步
```shell
[root@ldap-server1 ~]# cat >configrep.ldif<<EOF
### Update Server ID with LDAP URL ###

dn: cn=config
changetype: modify
replace: olcServerID
olcServerID: 1 ldap://openldap-master-1.colinlee.fish
olcServerID: 2 ldap://openldap-master-2.colinlee.fish

### Enable replication ###

dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov

### Adding details for replication ###

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl:
  rid=001
  provider=ldap://openldap-master-1.colinlee.fish
  binddn="cn=admin,dc=colinlee,dc=fish"
  bindmethod=simple
  credentials=MyAdMiNP@ssW0rd
  searchbase="dc=colinlee,dc=fish"
  type=refreshAndPersist
  retry="5 5 300 5"
  timeout=1
olcSyncRepl:
  rid=002
  provider=ldap://openldap-master-2.colinlee.fish
  binddn="cn=admin,dc=colinlee,dc=fish"
  bindmethod=simple
  credentials=MyAdMiNP@ssW0rd
  searchbase="dc=colinlee,dc=fish"
  type=refreshAndPersist
  retry="5 5 300 5"
  timeout=1
-
add: olcMirrorMode
olcMirrorMode: TRUE

#然后在两台服务器上分别执行这个 ldap_sync.ldif 文件即可。
[root@ldap-server1 ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f configrep.ldif
```
