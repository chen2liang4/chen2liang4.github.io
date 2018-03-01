---
title: Authenticate and authorize users using windows AD via LDAP
---

updated 2018-03-01: this post is origin version of the [article](https://www.mongodb.com/blog/post/how-to-configure-LDAP-authentication-for-mongodb), which is posted on mongoDB blog.

This is an supplement of [article](https://docs.mongodb.com/manual/tutorial/authenticate-nativeldap-activedirectory/)

## Mini-clinic Windows AD users and groups
We create some AD users and groups for Mini-clinic. You can download [ADExplorer](https://technet.microsoft.com/en-us/sysinternals/adexplorer.aspx) tool to view these objects.
### User objects
```
dn: CN=scott,CN=Users,DC=ecwise,DC=local  
memberof: CN=Mongo Scheduler,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local

dn: CN=page,CN=Users,DC=ecwise,DC=local  
memberof:  CN=Mongo Practitioner,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local

dn: CN=parker,CN=Users,DC=ecwise,DC=local  
memberof: CN=Mongo Pharmacist,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local

dn: CN=Adam,CN=Users,DC=ecwise,DC=local  
memberof: CN=Mongo Auditor,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local

dn: CN=Duke,CN=Users,DC=ecwise,DC=local  
memberof: CN=Mongo DBA,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local
```

### Group objects
```
dc: CN=Mongo Scheduler,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local  
member: CN=scott,CN=Users,DC=ecwise,DC=local

dc:  CN=Mongo Practitioner,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local  
member: CN=page,CN=Users,DC=ecwise,DC=local

dc: CN=Mongo Pharmacist,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local  
member: CN=parker,CN=Users,DC=ecwise,DC=local

dc: CN=Mongo Auditor,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local  
member: CN=Adam,CN=Users,DC=ecwise,DC=local

dc: CN=Mongo DBA,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local  
member: CN=Duke,CN=Users,DC=ecwise,DC=local
```

## Procedure
### Configure TLS/SSL for the server running MongoDB
mongod connect to AD via TLS/SSL by default, configure ldap.transportSecurity in mongoDb configuration file to none to disable TLS/SSL.  
```
ldap:  
    transportSecurity: none
```

This is not recommended, but it would be useful for debugging when getting set up issue.  
#### Ensure AD servier is enabled with TLS/SSL
Use [ldp.exe](https://www.itsupportguides.com/knowledge-base/windows-7/windows-7-how-to-install-the-active-directory-users-and-computers-tools/) to verify if AD servier is over SSL, refer to [http://windowsitpro.com/security/q-how-can-i-easily-verify-ldap-over-ssl-connectivity-my-windows-dcs]  
```
server: cdcorpwindc01.ecwise.local  
port: 636  
SSL: true 
```

If the response includes "Host supports SSL......Established connection", then AD server is over SSL.

#### Configure LDAPS
Export the Root CA certificates from AD server.
1. Click Start, Administrative Tools, Certification Authority
2. Right-click on your CA, and select Properties
3. In the CA Properties window, click on View Certificate
4. In the Certificate window, click the Details tab and click Copy to File
5. In the Certificate Export Wizard window, click Next
6. Select Base-64 encoded X.509 (.CER), and click Next
7. Enter the export name (e.g., c:\corpRootCa.cer), and click Next
8. Click Finish
9. Copy certificate to the Linux server, for example, to /etc/openldap/certs

**Notice:** step 6 is very important. If the certificate is not encoded with Base-64, it won't work for LDAPS.
Edit /etc/openldap/ldap.conf, add a line.
```
TLS_CACERT /etc/openldap/certs/ecwise-root.cer
```

ecwise-root.cer is stored in /resource of the repo.

Don't bother with TLS_CACERTDIR line. TLS_CACERT take the priority.

Use ldapsearch to verify if LDAPS works
```bash
ldapsearch -x -H ldaps://cdcorpwindc01.ecwise.local -b "DC=ecwise,DC=local" -D "CN=TM-EM, OU=Accounts,OU=Chengdu,OU=EC Wise Users,DC=ecwise,DC=local" -W

# -b starting point to search  
# -D specifies the DC with which to authenticate to the server
```

### Create user Administrative role
MongoDB grant AD group privileges instead of AD user! We create roles for AD group, and add AD user into the AD group. When the AD user login mongoDB and will be granted as role which his AD group is assigned.
Disable LDAP authentication, and execute scripts below, its in /init-script/create-user-administrative-role.js 
```bash
mongo create-user-administrative-role.js
```
```javascript
var admin = db.getSiblingDB("admin")
admin.createRole(
    {
        role: "CN=Mongo DBA,OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local",
        privileges: [],
        roles: [ "userAdminAnyDatabase" ]
    }
)
```
### Edit mongoDB configuration file
The configuration should look like
```yaml
    security:
        authorization: "enabled"
        ldap:
            servers: "cdcorpwindc01.ecwise.local"
            userToDNMapping:
            '[
                {
                match: "(.+)",
                ldapQuery: "CN=Users,dc=ecwise,dc=local??sub?(sAMAccountName={0})"
                }
            ]'
            authz:
                queryTemplate: "OU=Groups,OU=EC Wise Users,DC=ecwise,DC=local??sub?(&(objectClass=group)(member={USER}))"
            bind:
                queryUser: "duke"
                queryPassword: "ecwise@123"
    setParameter:
        authenticationMechanisms: 'PLAIN'
```
#### Configure LDAP query template for authorization
Use queryTemplate to find out groups the user belongs to. QueryTemplate pattern is:
> < AD grous DN to search >??sub?< query condition >

So in our configuration, it means search all DN under "OU=EC Wise Users,dc=ecwise,dc=local" and its property objectClass equals group and member equals user DN.

#### Tansform incomming usernames for authentication via AD
Usually, user DN is too long, not easy for remember or use. So userToDNMapping help transform username to full LDAP DN which the system will use it as ID.   
ldapQuery pattern is:
> < AD users DN to search >??sub?< math condition >

So in our configuration, it's to find out property sAMAccountName equals user name under "CN=User,dc=ecwise,dc=local".

#### Configure query credentials
MongoDB requires credentials for performing query on AD server. Using queryUser and queryPassword to specify the user who has permission to perfrom query.

### Create roles and users for Mini-clinic
Execute script(init_script/init_ldap_role_user.js) in repo with DBA user to create roles and users for Mini-clinic.
```bash
mongo -u duke -p --authenticationMechanism PLAIN --authenticationDatabase $external init_ldap_role_user.js 
```
