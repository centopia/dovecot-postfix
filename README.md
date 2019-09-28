### Install and Configure a mail service in CentOs7
-----

Follow this guide (even on freshly instlled VPS) and by the end of the tutorial, 
you will have an email foo@your-domain.tld to send to gmail or whatever.

just follow every line. 


##### INSTALL and configure postfix

```
yum remove sendmail
yum install postfix
alternatives --set mta /usr/sbin/postfix
```
if the last command throws an error saying "/usr/sbin/postfix has **not** been configured as an alternativ" do the following line of command 

```
alternatives --set mta /usr/sbin/sendmail.postfix
```
The edit main.cf and apply changes

```
vi /etc/postfix/main.cf
```
Find these settings and apply the values. YOUR_DOMAIN is the current domain in which your VPS is accessible by. 

```
  myhostname = mail.YOUR_DOMAIN.net
  mydomain = YOUR_DOMAIN.net
  myorigin = $mydomain
  inet_interfaces = all
  mydestination = $myhostname, localhost, $mydomain
  mynetworks = 127.0.0.0/8, /32
  relay_domains = $mydestination
  home_mailbox = Maildir/
 ```
Not restart dovecot and postfix
```
service postfix restart
chkconfig postfix on
```
Configure  iptables

```
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 25 -j ACCEPT
iptables -A INPUT -m state --state NEW -m udp -p udp --dport 25 -j ACCEPT
```

##### Install and configure Dovecot

```
yum install dovecot
vi /etc/dovecot.conf 
``` 
and past the following at the end of the file
 
```
protocols = imap imaps pop3 pop3s
mail_location = maildir:~/Maildir
pop3_uidl_format = %08Xu%08Xv
# Required on x86_64 kernels
login_process_size = 64
```

#### restart server
```
chkconfig --level 345 dovecot on
service nginx restart
service dovecot restart
```

##### squirrelmail

```
 yum install squirrelmail
 ```

##### configure squirrelmail
```
cd /usr/share/squirrelmail/config/
 ./conf.pl
 ```
Navigate using the numbers and change "domain name" and save it

##### HTTPD
```
vi /etc/httpd/conf/httpd.conf
```

and past the following at the end

```
Alias /webmail /usr/share/squirrelmail
<Directory /usr/share/squirrelmail>
Options Indexes FollowSymLinks
RewriteEngine On
AllowOverride All
DirectoryIndex index.php
Order allow,deny
Allow from all
</Directory>
```
##### server start
```
 systemctl restart httpd

 ```
##### check webmail
go to yourdomain/webmail and login using the username and passwd you made





##### resources:


https://tecadmin.net/install-and-configure-postfix-on-centos-redhat/
https://wiki.centos.org/HowTos/postfix
https://linuxtechlab.com/setting-mail-server-postfix-dovecot-squirrelmail/



