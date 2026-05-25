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
