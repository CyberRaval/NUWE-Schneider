# NUWE-Schneider

# Team 5:
- [Marzin](https://www.linkedin.com/in/martin-shell/)
- [Alex](https://www.linkedin.com/in/a96lex/)
- [Cristina](https://www.linkedin.com/in/cristina-outeda-rua/)

Brief description here of the challenge

## Break into Windows

Describe here the process to find the flag (you can include videos, images, etc)

## FLAG_NAME_2

Describe here the process to find the flag (you can include videos, images, etc)

## FLAG_NAME_N

Describe here the process to find the flag (you can include videos, images, etc)

We found open ports and software versions behind:

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-05-21 12:41:19Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: geohome.com0., Site: Default-First-Site-Name)
443/tcp  open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: geohome.com0., Site: Default-First-Site-Name)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: geohome.com0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: geohome.com0., Site: Default-First-Site-Name)
3306/tcp open  mysql         MySQL 8.0.29
5000/tcp open  upnp?
```

We found valid usernames in mysql:

```
| mysql-enum:
|   Valid usernames:
|     root:<empty> - Valid credentials
|     netadmin:<empty> - Valid credentials
|     guest:<empty> - Valid credentials
|     test:<empty> - Valid credentials
|     user:<empty> - Valid credentials
|     sysadmin:<empty> - Valid credentials
|     administrator:<empty> - Valid credentials
|     webadmin:<empty> - Valid credentials
|     admin:<empty> - Valid credentials
|     web:<empty> - Valid credentials
```

Performed a bruteforce against these usernames, no luck so far.

Managed to get a session token from /register endpoint.

