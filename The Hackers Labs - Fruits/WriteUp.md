# CTF - The Hackers Lab - Fruit

# 📑 Información General

- **Nombre:** Fruits
- **Dificultad:** Muy Fácil
- **SO:** Linux
- **Fecha resolución:** 05/06/2025
- **MD5:** 128b77040d5ec9365a158ac0c1bb5021
- **Enlace:** [The Hackers Lab - Fruits](https://thehackerslabs.com/fruits/)

>[!tip] Objetivo
>Obtener las flags: 
> - 🚩 user.txt 
> - 🚩 root.txt

# 🔎 Reconocimiento

## Escaneo de Puertos

### TCP

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.10.24
```

Encontramos dos puertos abiertos, el **22** y el **80**.

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(1).png)

### UDP

```bash
nmap -sU --top-ports 200 --min-rate 5000 -n -Pn 192.168.10.24
```

No hay ningún puerto UPD abierto.

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(2).png)

## Enumeración de Servicios

| Puerto | Servicio | Versión       |    Notas     |
| ------ | -------- | ------------- |:------------:|
| 22     | SSH      | OpenSSH 9.2p1 |      -       |
| 80     | HTTP     | Apache 2.4.57 | Servidor web |

# 🌐 Exploración

Utilizamos el navegador para visualizar el sitio web.

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(3).png)

Probamos a introducir cualquier palabra, pero nos devuelve un error, así que pasamos a hacer fuzzing para intentar descubrir directorios o ficheros interesantes.

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(4).png)


Encontramos un fichero potencialmente aprovechable, `/fruits.php`. Al abrirlo en el navegador, observamos que devuelve una pantalla en blanco. Analizo por encima la petición con Burp Suite y descubro que utiliza el método GET, lo que me hace sospechar que quizás se pueda explotar una vulnerabilidad LFI (Local File Inclusion. Para confirmar esta hipótesis, decido utilizar la herramienta **wfuzz** para hacer fuzzing y descubrir un posible parámetro que me permita acceder al fichero `/etc/passwd`.

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(5).png)

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(6).png)

Descubrimos que el parámetro `file` devuelve una respuesta más larga que el resto, así que decidimos investigarlo. Efectivamente, hemos podido explotar la vulnerabilidad y acceder a la ruta especificada.

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(7).png)

Encontramos el usuario **bananaman**.

# ⚔️ Explotación

Realizamos un ataque de fuerza bruta contra el puerto ssh utilizando el usuario previamente descubierto.

```bash
hydra -l bananaman -P /usr/share/wordlists/rockyou.txt ssh://192.168.10.24
```

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(8).png)

Obtenemos las credenciales de acceso de "bananaman" en la máquina víctima. Accedemos y, al ejecutar el comando `ls` para listar el contenido del directorio en el que nos encontramos, hallamos la **flag del usuario**.

![[dwqdeqewdqwa.png]]

🚩 **Flag de usuario encontrada.**

# 🔐Escalada de Privilegios

Ejecutamos el comando `sudo -l` dentro de la sesión que acabamos de abrir en la máquina víctima con el usuario "bananaman" para comprobar sus permisos. Vemos que podemos ejecutar el comando `find` como administrador.

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(9).png)

Nos dirigimos a la página web https://gtfobins.github.io/ y buscamos cómo podemos explotar esta vulnerabilidad. 

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(10).png)

Ejecutamos el código que nos muestra y habremos conseguido **escalar privilegios**.  Ahora solo nos falta buscar la flag en el directorio `/root`.

![image](https://github.com/eliferrob/CTFs/blob/main/The%20Hackers%20Labs%20-%20Fruits/assets/fruits%20(11).png)

🚩 **Flag de root encontrada.**
