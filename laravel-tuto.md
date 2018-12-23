# Información manejo Laravel

### Creación de un nuevo proyecto
Para la creación de un nuevo proyecto, debemos situarnos en la ruta elegida y utilizar composer
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

#### Generación de la info de BBDD
en el directorio del proyecto, escribimos:
```bash
php artisan migrate
```

Si aparece el error:
```
PDOException::("SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes")
```
tenemos que modificar el siguiente archivo incluyendo lo siguiente:
```php

```
