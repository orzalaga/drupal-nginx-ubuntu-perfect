```
                      _                   
                     | |                  
   ___  _ __ ______ _| | __ _  __ _  __ _ 
  / _ \| '__|_  / _` | |/ _` |/ _` |/ _` |
 | (_) | |   / / (_| | | (_| | (_| | (_| |
  \___/|_|  /___\__,_|_|\__,_|\__, |\__,_|
                               __/ |      
                              |___/       
```

# Instalación Perfecta de Drupal con Nginx en VPS Ubuntu

Comparto una serie de documentos que mediante una recopilacion de informacion la cual fui encontrando en internet estoy publicando para realizar unas instalacion perfecta para Drupal 9 en un servidor VPS con Ubuntu.

Cualquier comentario y mejora es bienvenida. 

---
![](assets/20220412_180731_cdnlogo.com_drupal.svg)
 ---

## Pre-requisitos

1. Ubuntu 20.04
2. Acceso root SSH
3. sudo privilegios 

### Actualizamos Sistema Operativo
```bash
$ sudo apt update && sudo apt upgrade
```

### Instalación de Nginx
Los paquetes de Nginx están disponibles en repositorios predeterminados de Ubuntu 20.04. Puede instalarlo sin Problemas desde SSH con privilegios de administrador

```bash
$ sudo apt instalar nginx
```
Después de aceptar el procedimiento, el comando `apt` instalará **Nginx** y cualquier dependencia necesaria en su servidor.

### Instalar PHP y PHP-FPM

