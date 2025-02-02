# HackTheBox - Reaper

## Información General

- **Nombre del desafío:** Reaper (Sherlock)
- **Categoría:** Análisis de tráfico (DFIR)
- **Dificultad:** Muy Fácil
- **Fecha:** 02/02/2025

## Descripción del Desafío

Nuestro SIEM nos alertó sobre un evento de inicio de sesión sospechoso que debe analizarse de inmediato. Los detalles de la alerta indicaron que la dirección IP y el nombre de la estación de trabajo de origen no coincidían. Se le proporciona una captura de red y registros de eventos del período de tiempo cercano al incidente. Relacione la evidencia proporcionada e informe a su gerente del SOC.

## Solución

### Task 1

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(1).png)

1. Abrimos el fichero `ntlmrelay.pcapng` en Wireshark. 
2. Filtramos por tráfico NBNS (nbns) para ver solicitudes de resolución de nombres.
3. Nos centramos en la primera consulta, donde observamos una petición "Refresh" de `Forela-Wkstn001`. "Refresh NB" indica que una máquina está renovando su nombre en la red, asegurando que otros dispositivos sepan que sigue activa. Por tanto, la dirección IP en la columna "Source" es la de Forela-Wkstn001, ya que es el dispositivo que está enviando este anuncio para mantenerse registrado en la red.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(2).png)

### Task 2

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(3).png)

Seguimos el mismo proceso de la tarea 1, solo que en esta ocasión buscamos el paquete enviado por `Forela-Wkstn002`.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(4).png)

### Task 3

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(5).png)

1. Filtramos por tráfico `ntlmssp` para ver solo los paquetes de autenticación NTLM.
2. Buscamos el paquete `NTLMSSP_AUTH` para extraer el nombre de usuario.
3. Analizamos el contenido y nos dirigimos al apartado "SMB2" > "SMB2 Header" > "Session ID".

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(6).png)

### Task 4

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(7).png)

Filtramos por `llmnr` y buscamos la IP que está respondiendo a las solicitudes de resolución de nombres: esta es la del atacante.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(8).png)

### Task 5

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(9).png)

1. Filtramos por el protocolo `SMB`.
2. Buscamos las solicitudes de acceso a recursos compartidos (Tree Connect Request).
3. La ruta podemos visualizarla desde la info del paquete, o bien accediendo a "SMB2" > "Tree Connect Request" > "Tree".

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(10).png)

### Task 6

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(11).png)

1. Filtramos en Wireshark por `ntlmssp`.
2. Buscamo la autenticación NTLM del atacante.
3. El puerto de origen se encuentra en el campo `Source Port` de los paquetes TCP asociados.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(12).png)

### Task 7

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(13).png)

1. Abrimos el ficheor `Security.evtx` con el Visor de Eventos de Windows.
2. Filtramos por el "Event ID 4624" (evento de inicio de sesión).
3. Buscamos sesiones donde el usuario comprometido (arthur.kyle) inicia sesión.
4. El `Logon ID` está en los detalles del evento.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(14).png)

### Task 8

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(15).png)

En el mismo evento, encontramos la información si descendemos un poco más abajo.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(16).png)

### Task 9

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(17).png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(18).png)

### Task 10

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(19).png)

En esta ocasión, filtramos por el ID 5140 (evento de compartición de archivos) y encontramos la ruta.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Reaper/assets/image(10).png)


## Lecciones Aprendidas

- La captura de paquetes es esencial para entender cómo se lleva a cabo un ataque, especialmente si se trata de un ataque de Relay NTLM o un ataque de brute-force. Saber cómo filtrar por `ntlmssp` o `smb2` es crucial para analizar la autenticación.
- La máquina Reaper enseña cómo los ataques de relay NTLM pueden ser explotados para obtener credenciales y acceder a sistemas vulnerables.

## Recursos

- [Herramienta Wireshark](https://www.wireshark.org/)
- [Análisis básico de ficheros PCAP](https://fwhibbit.es/analisis-basico-de-ficheros-pcap)
- [Event ID codes](https://learn.microsoft.com/en-us/previous-versions/tn-archive/dd639428(v=technet.10))
