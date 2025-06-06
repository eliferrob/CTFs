# CTF - The Hackers Labs - Cyberpunk

## 📑 Información General

- **Nombre:** Cyberpunk
- **Dificultad:** Fácil
- **SO:** #Linux
- **Vulnerabilidad principal:** Inicio de sesión FTP anónimo
- **Fecha resolución:** 06/06/2025
- **MD5:** 5966f4a2369056531380b61f27aa73c7
- **Enlace:** https://thehackerslabs.com/cyberpunk/

>Objetivo > Obtener las flags: 
> - 🚩 user.txt 
> - 🚩 root.txt

## 🔎 Reconocimiento

### Escaneo Inicial

IP máquina víctima: **192.168.10.21**

Comprobamos la conectividad enviando un paquete `icmp` a dicha máquina. Además, en función del TTL sabremos distinguir el SO instalado en la misma. Dado que su TTL es 64, estamos frente a una máquina **Linux**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(2).png)

### Escaneo de Puertos

A continuación, proseguimos con un escaneo de puertos para tratar de averiguar qué servicios están activos y accesibles. Se detectan abiertos los puertos **21**, **22** y **80**. Además, el propio escaneo nos revela **3 archivos** en el directorio compartido a través de FTP. El tipo de archivos que visualizamos nos indica que probablemente el directorio aloje el servicio web, arrojando algunas pistas sobre cómo podemos proceder en la explotación. Sin embargo, primero visitaremos el sitio web.

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.10.21
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(3).png)

### Enumeración de Servicios

| Puerto | Servicio | Versión       |         Notas         |
| ------ | -------- | ------------- |:---------------------:|
| 21     | FTP      | -             | Permite login anónimo |
| 22     | SSH      | OpenSSH 9.2p1 |           -           |
| 80     | HTTP     | Apache 2.4.59 |     Servidor web      |


## 🌐 Exploración

### Reconocimiento Web

Visitamos el sitio web a través del explorador, pero no encontramos ninguna información relevante, tan solo una imagen relacionada con la temática del CTF.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(4).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(5).png)

No obstante, recordamos que durante el escaneo de puertos encontramos un fichero `secret.txt` alojado en el servidor FTP, al que podemos acceder desde la url del sitio.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(6).png)
## ⚔️ Explotación

### Acceso a la máquina víctima

Dado que el contenido del directorio compartido por FTP es el mismo que aloja la página web, creamos un **payload reverse shell** en php que posteriormente cargaremos en la ruta y ejecutaremos mediante el navegador.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(7).png)

Iniciamos sesión de forma anónima en el servicio FTP.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(8).png)

Configuramos el modo escucha de la máquina atacante mediante la herramienta **Netcat**. Estaremos esperando respuesta por el mismo puerto que configuramos en el payload, en este caso el 443.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(9).png)

Abrimos el fichero desde la web, lo cual ejecutará el payload y nos proporcionará acceso al sistema.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(10).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(11).png)

Como las sesiones abiertas con `msfvenom` son muy inestables y no duran más que unos pocos minutos, nos dirigimos a www.revshells.com, creamos un payload con la IP de la máquina atacante y un puerto en el que nos pondremos a la escucha. Ejecutamos el siguiente comando en la shell que ya tenemos abierta y migramos la sesión a un entorno más estable para continuar con la explotación.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(12).png)

### Tratamiento TTY

Ahora realizaremos el tratamiento de la TTY para que sea más manejable y normalizar así la conexión. Ejecutamos el siguiente código:

```bash
script /dev/null -c bash
```

Pulsamos `Ctrl+Z` para mandar la sesión a segundo plano. Acto seguido, escribimos en la terminal lo siguiente:

```bash
stty raw -echo; fg
reset xterm
```

Es posible que al ejecutar la primera línea no nos aparezca como que estamos escribiendo nada. Igualmente escribimos `reset xterm` y presionamos enter. Se nos abrirá nuevamente la TTY pero con un prompt más legible.

Sin embargo, todavía habrá comandos que no podremos ejecutar, así que tendremos que exportar las siguientes variables:

```bash
export TERM=xterm
export SHELL=bash
```

## 🔐Escalada de Privilegios

Analizamos el fichero `/etc/passwd` para determinar los usuarios con los que se puede hacer login. Encontramos dos, **arasaka** y **root**.

```bash
cat /etc/passwd | grep /bin/bash
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(13).png)

Como hemos conseguido acceder a la máquina con el usuario "www-data", apenas tenemos privilegios. Ahora que ya hemos confirmado el nombre de un usuario, seguimos explorando algunos directorios a ver si encontramos algo más sobre el usuario. Para realizar una búsqueda rápida, ejecutamos el siguiente comando y localizamos el fichero **arasaka.txt** en `/opt` que contiene un código encriptado.

```bash
find / -name *arasaka* 2>/dev/null
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(14).png)

Este tipo de encriptación se llama **Brainfuck**.

Brainfuck es un lenguaje de programación creado por Urban Müller en 1993. Es famoso por ser extremadamente minimalista: tiene solo 8 comandos (cada uno opera sobre una cinta de celdas de memoria, como una máquina de Turing) y una sintaxis deliberadamente confusa. Su nombre hace referencia a que, efectivamente, traducir esto conlleva serios dolores de cabeza.

Copiamos el código en algún decodificador de internet y nos devuelve lo que parece ser la contraseña del usuario **arasaka**, **cyberpunk2077**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(15).png)

Iniciamos sesión en la máquina víctima a través del **puerto 22 (SSH)** y comprobamos los permisos con que contamos mediante `sudo -l`. Encontramos que podemos ejecutar `python3.11` sobre el fichero `randombase64.py` con **sudo**.

Localizamos primero la flag del usuario y proseguimos con la escala de privilegios.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(16).png)

🚩 **Flag de usuario encontrada.**

Revisando el sitio web https://gtfobins.github.io/, encuentro que el binario `python` podría explotarse ejecutando el siguiente comando.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(17).png)

 Dado que en este caso no tenemos libertad absoluta para ejecutar `python`, sino que solo podemos hacerlo como `root` sobre un fichero en concreto, vamos a analizarlo a fin de hallar una vulnerabilidad que podamos explotar.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(18).png)

El fichero importa una biblioteca con una **ruta relativa**, así que aprovecharemos esta vulnerabilidad creando otro archivo llamado **base64.py** con un payload similar al que nos indica GTFOBINS que nos permita escalar privilegios.

El contenido del fichero `base64.py` será el siguiente:

```bash
import os;
os.system("chmod u+s /bin/bash")
```

Al aplicarle el permiso *u+s* estaremos activando el bit **SUID** en el binario `/bin/bash`. Si luego lo ejecutamos con la opción **-p**, obtendremos una shell con privilegios de root.

A continuación, le damos permisos de ejecución a `base64.py` y ejecutamos el siguiente código. Es importante emplear las **rutas absolutas** tanto del binario como del script, ya que si no no funcionará.

```bash
sudo -u root /usr/bin/python3.11 /home/arasaka/randombase64.py
```

Ignoramos el error que devuelve el script y revisamos los permisos del binario `bash` y lo ejecutamos tal y como se aclaró anteriomente.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(19).png)

Perfecto, ya hemos escalado privilegios. Ahora nos dirigimos a la ruta `/root` y buscamos la flag.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Cyberpunk%20(20).png)

🚩 **Flag de root encontrada.**
