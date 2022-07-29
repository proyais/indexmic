# Seguridad

## General

IndexMic es bifurcación de [ShinyProxy](https://www.shinyproxy.io/). ShinyProxy es una
herramienta de fuente abierta que permite desplegar aplicaciones Shiny. En caso de
descrubrir problemas de seguridad directamente en IndexMic o en ShinyProxy por favor
contactar a [Proyecto AIS](soporte@proyais.com), reportar el problema en el repositorio
oficial de [GitLab](https://gitlab.proyais.com/proyectoais/indexmic) o reportar directamente
el problema en el repositorio de [GitHub](https://github.com/openanalytics/shinyproxy/issues)
de ShinyProxy sin exponer ningún credencial, contraseña, dirección IP, etc.

## Recomendaciones

### Firewall

Para garantizar la seguridad de IndexMic y de los contenedores que corren dentro de la
máquina o del cluster, se recomienda solo exponer el puerto 443 (en caso de usar HTTPS)
y/o el puerto 80. Los contenedores corren en el rango de los puertos mayores a 2000
(a menos que el usuario lo configure de diferente manera). Aunque por defecto
estos no deberían poder accederse externamente, igualmente se recomienda bloquear
todas las conexiones a estos puertos.

### HTTPS (SSL/TLS)

Para correr IndexMic con HTTPS es necesario utilizar un _reverse proxy_ como Nginx.
En esta sección se mostrará el paso a paso para utilizar IndexMic con Nginx,
utilizando Certbot/Let'sEncrypt para obtener un certificado SSL.

#### Instalar Nginx

Primero instalaríamos nginx, certbot y la configuración especifica de certbot
para nginx.

```
sudo apt install nginx certbot python3-certbot-nginx
```

Al ser instalado ya se puede iniciar el proceso de configuración. En el
directorio `/etc/nginx/sites-enabled` vamos a crear un archivo
con el subdominio que apunta a la dirección IP del servidor
que hostea IndexMic. El ejemplo se se hará con `example.com`.

```
sudo nano /etc/nginx/sites-enabled/example.com
```

Dentro de este archivo vamos a escribir la siguiente configuración:

```
server {
    server_name example.com;
}
```

Para validar que no hayan errores en la configuración de Nginx corremos:

```
sudo nginx -t
```

Si no aparece ningún error podemos recargar Nginx con:

```
sudo systemctl reload nginx
```

#### Obtener el certificado

Para obtener el certificado vamos a utilizar certbot. Una vez configurado Nginx
podemos ejecutar:

```
sudo certbot --nginx -d example.com
```

Aquí iniciará el proceso de obtener e instalar el certificado. Certbot
preguntará si desea direccionar todo el tráfico del puerto 80 (HTTP)
al 443 (HTTPS). Se recomienda seleccionar el opción 2 para direccionar
todo el tráfico.

```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```

Certbot probablemente va a preguntar por otros datos cómo un correo electrónico.

Finalmente, certbot mostrará un mensaje de que la configuración fue exitosa.

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2020-08-18. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

#### Configurar el reverse proxy

Ya con el certificado instalado y configurado con nuestro dominio,
será necesario configurar Nginx para exponer a IndexMic.

Volvemos a editar el archivo que creamos `/etc/nginx/sites-enabled/example.com`.
Este debería tener un contenido similar después de las configuraciones
de Certbot.

```
server {
	server_name example.com;


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	server_name example.com;
    listen 80;
    return 404; # managed by Certbot


}
```

Ahora vamos a editar esta configuración para que Nginx pueda
permitir el acceso a IndexMic.

Dentro del primer bloque de `server` (el que escuchar al puerto 443)
vamos a agregar una `location`.

El argumento `client_max_body_size` se puede modificar para permitir
la subida de archivos. Este indica el límite en tamaño de los archivos
que el usuario podrá subir. Por defecto lo dejamos en 1024M.

```
server {
	server_name example.com;


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    location / {
	    proxy_pass http://127.0.0.1:8080/; # El URI interno donde corre IndexMic
	    client_max_body_size 1024M; # El tamaño máximo del body (esto limita el tamaño máximo de un archivo a subir)
	    proxy_http_version 1.1;
	    proxy_set_header Upgrade $http_upgrade; # Esto nos habilitará las conexion con web-sockets
	    proxy_set_header Connection "upgrade";
	    proxy_read_timeout 600s;
	    proxy_redirect off;
	    proxy_set_header  Host $http_host;
	    proxy_set_header  X-Real-IP $remote_addr;
	    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header  X-Forwarded-Proto $scheme;
    }

}
server {
    if ($host = example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	server_name example.com;
    listen 80;
    return 404; # managed by Certbot


}
```

!Ya debe poder acceder a IndexMic con HTTPS!
