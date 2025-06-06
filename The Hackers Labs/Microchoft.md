# CTF - The Hackers Lab - Microchoft

## 📑 Información General

- **Nombre:** Microchoft
- **Dificultad:** Muy Fácil
- **SO:** #Windows
- **Vulnerabilidad principal:** EternalBlue
- **Fecha resolución:** 05/06/2025
- **MD5:** d3c63284f99639d8a19467f39ee7c57f
- **Enlace:** [The Hackers Lab - Microchoft](https://thehackerslabs.com/microchoft/)

>Objetivo > Obtener las flags: 
> - 🚩 user.txt 
> - 🚩 root.txt

## 🔎 Reconocimiento

### Escaneo Inicial
IP máquina víctima: **192.168.10.20**

Comprobamos la conectividad enviando un paquete `icmp` a dicha máquina. Además, en función del TTL sabremos distinguir el SO instalado en la misma. Dado que su TTL es 128, estamos frente a una máquina **Windows**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/microchoft%20(1).png)

### Escaneo de Puertos

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.10.20
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/microchoft%20(2).png)

### Enumeración de Servicios

| Puerto | Servicio     | Versión                        |        Notas         |
| ------ | ------------ | ------------------------------ |:--------------------:|
| 135    | msrpc        | Microsoft Windows RPC          |          -           |
| 139    | netbios-ssn  | Microsoft Windows netbios-ssn  |          -           |
| 445    | microsoft-ds | Windows 7 Home Basics 7601 SP1 | Workgroup: WORKGROUP |
| 41955  | msrpc        | Microsoft Windows RPC          |          -           |

## 🌐 Exploración

Analizando el resultado devuelto con `nmap`, detectamos que el sistema operativo es **Windows 7 Home Basic SP1 (Service Pack 1)** . Este sistema, con el puerto 445 abierto, es vulnerable al clásico exploit **EternalBlue**.

## ⚔️ Explotación

Abrimos **Metasploit** y buscamos por el término "eternalblue". Nos devolverá varios resultados, pero a nosotros nos interesa el primero, así que lo seleccionamos y lo configuramos con los siguientes comandos:

```bash
search eternalblue
use 0
show options
set RHOSTS 192.168.10.20
run
```

La explotación ha sido un éxito. Hemos conseguido abrir una sesión de **meterpreter** con el usuario "NT Authority\System".

![image](https://github.com/eliferrob/CTFs/blob/main/assets/microchoft%20(3).png)

Abrimos una shell escribiendo `shell` y comprobamos qué usuarios hay registrados en la máquina víctima.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/microchoft%20(4).png)

## 🔐Escalada de Privilegios

Nos desplazamos a la ruta `C:\Users` y listamos su contenido. Vemos que efectivamente tenemos las carpetas de los usuarios **Admin** y **Lola**, así que indagamos un poco en ellas y encontramos que en el escritorio guardan un fichero con su **flag**.

![image](https://github.com/eliferrob/CTFs/blob/main/assets/microchoft%20(5).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/microchoft%20(6).png)

🚩 **Flag de root encontrada.**

![image](https://github.com/eliferrob/CTFs/blob/main/assets/microchoft%20(7).png)

🚩 **Flag de user encontrada.**
