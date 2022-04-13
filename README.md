# Instalación Perfecta de Drupal con Nginx en VPS Ubuntu

Step to Step for Installation

Description

---

![](assets/20220412_180731_cdnlogo.com_drupal.svg)

---

## Pre-requisitos

1. Ubuntu 20.04
2. Acceso root SSH
3. sudo privileges

### Actualizamos Sistema Operativo

```bash
$ sudo apt update && sudo apt upgrade
```

### Instalacion de Nginx

Los paquetes de Nginx están disponibles en repositorios predeterminados de Ubuntu 20.04. Puede instalarlo sin Problemas desde SSH con privilegios de administrador

```bash
$ sudo apt instalar nginx
```

Despues de aceptar el procedimiento, el comando `apt` instalará **Nginx** y cualquier dependencia necesaria en su servidor.

### Instalar PHP

Para la instalación de PHP, recomendamos utilizar [ppa:ondrej/php](https://launchpad.net/~ondrej/+archive/ubuntu/php) PPA, que proporciona las últimas versiones de PHP para sistemas Ubuntu. Use el siguiente par de comandos para agregar el PPA a su sistema.

```bash
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php
```
