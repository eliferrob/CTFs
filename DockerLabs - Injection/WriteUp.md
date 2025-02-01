# DockerLabs - Injection

## Información General

- **Nombre del CTF:** Injection
- **Categoría:** Boot2Root
- **Dificultad:** Muy Fácil
- **Fecha:** 31/01/2025

## Descripción del Desafío

El desafío **Injection** plantea un entorno donde el objetivo es descubrir y explotar una vulnerabilidad en un servidor web en el que se puede introducir una SQL Injection.  

## Solución

### 1. Despliegue de la máquina

Como es de esperar, primero ejecutaremos el script proporcionado junto con la máquina para desplegar el laboratorio. Nos aparecerá un mensaje informándonos de la IP utilizada por la máquina vulnerable, que en este caso corresponde con la `192.168.100.2`.

```bash
sudo bash auto_deploy.sh injection.tar
```

### 2. Análisis de vulnerabilidades

Comenzamos con un escaneo básico del servidor para identificar servicios activos:

```bash
nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn 192.168.100.2
```

- `-p-`: Escanea todos los puertos disponibles (de 1 a 65535).
- `-sS`: Realiza un escaneo SYN (también conocido como escaneo "sigiloso" o "half-open"), que es más rápido y menos detectable en comparación con un escaneo completo de conexiones TCP.
- `-sC`: Ejecuta scripts predeterminados de Nmap (Nmap Scripting Engine) que proporcionan información adicional sobre los servicios detectados, como versiones y posibles vulnerabilidades.
- `-sV`: Detecta la versión de los servicios que se están ejecutando en los puertos abiertos.
- `--min-rate`: Establece una tasa mínima de envío de paquetes, en este caso de 5000 paquetes por segundo, lo que acelera el escaneo.
- `-n`: Desactiva la resolución de nombres DNS, lo que evita que Nmap intente resolver las direcciones IP a nombres de dominio, lo que mejora la velocidad del escaneo.
- `-Pn`: Desactiva el ping previo al escaneo, lo que significa que Nmap no hará un "ping" para verificar si el host está activo antes de realizar el escaneo de puertos. Esto es útil cuando el objetivo puede estar bloqueando pings o para escanear máquinas sin respuesta a los pings.

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20Injection/assets/Injection%20(1).png)

**Resultados del escaneo:**

- Puerto 80 abierto: Servidor HTTP.
- Puerto 22 abierto: SSH.

### 3. Explotación

Accedemos al servidor HTTP mediante el navegador escribiendo `http://192.168.100.2:80` y nos aparecerá una ventana de un login. Introducimos una inyección SQL básica en el campo del usuario, y como el carácter `#` comenta todo lo que se sitúe a la derecha del mismo, no tendrá en cuenta el resto de la query. Escribimos cualquier cosa como contraseña (porque es un campo requerido) y presionamos la tecla Enter.

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20Injection/assets/Injection%20(2).png)

Una vez logueados, nos devuelven un mensaje de bienvenida con una contraseña y el nombre de un usuario.

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20Injection/assets/Injection%20(3).png)

 Dado que también se encuentra abierto el puerto de SSH, probamos a introducir las credenciales que acabamos de obtener.

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20Injection/assets/Injection%20(4).png)

### 4. Escalada de Privilegios

A continuación, ejecutaremos el siguiente comando para realizar una búsqueda desde la raíz de todos los ficheros con el bit **SetUID** (SUID) activado para el usuario `root`. Este bit es un permiso especial de sistemas Linux que permite que un archivo ejecutable se ejecute con los permisos de su propietario, en lugar de los del usuario que lo ejecuta. 

```bash
find / -perm -4000 -user root 2>/dev/null
```

**Resultados de la búsqueda:**

- /usr/lib/dbus-1.0/dbus-daemon-launch-helper
- /usr/lib/openssh/ssh-keysign
- /usr/bin/chsh
- /usr/bin/su
- /usr/bin/mount
- /usr/bin/umount
- /usr/bin/passwd
- /usr/bin/gpasswd
- /usr/bin/chfn
- /usr/bin/env
- /usr/bin/newgrp

Hay uno de ellos que nos llama especialmente la atención, y es el fichero `/usr/bin/env`, un binario que permite la ejecución de un comando en un entorno modificado. Aprovechamos esta vulnerabilidad para abrir una shell a nombre de `root`.

```bash
/usr/bin/env /bin/sh -p
```

- `/usr/bin/env`: Ejecuta el binario env.
- `/bin/sh -p`: Ejecuta la shell (sh) con la opción -p.

Normalmente, cuando ejecutas una shell con `sudo` o con `setuid`, ciertos privilegios pueden reducirse por seguridad. Con `sh -p`, esos privilegios NO se reducen, lo que puede permitirte escalar privilegios a `root`.

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20Injection/assets/Injection%20(5).png)

¡Estamos dentro!

## Lecciones Aprendidas

- SUID permite que un programa se ejecute con los permisos de su propietario.
- Puede ser útil para programas legítimos como passwd.
- También puede ser peligroso si un atacante lo explota para escalar privilegios.

## Recursos

- [Documentación oficial de Nmap](https://nmap.org/man/es/index.html)
- [SetUID, SetGID, and Sticky Bits in Linux File Permissions](https://www.geeksforgeeks.org/setuid-setgid-and-sticky-bits-in-linux-file-permissions/)
- [Linux File Permissions: Understanding setuid, setgid, and the Sticky Bit](https://www.cbtnuggets.com/blog/technology/system-admin/linux-file-permissions-understanding-setuid-setgid-and-the-sticky-bit)
