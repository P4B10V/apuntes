# Máquina: Vacations  
**Dificultad:** Muy fácil  

---

La IP del contenedor es: `172.17.0.2`. Procedemos a hacer un `nmap` para ver si tiene algún servicio expuesto.  
Se detectan los siguientes servicios:  
- SSH en el puerto 22  
- HTTP en el puerto 80  

Antes de nada, visitamos la página web.  

La página aparece en blanco. Utilicé `gobuster` para buscar más directorios, pero no encontré nada relevante.  
Al inspeccionar el código fuente con "View page source", encontramos el siguiente mensaje:  

`<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->`

Con esto, ya tenemos dos posibles usuarios para probar en SSH. Como el correo es para Camilo, intentaremos con ese usuario.

Ejecutamos el siguiente comando de fuerza bruta con `hydra`:  

`hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`

Resultado:  

`[22][ssh] host: 172.17.0.2   login: camilo   password: password1`

---

Antes de nada, se probaron los comandos `sudo -l` y `env` para revisar si había alguna forma de escalar privilegios, pero no se encontró nada.

Con las credenciales obtenidas, nos conectamos por SSH y buscamos el correo que Juan le había dejado a Camilo.  
Nos posicionamos en la carpeta `/var/mail` y encontramos una carpeta llamada `camilo` que contiene un archivo `correo.txt` con el siguiente mensaje:  

Hola Camilo,  
Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb

---

Ahora, con el usuario `juan`, probamos a ejecutar `sudo -l` para comprobar si podemos ejecutar algún comando con privilegios:  

`User juan may run the following commands on 0582f92de872:  
    (ALL) NOPASSWD: /usr/bin/ruby`

Como no tengo experiencia con Ruby, busqué información en https://gtfobins.github.io/, donde encontré cómo escalar privilegios usando Ruby:  

`sudo ruby -e 'exec "/bin/sh"'`

Ejecutamos este comando y obtenemos acceso como root.










