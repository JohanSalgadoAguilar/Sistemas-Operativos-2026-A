# Sistemas-Operativos-2026-A


# Servidor NGINX/PHP

**Tecnológico de Estudios Superiores del Oriente de México**  
**INTEGRANTES:** Hernandez Sanchez Christopher Oswaldo  
Salgado Aguilar Johan Raciel  
Gonzalez Blanco Brian Ruben  

**Grupo:** 6S11  
**Profesor:** Gustavo Moises Romero Gonzalez  

---

# Objetivo General

Instalar, configurar y desplegar un entorno de servidor web compuesto por NGINX (versión 1.31.x) y PHP-FPM (versión 8.4.x), compilados e instalados desde su código fuente en un directorio personalizado (`/srv/nginx`) sobre la distribución AlmaLinux, asegurando su integración mediante sockets UNIX y su gestión automatizada a través de servicios SystemD.

# Objetivos Específicos

- Compilar e instalar NGINX 1.31.x y PHP 8.4.x estableciendo `/srv/nginx` como el prefijo de instalación común en un entorno empresarial compatible con RHEL.
- Configurar los usuarios y grupos de sistema requeridos (`nginx` y `php:nginx`) aplicando el principio de menor privilegio para asegurar el entorno de AlmaLinux.
- Habilitar extensiones críticas en PHP (procesamiento de imágenes, manejo de fechas e internacionalización `intl`) resolviendo sus dependencias mediante repositorios nativos y EPEL.
- Crear y registrar los archivos de servicio SystemD para automatizar el arranque de NGINX y PHP-FPM en el inicio del sistema operativo (`multi-user.target`).
- Configurar la comunicación interna utilizando FastCGI a través de un Socket UNIX local para optimizar el rendimiento y la seguridad.
- Validar el funcionamiento del entorno mediante el despliegue de un script de prueba (`phpinfo.php`).

# Desarrollo del Proyecto

## 1. Preparación del Sistema y Dependencias

En AlmaLinux, primero debemos activar el grupo de herramientas de desarrollo y los repositorios necesarios (incluido EPEL para ciertas bibliotecas de desarrollo) para poder compilar todo con éxito:

```bash
# Habilitar repositorio EPEL y herramientas de desarrollo
sudo dnf install -y epel-release
sudo dnf groupinstall -y "Development Tools"

# Instalar dependencias específicas de compilación para NGINX y PHP
sudo dnf install -y pcre-devel zlib-devel openssl-devel libxml2-devel \
 sqlite-devel libcurl-devel libpng-devel libjpeg-turbo-devel \
 libwebp-devel freetype-devel oniguruma-devel libicu-devel \
 pkgconfig
```

## 2. Compilación e Instalación de NGINX (1.31.x)

Descargamos la versión de la rama de desarrollo/estable solicitada, la configuramos con el prefijo `/srv/nginx` e inyectamos flags genéricas de optimización binaria (`-march=x86-64 -mtune=generic`) para mitigar errores de tipo instrucción ilegal generados por la arquitectura del procesador físico en entornos virtualizados:

```bash
wget https://nginx.org/download/nginx-1.31.2.tar.gz

tar -zxvf nginx-1.31.2.tar.gz

cd nginx-1.31.2

./configure \
 --prefix=/srv/nginx \
 --user=nginx \
 --group=nginx \
 --with-http_ssl_module \
 --with-cc-opt="-march=x86-64 -mtune=generic"

make

sudo make install
```

## 3. Compilación e Instalación de PHP (8.4.x)

Descargamos el código fuente de PHP 8.4, asegurándonos de activar PHP-FPM y las extensiones requeridas (`intl`, procesamiento de imágenes vía `gd`, etc.):

```bash
wget https://www.php.net/distributions/php-8.4.1.tar.gz

tar -zxvf php-8.4.1.tar.gz

cd php-8.4.1

./configure \
 --prefix=/srv/nginx \
 --with-config-file-path=/srv/nginx/html \
 --enable-fpm \
 --with-fpm-user=php \
 --with-fpm-group=nginx \
 --enable-intl \
 --with-external-gd \
 --enable-mbstring \
 --with-openssl \
 --with-curl \
 --with-zlib

make

sudo make install
```

## 4. Configuración de PHP-FPM y el Socket UNIX

Copiamos los archivos de configuración por defecto generados en el prefijo de instalación corporativo:

```bash
sudo cp /srv/nginx/etc/php-fpm.conf.default /srv/nginx/etc/php-fpm.conf

sudo cp /srv/nginx/etc/php-fpm.d/www.conf.default /srv/nginx/etc/php-fpm.d/www.conf
```

Modificamos el archivo `/srv/nginx/etc/php-fpm.d/www.conf`:

```ini
user = php
group = nginx

listen = /tmp/php84.sock
listen.owner = php
listen.group = nginx
listen.mode = 0660
```

## 5. Configuración de NGINX para FastCGI

Editamos el archivo `/srv/nginx/conf/nginx.conf`:

```nginx
user nginx nginx;

worker_processes auto;