Para la instalación de PHP, recomendamos utilizar [ppa:ondrej/php](https://launchpad.net/~ondrej/+archive/ubuntu/php) PPA, que proporciona las últimas versiones de PHP para sistemas Ubuntu. Use el siguiente par de comandos para agregar el PPA a su sistema.

```bash
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php
```
Luego instalamos PHP 8.1, siendo esta una de las versiones recomendadas para [Drupal 9.x.](https://www.drupal.org/docs/system-requirements/php-requirements) Para esta instalación simplemente ejecute los siguientes comandos para la instalación del paquetes PHP y PHP-FPM.

```bash
$ apt update
$ sudo apt install php8.1 php8.1-fpm
```

>  **Tip:** Cuando está utilizando PHP-FPM. Todas las configuraciones de los módulos PHP residen en el directorio /etc/php/8.0/fpm.

Después de instalar los paquetes anteriores, el servicio **php8.1-fpm** se iniciará automáticamente. Puede asegurarse escribiendo el siguiente comando en la terminal.
```bash
$ sudo systemctl status php8.1-fpm
● php8.1-fpm.service - The PHP 8.1 FastCGI Process Manager
     Loaded: loaded (/lib/systemd/system/php8.1-fpm.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-04-12 23:22:07 EDT; 5s ago
       Docs: man:php-fpm8.1(8)
    Process: 49492 ExecStartPost=/usr/lib/php/php-fpm-socket-helper install /run/php/php-fpm.sock /etc/php/8.1/fpm/pool.d/www.conf 81 (code=exited, status=0/SUCCESS)
   Main PID: 49478 (php-fpm8.1)
     Status: "Ready to handle connections"
      Tasks: 3 (limit: 9480)
     Memory: 7.6M
     CGroup: /system.slice/php8.1-fpm.service
             ├─49478 php-fpm: master process (/etc/php/8.1/fpm/php-fpm.conf)
             ├─49490 php-fpm: pool www
             └─49491 php-fpm: pool www
Apr 12 23:22:07 drupal.server.net systemd[1]: Starting The PHP 8.1 FastCGI Process Manager...
Apr 12 23:22:07 drupal.server.net systemd[1]: Started The PHP 8.1 FastCGI Process Manager.
```
> Buscar la linea: **Active: active (running)** since Que nos indica que esta corriendo el servicio

## Instalación de Librerías PHP

Instalamos algunas librerías que pueden ser necesarias para un funcionamiento inicial de Drupal. 

```bash
sudo apt install php8.1-bcmath php8.1-curl php8.1-gd php8.1-mbstring php8.1-mysql php8.1-xml php8.1-zip
```

## Configurar bloques de servidor
Al emplear el servidor web Nginx, se pueden usar  _bloques de servidor_  (similares a los hosts virtuales o VirtualHost de Apache) para almacenar la configuración y alojar más de un dominio en un único servidor. Configuraremos un dominio llamado  **example.com**, pero debería  **cambiarlo por su propio nombre de dominio**. 

Nginx en Ubuntu 20.04 tiene habilitado un bloque de servidor por defecto, que está configurado para suministrar documentos desde un directorio en  `/var/www/html`. Si bien esto funciona bien para un solo sitio, puede ser difícil de manejar si aloja varios sitios. En vez de modificar  `/var/www/html`, vamos a crear una estructura de directorios dentro de  `/var/www`  para nuestro sitio  **example.com**  y dejaremos  `/var/www/html`  como directorio predeterminado que se suministrará si una solicitud de cliente no coincide con nuestros dominios.

Cree el directorio para  **example.com**  como se muestra a continuación, usando el indicador  `-p`  para crear cualquier directorio principal necesario:

```bash
$ sudo mkdir -p /var/www/example.com/html
```
> **Nota:** Recuerde que el dominio *example.com* debe ser cambiado por el dominio que va a configurar en el primer bloque

A continuación, asigne la propiedad del directorio con la variable de entorno  `$USER`:

```bash
$ sudo chown -R $USER:$USER /var/www/example.com/html
```
Con la variable de entorno `$USER` los permisos de los roots web deberían ser correctos si no modificó el valor  `umask`, que establece permisos de archivos predeterminados. Para asegurarse de que sus permisos sean correctos y permitir al propietario leer, escribir y ejecutar los archivos, y a la vez conceder solo permisos de lectura y ejecución a los grupos y terceros, puede ingresar el siguiente comando

```bash
$ sudo chmod -R 755 /var/www/example.com
```
A continuación, creamos una página de ejemplo  `index.php`  utilizando  `nano`  o su editor favorito:

```bash
$  nano /var/www/example.com/html/index.html
```

Dentro de ella, agregue el siguiente ejemplo de HTML:

```html
<html>
    <head>
        <title>Hola Mundo!</title>
    </head>
    <body>
        <h1>Activo! Su Servidor esta funcionando correctamente</h1>
    </body>
</html>
```

> Cuando termine, escriba en  `CTRL`  y  `X`, y luego,  `Y`  y  `ENTER`,
> para guardar y cerrar el archivo.

Para que Nginx muestre correctamente este contenido, es necesario generar un **bloque** de servidor con las directivas correctas. En vez de modificar el archivo de configuración predeterminado directamente, crearemos uno nuevo en `/etc/nginx/sites-available/example.com`

Péguelo en el siguiente bloque de configuración, similar al predeterminado, pero actualizado para nuestro nuevo directorio y nombre de dominio:

```bash
server {
        listen 80;
        listen [::]:80;

        root /var/www/example.com/html;
        index index.php index.html index.htm;

        server_name example.com www.example.com;

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        }
}
```
Observe que actualizamos la configuración `root` en nuestro nuevo directorio y el `server_name` para nuestro nombre de dominio. Adicionalmente, agregamos la configuración de `php-fpm`.

Guarde sus cambios en el archivo de configuración y cree un enlace al directorio habilitado del sitio.

```bash 
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```
Ahora, contamos con dos bloques de servidor habilitados y configurados para responder a las solicitudes conforme a las directivas  `listen`  y  `server_name` 

-   `example.com`: responderá a las solicitudes de  `example.com`  y  `www.example.com`.
-   `default`: responderá a cualquier solicitud en el puerto 80 que no coincida con el o los dominios configurados en cada bloque.

Para evitar un problema de memoria de depósito de hash que pueda surgir al agregar nombres de servidor, es necesario aplicar ajustes a un valor en el archivo  `/etc/nginx/nginx.conf`. 

Abra el archivo:
```bash
$ sudo nano /etc/nginx/nginx.conf
```
Encuentre la directiva `server_names_hash_bucket_size` y borre el símbolo `#` para eliminar el comentario de la línea. Si utiliza nano, presione `CTRL` y `w` para buscar rápidamente palabras en el archivo.

```bash
http {
    server_names_hash_bucket_size 64;
}
```

Guarde y cierre el archivo cuando termine.

A continuación, comprobemos que no haya errores de sintaxis en ninguno de sus archivos de Nginx hasta el momento. Ejecute el siguiente comando:

```bash 
$ sudo nginx -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Si no hay problemas, reinicie Nginx para habilitar los cambios:

```bash
$ sudo systemctl restart nginx
```
Con esto, Nginx debería proporcionar su nombre de dominio. Puede probarlo visitando `http://example.com`, donde debería ver algo como esto:

**Activo! Su Servidor esta funcionando correctamente**