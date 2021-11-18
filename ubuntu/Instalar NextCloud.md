## Instalar nginx con php y mysql

Fuentes:

<https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-20-04-es>

<https://www.howtoforge.com/tutorial/ubuntu-nginx-nextcloud/>

### Instalar nginx

```
apt-get update
apt-get install nginx
```

Ingresar mediante un browser:

```
http://server_domain_or_IP
```

![nginx_default.png](nginx_default.png?fileId=2945#mimetype=image%2Fpng&hasPreview=true)

### Instalar mysql

```
apt-get install mysql-server
mysql_secure_installation
```

\
Crear base de datos y usuario

```
$ mysql
MariaDB [(none)]> create database nextcloud;
MariaDB [(none)]> create user nextclouduser@localhost identified by '12345678';
MariaDB [(none)]> grant all privileges on nextcloud.* to nextclouduser@localhost identified by '12345678';
MariaDB [(none)]> flush privileges;
```

### Instalar php

```
apt-get install php-fpm php-mysql
```

Cree el directorio web root para **your_domain** de la siguiente manera:

```
mkdir /var/www/your_domain
chown -R odroid:odroid /var/www/your_domain
```

Crear el archivo /etc/nginx/sites-available/your_domain

```
vim /etc/nginx/sites-available/your_domain
server {
    listen 80;
    server_name your_domain www.your_domain;
    root /var/www/your_domain;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```

Activar el domain

```
ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

Crear el archivo /var/www/your_domain/info.php

```
<?php
  phpinfo();
?>
```

Abrir con un browser:

```
http://server_domain_or_IP/info.php
```

Tiene que aparecer la info de php.

### Instalar y nginx php-fpm:

Crear la carpeta para el dominio (en nginx) para probar nginx-php:

```
mkdir /var/www/your_domain
```

Creamos el archivo de configuracion:

```
vim /etc/nginx/sites-available/your_domain
server {
    listen 80;
    server_name 192.168.1.248;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Habilitamos el dominio y probamos la configuración

```
ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

Creamos el archivo info.php:

```
vim /var/www/your_domain/info.php
<?php
phpinfo();
?>
```

Actualizamos los permisos en el directorio:

```
chmown -R www-data:nogroup /var/www/your_domain
```

En un browser abrimos la pagina http://192.168.1.248/info.php y debemos ver la info de php.

### Crear la base de datos para nextcloud

```
$ mysql
MariaDB [(none)]> create database nextcloud;
MariaDB [(none)]> create user nextclouduser@localhost identified by '12345678';
MariaDB [(none)]> grant all privileges on nextcloud.* to nextclouduser@localhost identified by '12345678';
MariaDB [(none)]> flush privileges;
```

### Install and Configure PHP7.4-FPM

De forma predeterminada, Ubuntu 20.04 viene con la versión PHP 7.4.

Vamos a instalar los paquetes PHP y PHP-FPM necesarios para Nextcloud usando el siguiente comando:

```
apt-get install php-fpm php-curl php-cli php-mysql php-gd php-common php-xml php-json php-intl php-pear php-imagick php-dev php-common php-mbstring php-zip php-soap php-bz2
```

Una vez completada la instalación, configuraremos los archivos php.ini para php-fpm y php-cli. \
Vaya al directorio '/etc/php/7.4'.

```
cd /etc/php/7.4/
```

Edite los archivos php.ini para php-fpm y php-cli usando vim.

```
vim fpm / php.ini 
vim cli / php.ini
```

Descomentar la línea 'date.timezone' y cambie el valor con su propia zona horaria.

```
date.timezone = Asia / Yakarta
```

Descomente la línea 'cgi.fix_pathinfo' y cambie el valor a '0'.

```
cgi.fix_pathinfo = 0
```

\
Guardar y Salir.

A continuación, edite la configuración del grupo php-fpm 'www.conf'.

```
vim fpm / pool.d / www.conf
```

```
Descomente esas líneas a continuación.env [HOSTNAME] = $ HOSTNAME
env [RUTA] = / usr / local / bin: / usr / bin: / bin
env [TMP] = / tmp
env [TMPDIR] = / tmp
env [TEMP] = / tmp
```

Reinicie el servicio PHP7.4-FPM y habilítelo para que se inicie cada vez que se inicie el sistema.

```
systemctl restart php7.4-fpm 
systemctl enable php7.4-fpm
```

Ahora verifique el servicio PHP-FPM usando el siguiente comando.

```
ss -xa | grep php 
systemctl status php7.4-fpm
```

Y obtendrá que php-fpm esté funcionando bajo el archivo sock '/run/php/php7.4-fpm.sock'.

### Instalar NextCloud

Antes de descargar el código fuente de nextcloud, asegúrese de que el paquete de descompresión esté instalado en el sistema. Si no tiene el paquete, instálelo usando el comando apt a continuación.

```
apt-get install wget unzip zip
```

\
Ahora vaya al directorio '/ var / www' y descargue la última versión de Nextcloud usando el siguiente comando.

```
cd /var/www/
wget -q https://download.nextcloud.com/server/releases/latest.zip
```

Extraiga el código fuente de Nextcloud y obtendrá un nuevo directorio 'netxcloud', cambie la propiedad del directorio nextcloud al usuario 'www-data'.

```
unzip -qq latest.zip
sudo chown -R www-data:www-data /var/www/nextcloud
```

Como resultado, Nextcloud se ha descargado en el directorio '/var/www/nextcloud' y será el directorio raíz de la web.

Ahora vaya al directorio '/etc/nginx/sites-available' y cree un nuevo archivo de host virtual 'nextcloud'.

```
cd /etc/nginx/sites-available/
vim nextcloud
```

\
Allí, pegue la siguiente configuración de host virtual de nextcloud.

```
upstream php-handler {
    #server 127.0.0.1:9000;
    server unix:/var/run/php/php7.4-fpm.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name cloud.hakase-labs.io;
    # enforce https
    return 301 https://$server_name:443$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name cloud.hakase-labs.io;

    # Use Mozilla's guidelines for SSL/TLS settings
    # https://mozilla.github.io/server-side-tls/ssl-config-generator/
    # NOTE: some settings below might be redundant
    ssl_certificate /etc/letsencrypt/live/cloud.hakase-labs.io/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cloud.hakase-labs.io/privkey.pem;

    # Add headers to serve security related headers
    # Before enabling Strict-Transport-Security headers please read into this
    # topic first.
    #add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;
    #
    # WARNING: Only add the preload option once you read about
    # the consequences in https://hstspreload.org/. This option
    # will add the domain to a hardcoded list that is shipped
    # in all major browsers and getting removed from this list
    # could take several months.
    add_header Referrer-Policy "no-referrer" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Download-Options "noopen" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Permitted-Cross-Domain-Policies "none" always;
    add_header X-Robots-Tag "none" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Remove X-Powered-By, which is an information leak
    fastcgi_hide_header X-Powered-By;

    # Path to the root of your installation
    root /var/www/nextcloud;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # The following 2 rules are only needed for the user_webfinger app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;

    # The following rule is only needed for the Social app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/webfinger /public.php?service=webfinger last;

    location = /.well-known/carddav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host:$server_port/remote.php/dav;
    }

    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    # Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;

    location / {
        rewrite ^ /index.php;
    }

    location ~ ^\/(?:build|tests|config|lib|3rdparty|templates|data)\/ {
        deny all;
    }
    location ~ ^\/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }

    location ~ ^\/(?:index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+)\.php(?:$|\/) {
        fastcgi_split_path_info ^(.+?\.php)(\/.*|)$;
        set $path_info $fastcgi_path_info;
        try_files $fastcgi_script_name =404;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param HTTPS on;
        # Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        # Enable pretty urls
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ ^\/(?:updater|oc[ms]-provider)(?:$|\/) {
        try_files $uri/ =404;
        index index.php;
    }

    # Adding the cache control header for js, css and map files
    # Make sure it is BELOW the PHP block
    location ~ \.(?:css|js|woff2?|svg|gif|map)$ {
        try_files $uri /index.php$request_uri;
        add_header Cache-Control "public, max-age=15778463";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        # Before enabling Strict-Transport-Security headers please read into
        # this topic first.
        #add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;
        #
        # WARNING: Only add the preload option once you read about
        # the consequences in https://hstspreload.org/. This option
        # will add the domain to a hardcoded list that is shipped
        # in all major browsers and getting removed from this list
        # could take several months.
        add_header Referrer-Policy "no-referrer" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-Download-Options "noopen" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Permitted-Cross-Domain-Policies "none" always;
        add_header X-Robots-Tag "none" always;
        add_header X-XSS-Protection "1; mode=block" always;

        # Optional: Don't log access to assets
        access_log off;
    }

    location ~ \.(?:png|html|ttf|ico|jpg|jpeg|bcmap)$ {
        try_files $uri /index.php$request_uri;
        # Optional: Don't log access to other assets
        access_log off;
    }
}
```

Guardar y Salir.

Habilite el host virtual y pruebe la configuración, y asegúrese de que no haya ningún error.

```
ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
nginx -t
```

Ahora reinicie el servicio PHP7.4-FPM y el servicio nginx usando el comando systemctl a continuación.

```
systemctl restart nginx
systemctl restart php7.4-fpm
```

## **Post-instalación de Nextcloud**

Abra su navegador web y escriba la dirección URL de nextcloud.

```
http://server_domain_or_IP
```

Completar los datos para crear usuario admin, directorio de datos y conexión a la base de datos