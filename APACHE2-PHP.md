## Termux Apache2 & PHP
### Install
```shell
pkg install php apache2 php-apache -y
```

### Start/stop the server
```shell
httpd
pkill httpd
```
Using termux-services:
```shell
sv-enable httpd # enable as a service
sv up httpd # start the service
sv down httpd # stop the service
```

### Test configuration files
```shell
httpd -t
```

### Always reload Apache
Remember to always reload Apache after changing its config and after changing php.ini as well.
```shell
pkill httpd && httpd
```
Using termux-services:
```shell
sv reload httpd && sv up httpd
```

### Configure Apache
- The `ServerRoot` should not be changed;
- The port 80 is not available on Android, so `Listen` is 8080;
- Enable `rewrite_module`;
- Set the default admin email;
- On Termux ´localhost´ can't resolve, so `ServerName` is `127.0.0.1:8080`;
- Disable all access to any folder by default;
- Set the root folder to `~/www`, for when calling `127.0.0.1:8080`;
- Set the default access to the `~/` folder and subfolders;
- Always load the `index.html` file when trying to load a folder;
- Disable access to any `.ht*` file, like `.htaccess`;

Doing all that, your `httpd.conf` should have configs seted up like this:

`$PREFIX/etc/apache2/httpd.conf`
```properties
ServerRoot "/data/data/com.termux/files/usr"
Listen 8080

LoadModule rewrite_module libexec/apache2/mod_rewrite.so

ServerAdmin email@example.com
ServerName 127.0.0.1:8080

<Directory />
    AllowOverride None
    Require all denied
    Options None
</Directory>

DocumentRoot "/data/data/com.termux/files/home/www"
<Directory "/data/data/com.termux/files/home">
    Options FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

<Files ".ht*">
    Require all denied
</Files>
```

### Enable php interpreter on Apache
- Enable `mpm_prefork_module`;
- Disable `mpm_worker_module`;
- Enable `php_module` (put this line before the `<IfModule unixd_module>` line);
- Add `index.php` as the first file to load when trying to load a folder;
- Set the interpreter to use when loading `*.php` files;

Doing all that, your `httpd.conf` should have configs seted up like this:

`$PREFIX/etc/apache2/httpd.conf`
```properties
LoadModule mpm_prefork_module libexec/apache2/mod_mpm_prefork.so
#LoadModule mpm_worker_module libexec/apache2/mod_mpm_worker.so
LoadModule php_module libexec/apache2/libphp.so

<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

<FilesMatch "\.(php*|phtm|phtml|asp|aspx)$">
    SetHandler application/x-httpd-php
</FilesMatch>
```

### Make Apache more secure to Production
- Enable `headers_module`;
- Disable apache http signatures;
- Disable ETag;
- Disable tracing;
- Set secure headers;

Doing all that, your `httpd.conf` should have configs seted up like this:

`$PREFIX/etc/apache2/httpd.conf`
```properties
LoadModule headers_module libexec/apache2/mod_headers.so

ServerName 127.0.0.1:8080
ServerTokens Prod
ServerSignature Off
FileETag None
TraceEnable Off
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
Header always append X-Frame-Options SAMEORIGIN
Header set X-XSS-Protection "1; mode=block"
```

### Make PHP more secure to Production
Create the php.ini file.
```shell
touch $PREFIX/lib/php.ini
```
Download a cacert.pem:
```shell
wget https://curl.se/ca/cacert.pem
mv cacert.pem ~/.ssh/cacert.pem
```
Disable all logs and add cacert:

`$PREFIX/lib/php.ini`
```properties
display_errors = off
display_startup_errors = off
error_reporting = -1
curl.cainfo=/data/data/com.termux/files/home/.ssh/cacert.pem
openssl.cafile=/data/data/com.termux/files/home/.ssh/cacert.pem
```

### Configure V-Hosts
- Create a file for your website at `$PREFIX/etc/apache2/conf.d/*.conf`;
- Termux don't resolve hostnames, so use ports to diferenciate the v-hosts;
- Configure the v-hosts, always put a `ServerName`, otherwise u can get redirect problems;
- If u want, delete the `$PREFIX/etc/apache2/conf.d/placeholder.conf`, as it can confuse u;
- If u want custom log files, u also need to create those files or httpd will break;
- Its good to place the port number on the file name, makes easy to know which ports are being used;

`$PREFIX/etc/apache2/conf.d/8081.[project-b].conf`
```properties
Listen 8081
<VirtualHost *:8081>
    DocumentRoot "/data/data/com.termux/files/home/[project-b]/public_html"
    ServerName [project-b.com]
    ServerAdmin mail@example.com
    ErrorLog "$PREFIX/var/log/apache2/[project-b]/error.log"
    TransferLog "$PREFIX/var/log/apache2/[project-b]/transfer.log"
    CustomLog "$PREFIX/var/log/apache2/[project-b]/access.log" common
    SetEnv ENV production
</VirtualHost>
```
- If u want custom log files, u also need to create those files or httpd will break;
```shell
mkdir $PREFIX/var/log/apache2/[project-b]
touch $PREFIX/var/log/apache2/[project-b]/error.log
touch $PREFIX/var/log/apache2/[project-b]/transfer.log
touch $PREFIX/var/log/apache2/[project-b]/access.log
```
All the `DocumentRoot` should be folders inside the `<Directory "/data/data/com.termux/files/home">` set on the `httpd.conf` file and all the v-hosts will follow those directory permissions.

