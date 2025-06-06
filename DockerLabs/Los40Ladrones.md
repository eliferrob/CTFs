# DockerLabs - Los40Ladrones

## Información General

- **Nombre del desafío:** Los40Ladrones
- **Categoría:** Boot2Root
- **Dificultad:** Muy Fácil
- **Fecha:** 21/01/2025
- **Plataforma:** https://dockerlabs.es/

## Descripción del Desafío

El desafío "Los 40 Ladrones" de DockerLabs es una máquina diseñada para evaluar habilidades en ciberseguridad, enfocándose en técnicas como escaneo de puertos, fuerza bruta y escalada de privilegios.

## Solución

### 1. Despliegue de la máquina

En primer lugar, ejecutaremos el script proporcionado junto con la máquina para desplegar el laboratorio. Nos aparecerá un mensaje informándonos de la IP utilizada por la máquina vulnerable, que es la `172.17.0.2`.

```bash
sudo bash auto_deploy.sh los40ladrones.tar
```


### 2. Análisis de vulnerabilidades

Emplearemos la herramienta Zenmap para ejecutar de forma gráfica un escaneo de `nmap`. El resultado revela que la máquina objetivo tiene el puerto 80 abierto con un servicio Apache activo.

```bash
nmap -T4 -A -v 172.17.0.2
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Los40Ladrones%20(1).png)

Si introducimos la IP en el navegador junto con el puerto, podremos comprobar que, efectivamente, el servicio está activo.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Los40Ladrones%20(2).png)

### 3. Análisis del sitio web

A continuación, realizaremos un escaneo con *Gobuster* utilizando un diccionario de palabras de tamaño mediano. 

El comando especificado a continuación accede a una lista de directorios y archivos en el servidor web con IP `172.17.0.2`, probando una serie de nombres de directorios y archivos tomados de la lista proporcionada (`directory-list-2.3-medium.txt`). Además, para cada nombre de directorio o archivo encontrado, probará las extensiones `.php`, `.html` y `.txt` para ver si existen en el servidor.

Este tipo de ataque es útil para descubrir archivos y directorios ocultos o no indexados en un servidor web, lo cual podría revelar recursos importantes o vulnerabilidades.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/directory-list-2.3-medium.txt -x php,html,txt
```

- `gobuster`: Es la herramienta utilizada para hacer un ataque de enumeración de directorios y archivos en servidores web.
- `dir`: Indica que el modo de operación es la búsqueda de directorios.
- `-u`: Especifica la URL del objetivo al que se está atacando.
- `-w`: Define el archivo de lista de palabras que se usará para probar los nombres de directorios y archivos.
- `-x`: Establece las extensiones de archivo que se deben probar junto con los nombres de los directorios.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Los40Ladrones%20(3).png)

Encontramos un fichero de texto con un nombre un tanto peculiar, así que decidimos acceder a él alterando la url. 

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Los40Ladrones%20(4).png)

En este archivo encontramos un mensaje propone la existencia de un posible usuario en sistema llamado "**toctoc**", así como una secuencia de números que se asemeja a una sucesión de puertos (**7000 8000 9000**). Esta serie de números apunta que la máquina utiliza la técnica *port knocking* para ocultar un puerto. 

### 4. Port Knocking

"Port knocking" es una técnica de seguridad que implica enviar paquetes a una secuencia predefinida de puertos cerrados para desbloquear o abrir un puerto específico, generalmente utilizado para ocultar servicios importantes. Si se sigue la secuencia correcta, el servidor configurado con "port knocking" abrirá el puerto deseado (por ejemplo, el puerto SSH).

En este caso, al enviar paquetes a los puertos 7000, 8000 y 9000 en secuencia, se espera que el servidor abra un puerto adicional, permitiendo el acceso al servicio correspondiente.

Así pues, utilizaremos el comando `knock` para enviar paquetes de red con dicha secuencia específica. La herramienta no se encuentra por defecto en Kali Linux, por lo que debemos instalarla primero.

```bash
knock 172.17.0.2 7000 8000 9000
```

- `knock`: Es la herramienta que se utiliza para enviar los paquetes de "port knocking".
- `172.17.0.2`: Es la dirección IP del objetivo al que se están enviando los paquetes.
- `7000 8000 9000`: Son los números de puerto a los que se envían los paquetes en la secuencia especificada.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Los40Ladrones%20(5).png)

Tras esto, si volvemos a realizar un escaneo comprobaremos que se ha abierto el puerto 21 (correspondiente a SSH).

### 5. Fuerza Bruta en SSH

Utilizamos la herramienta *Hydra* para intentar acceder mediante fuerza bruta con el diccionario rockyou.txt (que contiene una gran cantidad de contraseñas comunes) al servicio SSH. Tras ejecutar el comando, obtenemos una coincidencia. La contraseña es "kittycat".

```bash
hydra -l toctoc -P /usr/share/wordlists/rockyou.txt.gz ssh://172.17.0.2 -t 64
```

- `hydra`: Es la herramienta que se utiliza para realizar ataques de fuerza bruta.
- `-l`: Especifica el nombre de usuario que se utilizará en el ataque.
- `-P`: Utiliza un fichero como lista de posibles contraseñas. 
- `ssh://172.17.0.2`: Define el protocolo (SSH) y la dirección IP del objetivo (`172.17.0.2`).
- `-t`: Establece el número de hilos concurrentes en 64, lo que permite realizar el ataque de manera más rápida al probar varias combinaciones de contraseñas en paralelo.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Los40Ladrones%20(6).png)

### 6. Acceso SSH y Escalada de Privilegios

Seguidamente nos conectaremos a la máquina mediante SSH utilizando las credenciales obtenidas.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Los40Ladrones%20(7).png)

Finalmente, ejecutamos ```sudo -l``` para ver los comandos que podemos ejecutar a nivel de root, encontrándonos con dos comandos. Realizamos un sudo con uno de ellos y veremos que hemos escalado privilegios.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Los40Ladrones%20(8).png)
