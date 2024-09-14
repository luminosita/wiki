# Configure Open LDAP

# Access the LDAP container's shell  

```bash
$ kubectl exec --stdin --tty <pod name> -n openldap -- /bin/bash
```

After accessing the LDAP container’s shell, we can configure organizational units (OUs), groups, and user accounts using LDIF files. LDIF (LDAP Data Interchange Format) provides a structured way to define directory entries. Execute the following commands within the LDAP container to create OUs, groups, and user accounts from LDIF files:

# Create OU (Organization Units).   

```bash
$ ldapadd -x -w password -D "cn=admin,dc=example,dc=in" << EOF  
# LDIF file to add organizational unit "ou=devops" under "dc=example,dc=in"  
dn: ou=devops,dc=example,dc=in  
objectClass: organizationalUnit  
ou: devops  

# LDIF file to add organizational unit "ou=appdev" under "dc=example,dc=in"  
dn: ou=appdev,dc=example,dc=in  
objectClass: organizationalUnit  
ou: appdev  
EOF  
```

# Create User accounts.  
```bash
$ ldapadd -x -w password -D "cn=admin,dc=example,dc=in" << EOF  
# LDIF file to Create user "amrutha" in "ou=appdev" under "dc=example,dc=in"  
dn: cn=amrutha,ou=devops,dc=example,dc=in  
objectClass: person  
cn: amrutha  
sn: Amrutha  
uid: amrutha  
userPassword: Amrutha@123  
  
# LDIF file to Create user "charlie" in "ou=appdev" under "dc=example,dc=in"  
dn: cn=charlie,ou=appdev,dc=example,dc=in  
objectClass: inetOrgPerson  
cn: charlie  
sn: Charlie  
uid: charlie  
userPassword: Charlie@123  
  
# LDIF file to Create user "amit" in "ou=appdev" under "dc=example,dc=in"  
dn: cn=amit,ou=appdev,dc=example,dc=in  
objectClass: inetOrgPerson  
cn: amit  
sn: Amit  
uid: amit  
userPassword: Amit@123  
EOF  
```
  
# Create Groups  

```bash
$ ldapadd -x -w password -D "cn=admin,dc=example,dc=in" << EOF  
# Group: appdev-team  
dn: cn=appdev-team,dc=example,dc=in  
objectClass: top  
objectClass: groupOfNames  
cn: appdev-team  
description: App Development Team  
member: cn=amrutha,ou=devops,dc=example,dc=in  
member: cn=charlie,ou=appdev,dc=example,dc=in  
  
# Group: devops-team  
dn: cn=devops-team,dc=example,dc=in  
objectClass: top  
objectClass: groupOfNames  
cn: devops-team  
description: DevOps Team  
member: cn=amrutha,ou=devops,dc=example,dc=in  
member: cn=amit,ou=appdev,dc=example,dc=in  
EOF  
```

# Modify and apply MemberOf attribute to Users in Groups.  

```bash
$ ldapadd -x -w password -D "cn=admin,dc=example,dc=in" << EOF  
dn: cn=amrutha,ou=devops,dc=example,dc=in  
changetype: modify  
add: memberOf  
memberOf: cn=devops-team,dc=example,dc=in  
  
dn: cn=amit,ou=appdev,dc=example,dc=in  
changetype: modify  
add: memberOf  
memberOf: cn=appdev-team,dc=example,dc=in  
  
dn: cn=charile,ou=appdev,dc=example,dc=in  
changetype: modify  
add: memberOf  
memberOf: cn=devops-team,dc=example,dc=in  
EOF
```

List all the attributes OUs, Groups and Users accounts with default  `admin`  user in LDAP server.

# Search for users with deafult admin   

```bash
$ ldapsearch -x -b dc=example,dc=in -D "cn=admin,dc=example,dc=in" -w password -s sub "objectclass=*"
```

# Provide Read Access to LDAP User

To grant read access to specific LDAP users ex:`amrutha`, modify the ACLs (Access Control Lists) using the  `ldapmodify`  command. This ensures that authorized users can query LDAP directory information.

This can be done by LDAP default admin attributes `cn=admin,cn=config.`

# Check oclAccess permission.  

```bash
$ ldapsearch -Y EXTERNAL -Q -H ldapi:/// -LLL -o ldif-wrap=no -b cn=config '(objectClass=olcDatabaseConfig)' olcAccess  
```

# Modification to grant read access to the user "amrutha"  

```bash
ldapmodify -H ldapi:/// -Y EXTERNAL << EOF  
dn: olcDatabase={1}mdb,cn=config  
changetype: modify  
add: olcAccess  
olcAccess: {2}to * by dn="cn=amrutha,ou=devops,dc=example,dc=in" read  
EOF
```

# Verify Configuration

After configuring the LDAP server and granting access permissions, it’s essential to verify the setup, including listing all attributes and confirming access for the user “amrutha”.

To ensure that user “amrutha” has the necessary access permissions and retrieve all relevant attributes associated with their account, execute the following command:

# Search for particular user or attributes like cn,sn,groupOfNames and memberOf with Created LDAP admin  

```bash
ldapsearch -x -D "cn=amrutha,ou=devops,dc=example,dc=in" -w Amrutha@123 -b "dc=example,dc=in"
```

This command retrieves attributes such as  `cn`  (common name),  `sn`  (surname),  `groupOfNames`, and  `memberOf`  for the user "amrutha" within the specified LDAP directory structure. It verifies that the user can successfully access and retrieve the required information.

