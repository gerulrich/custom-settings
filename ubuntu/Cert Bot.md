## Configurar Cert Bot con nginx

Fuente: <https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04-es>

Los certificados van a quedar instalados en /etc/letsencrypt/live

Instalar cert bot:

```
apt-get install certbot python3-certbot-nginx
```

Configurar un dominio en nginx (reemplazar example.com con el dominio correspondiente):

```
vim /etc/nginx/sites-available/example.com

server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Obtener certificado ssl

```
certbot --nginx -d example.com -d www.example.com
```

Esto ejecuta `certbot` con el complemento `--nginx`, usando `-d` para especificar los nombres de dominio para los que queremos que el certificado sea válido.

Si es la primera vez que ejecuta `certbot`, se le pedirá que ingrese una dirección de correo electrónico y que acepte las condiciones de servicio. Después de esto, `certbot` se comunicará con el servidor de Let’s Encrypt y realizará una comprobación a fin de verificar que usted controle el dominio para el cual solicite un certificado.

Si la comprobación se realiza correctamente, `certbot` le preguntará cómo desea configurar sus ajustes de HTTPS:

```
OutputPlease choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```

Verificar la renovación automática de Certbot

```
systemctl status certbot.timer
```

Para probar el proceso de renovación, puede hacer un simulacro con `certbot`:

```
sudo certbot renew --dry-run