### Whole files after all configs:
`$PREFIX/etc/apache2/httpd.conf`
```properties
ServerRoot "/data/data/com.termux/files/usr"

Listen 8080

LoadModule mpm_prefork_module libexec/apache2/mod_mpm_prefork.so
LoadModule authn_file_module libexec/apache2/mod_authn_file.so
LoadModule authn_core_module libexec/apache2/mod_authn_core.so
LoadModule authz_host_module libexec/apache2/mod_authz_host.so
LoadModule authz_groupfile_module libexec/apache2/mod_authz_groupfile.so
LoadModule authz_user_module libexec/apache2/mod_authz_user.so
LoadModule authz_core_module libexec/apache2/mod_authz_core.so
LoadModule access_compat_module libexec/apache2/mod_access_compat.so
LoadModule auth_basic_module libexec/apache2/mod_auth_basic.so
LoadModule reqtimeout_module libexec/apache2/mod_reqtimeout.so
LoadModule include_module libexec/apache2/mod_include.so
LoadModule filter_module libexec/apache2/mod_filter.so
LoadModule mime_module libexec/apache2/mod_mime.so
LoadModule log_config_module libexec/apache2/mod_log_config.so
LoadModule env_module libexec/apache2/mod_env.so
LoadModule headers_module libexec/apache2/mod_headers.so
LoadModule setenvif_module libexec/apache2/mod_setenvif.so
LoadModule version_module libexec/apache2/mod_version.so
LoadModule slotmem_shm_module libexec/apache2/mod_slotmem_shm.so
LoadModule unixd_module libexec/apache2/mod_unixd.so
LoadModule status_module libexec/apache2/mod_status.so
LoadModule autoindex_module libexec/apache2/mod_autoindex.so
<IfModule !mpm_prefork_module>
    #LoadModule cgid_module libexec/apache2/mod_cgid.so
</IfModule>
<IfModule mpm_prefork_module>
    #LoadModule cgi_module libexec/apache2/mod_cgi.so
</IfModule>
LoadModule negotiation_module libexec/apache2/mod_negotiation.so
LoadModule dir_module libexec/apache2/mod_dir.so
LoadModule userdir_module libexec/apache2/mod_userdir.so
LoadModule alias_module libexec/apache2/mod_alias.so
LoadModule rewrite_module libexec/apache2/mod_rewrite.so
LoadModule php_module libexec/apache2/libphp.so

<IfModule unixd_module>
    #User daemon
    #Group daemon
</IfModule>

ServerAdmin contato@laravieira.me
ServerName 127.0.0.1:8080
ServerTokens Prod
ServerSignature Off
FileETag None
TraceEnable Off
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
Header always append X-Frame-Options SAMEORIGIN
Header set X-XSS-Protection "1; mode=block"

<Directory />
    AllowOverride None
    Require all denied
    Options None
</Directory>

DocumentRoot "/data/data/com.termux/files/home/www"

<Directory "/data/data/com.termux/files/home">
    Options FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

<FilesMatch "\.(php*|phtm|phtml|asp|aspx)$">
    SetHandler application/x-httpd-php
</FilesMatch>

<Files ".ht*">
    Require all denied
</Files>

ErrorLog "var/log/apache2/error_log"
LogLevel warn

<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common

    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>

    CustomLog "var/log/apache2/access_log" common
</IfModule>

<IfModule alias_module>
    ScriptAlias /cgi-bin/ "/data/data/com.termux/files/usr/lib/cgi-bin/"
</IfModule>

<IfModule cgid_module>
    #Scriptsock cgisock
</IfModule>

<Directory "/data/data/com.termux/files/usr/lib/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>

<IfModule headers_module>
    RequestHeader unset Proxy early
</IfModule>

<IfModule mime_module>
    TypesConfig etc/apache2/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
</IfModule>

<IfModule proxy_html_module>
    Include etc/apache2/extra/proxy-html.conf
</IfModule>

<IfModule ssl_module>
    SSLRandomSeed startup builtin
    SSLRandomSeed connect builtin
</IfModule>

Include etc/apache2/conf.d/*.conf
```
`$PREFIX/lib/php.ini`
```properties
display_errors = off
display_startup_errors = off
error_reporting = -1
```
`$PREFIX/etc/apache2/conf.d/[project-b].conf`
```properties
Listen 8081
<VirtualHost *:8081>
    DocumentRoot "/data/data/com.termux/files/home/[project-b]/public_html"
    ServerName [project-b.com]
    SetEnv ENV production
</VirtualHost>
```
