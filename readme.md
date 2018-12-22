

<p align="center"><img src="https://www.regalonatural.com/img/raluca-art-logo-1537553801.jpg"></p>

### Gestión de Clientes y Proveedores 


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


### Dockerfile
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

### docker-compose.yaml

Este fichero realiza el montaje de los docker, la relación entre ellos y los levanta.

```
version: "2"
services:
  mysql:
    image: 'mysql:5.6.35'
    environment:
      MYSQL_ROOT_PASSWORD: "vmsn2004"
      MYSQL_DATABASE: docker
  php:
      image: php:7.0-fpm
      links:
        - mysql:mysqldb
      volumes:
        - ./apache:/var/www/html
  apache:
    build: ./
    links:
      - php:phpfpm
    volumes:
      - ./apache:/var/www/html
    ports:
      - "8083:80"

```

A destacar varias cosas en este fichero:

* En las secciones php y apache se mapea con el directorio local *./apache*, esto se debería cambiar para que apuntase al raiz de nuestra web. **Ojo** mapea */var/www/html* y sin embargo en el fichero dockerfile se apunta a */var/www/html/public*.

* Las versiones de mysql, php e incluso apache no son las últimas. Inicialmente lo dejamos así para que funciones y después valoramos hacerle el upgrade necesario.

* Solo se han mapeado los puertos de apache (8083:80), deberían mapearse también los de mysql para que en el mismo host puedan convivir varios proyectos (de multiples dockers) sin machacarse puertos.


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
