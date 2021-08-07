Ref: https://www.itzgeek.com/how-tos/linux/centos-how-tos/step-step-openldap-server-configuration-centos-7-rhel-7.html
     https://www.itzgeek.com/how-tos/linux/centos-how-tos/migrate-local-users-ldap-accounts.html

### Install the following LDAP RPM packages on LDAP server 
```
# yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel
```

### Start the LDAP service and enable it for the auto start of service on system boot.
```
# systemctl start slapd
# systemctl enable slapd
```

### Verify the LDAP
```
# netstat -antup | grep -i 389
```

### Setup LDAP admin password, replace ldppassword with your password
```
# slappasswd -h {SSHA} -s ldppassword
```

### Create a .ldif file
```
# cd /etc/openldap/slapd.d/
# vi db.ldif

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=galaxy,dc=local

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=galaxy,dc=local

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}d/thexcQUuSfe3rx3gRaEhHpNJ52N8D3  // Replace the encrypted password ({SSHA}d/thexcQUuSfe3rx3gRaEhHpNJ52N8D3) with the password you generated in the previous step.

# ldapmodify -Y EXTERNAL  -H ldapi:/// -f db.ldif
```

### Create monitor.ldif file
```
# vi monitor.ldif

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, cn=auth" read by dn.base="cn=ldapadm,dc=galaxy,dc=local" read by * none

# ldapmodify -Y EXTERNAL  -H ldapi:/// -f monitor.ldif
```

### Set up LDAP database
```
# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
# chown ldap:ldap /var/lib/ldap/*
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
# vi base.ldif

dn: dc=galaxy,dc=local
dc: galaxy
objectClass: top
objectClass: domain

dn: cn=ldapadm ,dc=galaxy,dc=local
objectClass: organizationalRole
cn: ldapadm
description: LDAP Manager

dn: ou=People,dc=galaxy,dc=local
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=galaxy,dc=local
objectClass: organizationalUnit
ou: Group

# ldapadd -x -W -D "cn=ldapadm,dc=itzgeek,dc=local" -f base.ldif
```

### Migrate local users to LDAP accounts
```
# yum -y install migrationtools
# grep "ldpuser" /etc/passwd >> /root/users
# grep "ldpuser" /etc/group >> /root/groups
# vi /usr/share/migrationtools/migrate_common.ph

$DEFAULT_MAIL_DOMAIN = "galaxy.local";
$DEFAULT_BASE = "dc=galaxy,dc=local";
$EXTENDED_SCHEMA = 1;

# /usr/share/migrationtools/migrate_passwd.pl /root/users /root/users.ldif
# /usr/share/migrationtools/migrate_group.pl /root/groups /root/groups.ldif
# ldapadd -x -W -D "cn=ldapadm,dc=galaxy,dc=local" -f /root/users.ldif
# ldapadd -x -W -D "cn=ldapadm,dc=galaxy,dc=local" -f /root/groups.ldif
# ldapsearch -x cn=ldpuser1 -b dc=galaxy,dc=local
# ldapdelete -v -D "cn=ldapadm,dc=galaxy,dc=local" -W "uid=xxxxxxxx,ou=People,dc=galaxy,dc=local"
# ldapdelete -v -D "cn=ldapadm,dc=galaxy,dc=local" -W "cn=xxxxxxxx,ou=Group,dc=galaxy,dc=local"
```
