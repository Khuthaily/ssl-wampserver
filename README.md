# ssl-wampserver
Create a local SSL certificate

## How to add SSL to WampServer and Enable `https://` in Google Chrome

### 1. Get OpenSSL
1.1. Download _OpenSSL_ for your Operating System (32bit vs 64bit) from:

http://slproweb.com/products/Win32OpenSSL.html


I selected the `EXE` option of `Win64 OpenSSL v1.1.1 Light`. The `Light` version is recommended for users.

1.2. Install **OpenSSL**, during the installation process, select **Copy OpenSSL DLLs** to the _OpenSSL binaries (/bin)_ directory.

### 2. Generate an RSA Private Key
2.1. cd to

    /c/OpenSSL-Win32/bin

2.2. Create a directory here to save the key in it. Replace _[website]_ with the name of your project.

2.3. Enter this command line:

    openssl genrsa -out [website]/[website].key 2048

2.4. Now, the file (key) **[website].key** will be available in the _[website]_ directory

### 3. Generate a v3.ext File
3.1. In the directory _[website]_, create a file with the extension **.ext**. I will call mine **v3.ext**

3.2. Paste the following (the region you want to change is in the DNS region):

    authorityKeyIdentifier=keyid,issuer
    basicConstraints=CA:FALSE
    keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
    subjectAltName = @alt_names

    [alt_names]
    DNS.1 = [website].test
    DNS.2 = *.[website].test

### 4. Generate a CSR (Certificate Signing Request)
4.1. Type

    openssl req -new -key [website]/[website].key -out [website]/[website].csr

4.2. During this process, you will be asked the following (sample answers are included)

    Country Name (2 letter code) [AU]:US
    State or Province Name (full name) [Some-State]:State
    Locality Name (eg, city) []:City
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Company
    Organizational Unit Name (eg, section) []:Web Development
    Common Name (e.g. server FQDN or YOUR name) []:[website].test
    Email Address []:support@[website].com
    A challenge password []: ( LEAVE THIS BLANK; Just click ENTER )
    An optional company name []: ( LEAVE THIS BLANK; Just click ENTER )

Now, a **[website].csr** will be generated in the directory _[website]_.

### 5. Generate a Self-Signed Certificate
5.1. Use the command:

    openssl x509 -req -days 365 -in [website]/[website].csr -signkey [website]/[website].key -out [website]/[website].crt -sha256 -extfile [website]/v3.ext

Now, **[website].crt** will be generated in the directory _[website]_. Finally, this is our self-signed certificate!

### 6. Prepare Apache to accept SSL
6.1. In the **httpd.conf** file, make sure these lines are not commented out:

    LoadModule authn_socache_module modules/mod_authn_socache.so
    LoadModule ssl_module modules/mod_ssl.so
    LoadModule socache_shmcb_module modules/mod_socache_shmcb.so

6.2. In _c:/wamp/bin/php/phpX.X.X_, edit the file **phpForApache.ini**, make sure the line below is not commented out:

    extension=openssl

### 7. Adjust Virtual Host
First, make sure the fild begins with

    Listen 443 https

Instead of

    <VirtualHost *:80>
        ServerName [website].test
        DocumentRoot "${INSTALL_DIR}/www/[website]/public"
        <Directory  "${INSTALL_DIR}/www/[website]/public/">
            Options +Indexes +Includes +FollowSymLinks +MultiViews
            AllowOverride All
            Require local
        </Directory>
    </VirtualHost>

Use

    <VirtualHost *:80>
        ServerName [website].test
        ServerAlias *.[website].test
        DocumentRoot "${INSTALL_DIR}/www/[website]/public"
        Redirect permanent /secure https://[website].test
    </VirtualHost>

    <VirtualHost *:443>
       DocumentRoot "${INSTALL_DIR}/www/[website]/public"
       ServerName [website].test
       ServerAlias *.[website].test
       ServerAdmin support@[website].com
       SSLEngine on
       SSLCertificateFile "c:/OpenSSL-Win64/bin/[website]/[website].crt"
       SSLCertificateKeyFile "c:/OpenSSL-Win64/bin/[website]/[website].key"
       SSLCertificateChainFile "c:/OpenSSL-Win64/bin/[website]/[website].crt"
       <directory "${INSTALL_DIR}/www/[website]/public/">
          SSLOptions +StdEnvVars
          Options +Indexes +Includes +FollowSymLinks +MultiViews
          AllowOverride All
          Require local
          Allow from 127.0.0.1 localhost ::1
       </directory>
    </VirtualHost>

### 8. .htaccess
Adjust the root directory of your project; in **Laravel**, this is the _public_ folder. Add these lines after `RewriteEngine On` line:

    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R,L]
    
### Feedback
Feedback is highly appreciated.
