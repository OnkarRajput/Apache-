# Apache----

# What is a reverse proxy?

A reverse proxy accepts connections and then routes them to the appropriate backend server. For example, if we have a Ruby application running on 127.0.0.1:3000, we can configure
a reverse proxy to accept connections on HTTP or HTTPS, which can then transparently proxy requests to the ruby backend.


# What are reverse proxies used for?

Backend routing logic/transparent routing
Network ACLs
Logging
URL rewriting
Virtualhost configuration
Easy SSL configuration

# Firstly, ensure that Apache is installed

yum install httpd mod_ssl -y

# Define Apache reverse proxy configuration

vim /etc/httpd/conf.d/www.test.com.conf


=== HTTP

<VirtualHost *:80>

  ServerName test.com

  ServerAlias www.test.com

  DocumentRoot /var/www/test/public

===Redirect any HTTP request to HTTPS

RewriteEngine On

RewriteCond %{HTTPS} off

RewriteRule (.*) https://%{SERVER_NAME}/$1 [R,L]

=== Logging

LogLevel warn

ErrorLog /var/log/httpd/test.error.log

CustomLog /var/log/httpd/test.access.log combined

<Directory /var/www/test/public>

AllowOverride All

</Directory>

<DirectoryMatch "/\.git">

    Require all denied

</DirectoryMatch>

</VirtualHost>


=== HTTPS

<VirtualHost *:443>

 ServerName test.com

 ServerAlias www.test.com

=== Logging

LogLevel warn

ErrorLog /var/log/httpd/test.error.log

CustomLog /var/log/httpd/test.access.log combined

<Directory /var/www/test.com/167fr/public>

AllowOverride All


</Directory>

<DirectoryMatch "/\.git">

Require all denied

</DirectoryMatch>


=== SSL Configuration - uses strong cipher list - these might need to be downgraded if you need to support older browsers/devices

SSLEngine on

SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH

SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1

SSLHonorCipherOrder On

SSLCACertificateFile  /var/www/test.com/sslforfree/ca_bundle.crt

SSLCertificateKeyFile /var/www/test.com/sslforfree/private.key

SSLCertificateFile /var/www/test.com/sslforfree/certificate.crt

===HSTS (optional)

Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains;"

== Remove this if you need to use frames or iframes

Header always set X-Frame-Options DENY

== Prevent MIME based attacks

Header set X-Content-Type-Options "nosniff"

== Reverse proxy configuration

<Location />

ProxyPass http://localhost:8070/

ProxyPassReverse http://localhost:8070/

</Location>


</VirtualHost>

# Enable and start the Apache service

systemctl enable httpd && systemctl start httpd


