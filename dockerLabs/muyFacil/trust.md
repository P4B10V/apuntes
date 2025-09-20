# Máquina: Trust

- **Dificultad** Muy fácil

--- 

Empezamos con un escaneo simple, que nos indica que los puertos 22/TCP y 80/TCP están abiertos, usaremos -sV para obtener más información de esos servicios.

```
nmap 172.18.0.2 -sV
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
MAC Address: 02:42:AC:12:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Visitamos 172.18.0.2 para ver la página web y es la típica página por defecto de Apache. Como no sabemos nada mas intentaremos descubrir si hay directorios a los que 
podamos acceder utilizando herramientas para ello:

```
gobuster dir -u 172.18.0.2 -w /usr/share/wordlists/wfuzz/webservices/ws-files.txt -x .php,.py,.html,.sh
```

dir -> Para que busque directorios
-u -> el objetivo 
-w -> la lista de palabras que intentará probar
-x -> la extension que probará con esos nombres 

Tuvimos una respuesta inmediata de:
```
/secret.php           (Status: 200) [Size: 927]
```













