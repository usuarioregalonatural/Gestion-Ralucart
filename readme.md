

<p align="center"><img src="https://www.regalonatural.com/img/raluca-art-logo-1537553801.jpg"></p>

### Gestión de Clientes y Proveedores 

NOTA: Actualmente se ha descartado la utilización de dockers dada su complejidad. De momento, hasta hacer funcionar Laravel en Producción, utilizaremos el servidor host sin dockers.

## Tareas de configuración del server

* Redireccionamiento del dominio gestion.regalonatural.com hacia la ruta adecuada del proyecto mediante Apache
* Verificación de la instalación y versiones de Apache, PHP y MySql


### Redirección del dominio


# Versión anterior con dockers

En este proyecto se va a crear la aplicación web de gsetion mediante el uso de dockers utilizando **docker-compose**

El procedimiento seguido se ha obtenido de [https://blog.irontec.com/desarrollando-con-docker/](https://blog.irontec.com/desarrollando-con-docker/)

---
## Ficheros Creados

Todos los ficheros se han creado en una única ruta: 

```
/home/dockers/gestion-natural/pack3
```

Dentro de esa ruta se han generado los siguientes ficheros:

* apache2-foreground
* Dockerfile
* 000-default.conf
* docker-compose.yaml


tambien se ha creado la ruta *apache/web* donde, de momento, residirá la web. Más adelante se cambiará la referencia en *docker-compose* para que apunte a donde realmente está nuestra web en el sistema host.


### Dockerfile de Apache
Este fichero contiene la información necesaria para generar una imagen de apache, en este caso se ha optado por **debian:jessie**

El contenido del fichero es el siguiente:

```
FROM debian:jessie

RUN apt-get update && apt-get install -y apache2-bin apache2.2-common --no-install-recommends && rm -rf /var/lib/apt/lists/*

ENV APACHE_CONFDIR /etc/apache2
ENV APACHE_ENVVARS $APACHE_CONFDIR/envvars

RUN set -ex \
        \
# generically convert lines like
#   export APACHE_RUN_USER=www-data
# into
#   : ${APACHE_RUN_USER:=www-data}
#   export APACHE_RUN_USER
# so that they can be overridden at runtime ("-e APACHE_RUN_USER=...")
    && sed -ri 's/^export ([^=]+)=(.*)$/: ${\1:=\2}\nexport \1/' "$APACHE_ENVVARS" \
    \
# setup directories and permissions
    && . "$APACHE_ENVVARS" \
    && for dir in \
        "$APACHE_LOCK_DIR" \
        "$APACHE_RUN_DIR" \
        "$APACHE_LOG_DIR" \
        /var/www/html \
    ; do \
        rm -rvf "$dir" \
        && mkdir -p "$dir" \
        && chown -R "$APACHE_RUN_USER:$APACHE_RUN_GROUP" "$dir"; \
    done

# logs should go to stdout / stderr
RUN set -ex \
        && . "$APACHE_ENVVARS" \
        && ln -sfT /dev/stderr "$APACHE_LOG_DIR/error.log" \
        && ln -sfT /dev/stdout "$APACHE_LOG_DIR/access.log" \
        && ln -sfT /dev/stdout "$APACHE_LOG_DIR/other_vhosts_access.log"

COPY apache2-foreground /usr/local/bin/
RUN chmod +x /usr/local/bin/apache2-foreground

COPY 000-default.conf /etc/apache2/sites-enabled/000-default.conf
RUN a2enmod proxy proxy_fcgi

WORKDIR /var/www/html

EXPOSE 80

CMD ["apache2-foreground"]


```

A tener muy en cuenta que, en la última línea, hace referencia a un fichero **apache2-foreground** que debe existir en el raiz de nuestro directorio, esto es, en */home/dockers/gestion-natural/pack3*


### Dockerfile de PHP
Se genera un dockerfile propio para instalar la extesión de mysqli

```
FROM php:7.2-fpm
RUN docker-php-ext-install mysqli
```

### docker-compose.yaml

Este fichero realiza el montaje de los docker, la relación entre ellos y los levanta.

```
version: "2"
services:

  mysql:
    image: 'mysql:5.6.35'
    ports:
     - "9306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "vmsn2004"
      MYSQL_DATABASE: regalonatural
    volumes:
        - /home/webs/gestion-natural/DB-Gestion-Ralucart:/var/lib/mysql
  php:
      build: ./docker-php/
      links:
        - mysql:mysqldb
      volumes:
        - /home/webs/gestion-natural/Gestion-Ralucart:/var/www/html
  apache:
    build: ./docker-apache/
    links:
      - php:phpfpm
    volumes:
      - /home/webs/gestion-natural/Gestion-Ralucart:/var/www/html
    ports:
      - "8083:80"


```

A destacar varias cosas en este fichero:

* En las secciones php y apache se mapea con el directorio local *./apache*, esto se debería cambiar para que apuntase al raiz de nuestra web. **Ojo** mapea */var/www/html* y sin embargo en el fichero dockerfile se apunta a */var/www/html/public*.
- [x] Hecho

* Las versiones de mysql, php e incluso apache no son las últimas. Inicialmente lo dejamos así para que funciones y después valoramos hacerle el upgrade necesario.
 - En mysql no podemos subir de la versión 5.6.35 ya que no funciona.
 - En php si tenemos la versión 7.2
 - Apache está en 2.2

* Solo se han mapeado los puertos de apache (8083:80), deberían mapearse también los de mysql para que en el mismo host puedan convivir varios proyectos (de multiples dockers) sin machacarse puertos.
- [x] Hecho. Se ha mapeado también mysql -> 9306:3306

### Modificaciones en httpd en host

Para poder redirigir desde un dominio hacia localhost:puertoxx, es necesario modificar el fichero httpd.conf que está en /etc/httpd/conf

``` bash
<VirtualHost example.com:80>
    ProxyPass / http://localhost:8001
    ProxyPassReverse / http://localhost:8001
</VirtualHost>
```

después reiniciar el servicio con:

```bash
systemctl restart httpd
```

## Tareas adicionales

- Se ha tenido que crear en el sistema host (Centos 7) el usuario **www-data** y agregarlo al grupo **apache**
```bash
useradd -G apache www-data
```

- En la carpeta de la web (por encima de /public) ha habido que darle permisos 777 a **todo** 

#### Pendiente
- [] Revisar y mejorar el tema de permisos


# Paso a Produccion
Estos son los paso para poder poner en producción el proyecto.

### Instalar Composer en Produccion
```bash
cd ~
curl -sS https://getcomposer.org/installer | php
```
Ahora ya tenemos el archivo <code>composer.phar</code> en nuestro home, tenemos que moverlo a la carpeta de binarios para que pueda ejecutar. Lo renombraremos para que sea más sencillo invocarlo.

```bash
mv composer.phar /usr/local/bin/composer
```
Después podemos ejecutar 
```bash
[root@vicsoft ~]# composer
```
y comprobar que ejecuta bien.

### Ejecutar Composer en Produccion
Antes de ejecutar composer tenemos que estar seguros de que estamos situados en el directorio del proyecto (*/home/webs/gestion-natural/Gestion-Ralucart*)

Puede ser que de un error como este:
```bash
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - laravel/framework v5.7.16 requires ext-mbstring * -> the requested PHP extension mbstring is missing from your system.
    - laravel/framework v5.7.16 requires ext-mbstring * -> the requested PHP extension mbstring is missing from your system.
    - Installation request for laravel/framework v5.7.16 -> satisfiable by laravel/framework[v5.7.16].
```

para solventarlo tendremos que instalar lo necesario:

```bash
yum install php-mbstring
```
Ahora ya debería funcionar la ejecución de Composer.

### Configurar Laravel en Producción
Primer paso copiar el fichero de entorno ejemplo como final
```bash
cp .env.example .env
```
Luego editamos el fichero <code>.env</code> y modificamos lo siguiente:
```bash
APP_ENV=production <-- esto
APP_DEBUG=false <-- esto
APP_KEY=SomeRandomString (esto se genera luego)

DB_HOST=localhost <-- esto
DB_DATABASE=gestionatural <-- esto
DB_USERNAME=root <-- esto
DB_PASSWORD=XXXXXXXXX <-- esto
```

Una vez realizadas las modificaciones, guardaremos y generaremos la APP_KEY

```bash
php artisan key:generate
```
luego ir al fichero <code>config/app.php</code> y confirmar que la url es la adecuada

### Configurar Cache
Es bueno recompilar los ficheros de configuración para que el cacheo se haga bien
Dentro del directorio de la aplicación (*/home/webs/gestion-natural/Gestion-Ralucart*)

```bash
php artisan config:cache
```
Después debe aparece un mensaje como este:
```bash
[root@vicsoft Gestion-Ralucart]# php artisan config:cache
Configuration cache cleared!
Configuration cached successfully!
```

### Migrar la base de datos


<p align="center"><img src="https://laravel.com/assets/img/components/logo-laravel.svg"></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/d/total.svg" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/v/stable.svg" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/license.svg" alt="License"></a>
</p>

## About Laravel

Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel attempts to take the pain out of development by easing common tasks used in the majority of web projects, such as:

- [Simple, fast routing engine](https://laravel.com/docs/routing).
- [Powerful dependency injection container](https://laravel.com/docs/container).
- Multiple back-ends for [session](https://laravel.com/docs/session) and [cache](https://laravel.com/docs/cache) storage.
- Expressive, intuitive [database ORM](https://laravel.com/docs/eloquent).
- Database agnostic [schema migrations](https://laravel.com/docs/migrations).
- [Robust background job processing](https://laravel.com/docs/queues).
- [Real-time event broadcasting](https://laravel.com/docs/broadcasting).

Laravel is accessible, yet powerful, providing tools needed for large, robust applications.

## Learning Laravel

Laravel has the most extensive and thorough [documentation](https://laravel.com/docs) and video tutorial library of any modern web application framework, making it a breeze to get started learning the framework.

If you're not in the mood to read, [Laracasts](https://laracasts.com) contains over 1100 video tutorials on a range of topics including Laravel, modern PHP, unit testing, JavaScript, and more. Boost the skill level of yourself and your entire team by digging into our comprehensive video library.

## Laravel Sponsors

We would like to extend our thanks to the following sponsors for helping fund on-going Laravel development. If you are interested in becoming a sponsor, please visit the Laravel [Patreon page](https://patreon.com/taylorotwell):

- **[Vehikl](https://vehikl.com/)**
- **[Tighten Co.](https://tighten.co)**
- **[Kirschbaum Development Group](https://kirschbaumdevelopment.com)**
- **[Cubet Techno Labs](https://cubettech.com)**
- **[British Software Development](https://www.britishsoftware.co)**
- **[Webdock, Fast VPS Hosting](https://www.webdock.io/en)**
- [UserInsights](https://userinsights.com)
- [Fragrantica](https://www.fragrantica.com)
- [SOFTonSOFA](https://softonsofa.com/)
- [User10](https://user10.com)
- [Soumettre.fr](https://soumettre.fr/)
- [CodeBrisk](https://codebrisk.com)
- [1Forge](https://1forge.com)
- [TECPRESSO](https://tecpresso.co.jp/)
- [Runtime Converter](http://runtimeconverter.com/)
- [WebL'Agence](https://weblagence.com/)
- [Invoice Ninja](https://www.invoiceninja.com)
- [iMi digital](https://www.imi-digital.de/)
- [Earthlink](https://www.earthlink.ro/)
- [Steadfast Collective](https://steadfastcollective.com/)

## Contributing

Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the [Laravel documentation](https://laravel.com/docs/contributions).

## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via [taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
