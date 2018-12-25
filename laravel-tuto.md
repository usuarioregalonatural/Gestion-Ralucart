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

