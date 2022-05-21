# NUWE-Schneider

# Team 5:
- [Marzin](https://www.linkedin.com/in/martin-shell/)
- [Alex](https://www.linkedin.com/in/a96lex/)
- [Cristina](https://www.linkedin.com/in/cristina-outeda-rua/)

Brief description here of the challenge

# Configuring vm network and discovering running ports

We enable the machine to be exposed in our network by enabling the Bridge Adapter.
![setup](./img/machine_setup.png)

After, we run `sudo nmap -sV 192.168.1.138/24`. We know this ip adress because it is our wifi interface ip adress. We run the scan for the whole subnet.

The output for the virtual machine is:
```bash
Nmap scan report for 192.168.1.145
Host is up (0.00050s latency).
Not shown: 985 closed tcp ports (reset)
PORT     STATE SERVICE
443/tcp  open  https
| ssl-cert: Subject: commonName=GEOHOME-DC.geohome.com
| Subject Alternative Name: DNS:GEOHOME-DC.geohome.com
| Not valid before: 2022-05-19T03:32:36
|_Not valid after:  2023-05-18T00:00:00
|_ssl-date: 2022-05-21T12:45:32+00:00; 0s from scanner time.
|_http-title: Not Found
| tls-alpn:
|_  http/1.1
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
|_ssl-date: 2022-05-21T12:45:32+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=GEOHOME-DC.geohome.com
| Subject Alternative Name: DNS:GEOHOME-DC.geohome.com
| Not valid before: 2022-05-19T03:32:36
|_Not valid after:  2023-05-18T00:00:00
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
| ssl-cert: Subject: commonName=GEOHOME-DC.geohome.com
| Subject Alternative Name: DNS:GEOHOME-DC.geohome.com
| Not valid before: 2022-05-19T03:32:36
|_Not valid after:  2023-05-18T00:00:00
|_ssl-date: 2022-05-21T12:45:32+00:00; 0s from scanner time.
3306/tcp open  mysql
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
| mysql-info:
|   Protocol: 10
|   Version: 8.0.29
|   Thread ID: 28
|   Capabilities flags: 65535
|   Some Capabilities: IgnoreSigpipes, Speaks41ProtocolOld, ConnectWithDatabase, ODBCClient, Speaks41ProtocolNew, Support41Auth, SupportsCompression, SupportsLoadDataLocal, SwitchToSSLAfterHandshake, LongPassword, LongColumnFlag, InteractiveClient, IgnoreSpaceBeforeParenthesis, FoundRows, SupportsTransactions, DontAllowDatabaseTableColumn, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: \x0BC7]L\x06GJa\x08wT\x0FK.^4jp\x0C
|_  Auth Plugin Name: caching_sha2_password
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
5000/tcp open  upnp
MAC Address: 08:00:27:B2:23:49 (Oracle VirtualBox virtual NIC)
```

After checking all adresses in the browser, we found:

 - Port 80: A landing page
![image](https://user-images.githubusercontent.com/62766970/169664577-384db182-fc55-459f-abd2-fd903bd0be5d.png)


 - Port 5000: An API
 Calling the base path gives us a json response `There is nothing to see here (I guess)`
 After inspecting response headers, we see that the app is running a `Werkzeug/2.1.2` server with `Python/3.7.0`.
 We suppose we have found a Flask API. Calling the `/admin` endpoint returns the following message `Missing Authorization Header`. Since the `/admin` route exists, we also suspect this could be a `django` backend.
 
 By running `gobuster` we fond some interesting endpoints.
 ![image](https://user-images.githubusercontent.com/62766970/169664909-129aa774-1b41-45b0-a692-4b540c2a1092.png)

 
 After some manual trial and error, we found that the `/index.php` route shows a "comment posting" interface.

## Stored XSS

In the websote available at the `index.php` route, there is a comment box:


![image](https://user-images.githubusercontent.com/62766970/169664619-800fe0e5-66d9-4fcc-b5e3-41bd79431ce3.png)

We realised that is easy to embed custom HTML, thus javaScript code in the webpage by wrapping it in a `script`tag. We demonstrate it by adding an alert. 

We added 

```html
<script>alert("Sí va a ser tan facil ajaj")</script>
```
Which results in:

![image](https://user-images.githubusercontent.com/62766970/169664753-647d9691-b1b3-44b1-8f87-62ed6df59fa0.png)

This messages persist across sessions and different devices, which means they are stored on the server.

## ALWAYS_CHECK_COMMITS
We were able to access a wordpress server after sending a get request to `/robots.txt` and finding the following:

```
# wp.geohome.com
```
We add the following line to our `/etc/hosts´:

´´´
192.168.1.148 wp.geohome.com
´´´

Then we can access the website using the https protocol through the browser.

After being able to access the wordpress site hosted on the machine, we found at the very bottom of the page a github url.
![image](https://user-images.githubusercontent.com/62766970/169664833-6a354abd-43d7-45b0-8bb9-a5e03cb27d1a.png)

Checking the commits message, we find a secret encoding key that was attempted to be removed in [this commit](https://github.com/geohome-dev/GeoAPI/commit/e82c17ed045e205a2ea07a354ae5b39c8b7d7ea0):

![unsafe!](https://user-images.githubusercontent.com/62766970/169665191-ba5bb4ed-bc6e-40da-b03f-b142355a43c4.png)


The encoding key is `Ge0HomeIsThePlaceWhereFantasyMeetsReality`

We already knew the routes of the flask application, so the next step was easy :)

We created a user called `marcin`, one of the teammates. We then used [jwt.io](https://jwt.io/) to decode, change the user name to `admin` and encode using the secret key:

![image](https://user-images.githubusercontent.com/62766970/169665148-ebcb6427-49f9-40f0-b0c8-4afe3a16ce2e.png)


We then ping the `/admin endpoint` adding the token to the Authentication header and got this response:

![image](https://user-images.githubusercontent.com/62766970/169665133-e128d7e6-4e1e-40aa-a307-09c87e235049.png)


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

