# Información manejo Laravel

### Creación de un nuevo proyecto
Para la creación de un nuevo proyecto, debemos situarnos en la ruta elegida, esta debe ser la ruta padre donde se generará el nuevo poryecto y utilizar composer
```
composer create-project laravel/laravel pru01 "5.7.*"
```

Despues creamos una bbdd en mysql, por ejemplo <code>pruebas</code>

Abrimos el proyecto y en el archivo <code>.env</code> sustiuimos los valores para conectar a la bbdd
```
DB_DATABASE=prueba
DB_USERNAME=root
DB_PASSWORD=xxxxx
```
Seguidamente configuramos IntelliJ para poder ejecutar en local.
![image info](./tutos/img/Settings-Entorno-01.jpg)
![image info](./tutos/img/Settings-Entorno-02.jpg)
![image info](./tutos/img/Settings-Entorno-03.jpg)

### Generación de la info de BBDD
Para asegurarno que no aparece el error:
```
PDOException::("SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes")
```
tenemos que modificar el siguiente archivo incluyendo lo siguiente:
<code>app/providers/AppServiceProvider.php</code>

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema; <--añadir esta linea

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191); <--añadir esta linea
    }

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

después en el directorio del proyecto, escribimos:
```bash
php artisan migrate
```

### Generar las opciones de autentificación
Para ello en el directorio del proyecto escribiremos:
```
php artisan make:auth
```

## Subir a Github el proyecto
En el directorio local, inicializaremos GIT
```
H:\SERVIDOR-LOCAL\Ampps\www\Proyectos\gestionv3>git init
Initialized empty Git repository in H:/SERVIDOR-LOCAL/Ampps/www/Proyectos/gestionv3/.git/

H:\SERVIDOR-LOCAL\Ampps\www\Proyectos\gestionv3>
```
Después crearemos un nuevo repositorio en github (gestionv3)

y en el directorio local lo subiremos:
```
git remote add origin https://github.com/usuarioregalonatural/gestionv3.git
git add *
git commit -m "Commit Inicial"
git push -u origin master
```
## Sincronización en el Server
Nos movemos al servidor, y en el directorio padre donde vayamos a crearlo, lo clonamos:
```
[root@vicsoft Gestion]# git clone https://github.com/usuarioregalonatural/gestionv3.git
```
Esto nos creará la carpeta con el proyecto.

Acto seguido creamos en link simbólico en la carpeta de acceso web del servidor hacia donde tenemos nuestro proyecto:
```bash
[root@vicsoft html]# pwd
/var/www/html
[root@vicsoft html]# mv gestion 20181225-gestion
[root@vicsoft html]# ln -s /home/webs/Gestion/gestionv3/public gestion
```
Después le asignamos permisos y propietarios a la carpeta:
 ```bash
[root@vicsoft gestionv3]# chown -R www-data:apache *
[root@vicsoft gestionv3]# chmod -R 777 *
```

y por último creamos y configuramos el fichero .env en la raiz de nuestro proyecto:

```bash
[root@vicsoft gestionv3]# cp .env.example .env
```
luego editamos el fichero .env y lo dejamos como sigue:
 ```bash
 APP_NAME=Gestion
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=gestionv3
DB_USERNAME=root
DB_PASSWORD={mi_password}
 ```

