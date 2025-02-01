# HackTheBox - Noxious

## Información General

- **Nombre del desafío:** Noxious (Sherlock)
- **Categoría:** Análisis de tráfico (SOC)
- **Dificultad:** Muy Fácil
- **Fecha:** 01/02/2025

## Descripción del Desafío

El IDS nos alertó sobre la posible presencia de un dispositivo no autorizado en la red interna del Active Directory. El Sistema de Detección de Intrusos también indicó señales de tráfico LLMNR, lo cual es inusual. Se sospecha que ocurrió un ataque de envenenamiento de LLMNR.

El tráfico LLMNR estaba dirigido a **Forela-WKstn002**, que tiene la dirección IP **172.17.79.136**. Se te ha proporcionado una captura de paquetes limitada correspondiente al período en que ocurrió el evento.

Dado que esto ocurrió en la VLAN del Active Directory, se recomienda realizar una búsqueda de amenazas en la red teniendo en cuenta los vectores de ataque contra Active Directory, enfocándose específicamente en el envenenamiento de LLMNR.

## Solución

### Task 1

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(1).png)


LLMNR (Link-Local Multicast Name Resolution) es un protocolo de resolución de nombres en redes locales que permite a los dispositivos resolver nombres de host cuando no hay un DNS disponible. Es comúnmente explotado en ataques de envenenamiento de red para capturar credenciales.

Para analizar el tráfico LLMNR en el archivo `.pcap` proporcionado, seguimos los siguientes pasos:

1. **Abrir el archivo** `.pcap` en Wireshark.
2. **Filtrar el tráfico LLMNR**.
3. **Buscar las solicitudes y respuestas**:
    - Normalmente, los paquetes de solicitud (`Standard query`) se envían a la dirección multicast `224.0.0.252` en IPv4 o `FF02::1:3` en IPv6.
    - Las respuestas (`Standard query response`) contienen direcciones IP asociadas a los nombres solicitados.
4. **Encontrar la IP de la máquina objetivo**:
    - Observamos las respuestas LLMNR. La máquina que responde es el objetivo.
    - También revisamos las solicitudes para ver qué máquina está intentando resolver nombres.

(Si no apareciese nada filtrando por "llmnr", puede utilizarse el filtro `udp.port == 5355`, ya que LLMNR emplea ese puerto.)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(2).png)

### Task 2

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(3).png)

Cuando una máquina se conecta a la red y solicita una dirección IP mediante **DHCP**, suele enviar un **DHCP Request** o **DHCP Discover**. En estos mensajes, muchas veces incluye su **hostname** en un campo llamado **Option 12 (Host Name)**.

Así pues, en esta ocasión filtramos por la IP de la máquina atacante y el protocolo DHCP. Obtenemos varios paquetes, así que nos centraremos en uno de ellos, el **DHCP Request**.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(4).png)

### Task 3

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(5).png)

Filtramos por `smb2` para ver intentos de autenticación en SMB, y a esta búsqueda le añadimos el filtro `ntlmssp` para ver solo los paquetes de autenticación NTLM dentro del tráfico SMB.

Buscamos el paquete `NTLMSSP_AUTH` para extraer el nombre de usuario.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(6).png)

### Task 4

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(7).png)

Para visualizar la llegada del paquete podemos ir a "Visualización > Formato de visualización de fecha y hora" y seleccionar la opción "Fecha y hora de día UTC", o bien abrimos el menú `Frame` del paquete y nos dirigimos al apartado "UTC Arrival Time".

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(8).png)

### Task 5

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(9).png)

### Task 6

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(10).png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(11).png)

### Task 7

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(12).png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(13).png)

### Task 8

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(14).png)

Necesitamos rellenar el siguiente formato para poder crackear la contraseña del usuario:

`User::Domain:ServerChallenge:NTProofStr:NTLMv2Response(sin los 16 primeros bytes)`

Ya tenemos la mayoría, solo nos falta el último campo. Nos dirigimos a Wireshark y extraemos el valor del "NTLMv2 Response", el cual podremos encontrar en esta ruta.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(15).png)

Obtenemos la siguiente cadena:

`c0cc803a6d9fb5a9082253a04dbd4cd4010100000000000080e4d59406c6da01cc3dcfc0de9b5f2600000000020008004e0042004600590001001e00570049004e002d00360036004100530035004c003100470052005700540004003400570049004e002d00360036004100530035004c00310047005200570054002e004e004200460059002e004c004f00430041004c00030014004e004200460059002e004c004f00430041004c00050014004e004200460059002e004c004f00430041004c000700080080e4d59406c6da0106000400020000000800300030000000000000000000000000200000eb2ecbc5200a40b89ad5831abf821f4f20a2c7f352283a35600377e1f294f1c90a001000000000000000000000000000000000000900140063006900660073002f00440043004300300031000000000000000000`

Los 16 primeros bytes (32 caracteres) corresponden con el NTProofStr que hallamos anteriormente, por lo que los eliminamos, quedándonos el siguiente valor:

`010100000000000080e4d59406c6da01cc3dcfc0de9b5f2600000000020008004e0042004600590001001e00570049004e002d00360036004100530035004c003100470052005700540004003400570049004e002d00360036004100530035004c00310047005200570054002e004e004200460059002e004c004f00430041004c00030014004e004200460059002e004c004f00430041004c00050014004e004200460059002e004c004f00430041004c000700080080e4d59406c6da0106000400020000000800300030000000000000000000000000200000eb2ecbc5200a40b89ad5831abf821f4f20a2c7f352283a35600377e1f294f1c90a001000000000000000000000000000000000000900140063006900660073002f00440043004300300031000000000000000000`

Ahora colocamos los datos según el formato mencionado anteriormente:

`john.deacon::FORELA:601019d191f054f1:c0cc803a6d9fb5a9082253a04dbd4cd4:010100000000000080e4d59406c6da01cc3dcfc0de9b5f2600000000020008004e0042004600590001001e00570049004e002d00360036004100530035004c003100470052005700540004003400570049004e002d00360036004100530035004c00310047005200570054002e004e004200460059002e004c004f00430041004c00030014004e004200460059002e004c004f00430041004c00050014004e004200460059002e004c004f00430041004c000700080080e4d59406c6da0106000400020000000800300030000000000000000000000000200000eb2ecbc5200a40b89ad5831abf821f4f20a2c7f352283a35600377e1f294f1c90a001000000000000000000000000000000000000900140063006900660073002f00440043004300300031000000000000000000`

Guardamos la cadena en un fichero y utilizamos la herramienta `JohnTheRipper` para crackearla junto con el diccionario `rockyou.txt`.

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(16).png)


### Task 9

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(17).png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Noxious/assets/image(18).png)

## Lecciones Aprendidas

- **El envenenamiento de LLMNR es un ataque efectivo en redes mal configuradas.** LLMNR permite a los atacantes interceptar consultas fallidas y engañar a las víctimas para que revelen sus hashes NTLMv2. La mejor defensa es **deshabilitar LLMNR** en la red y depender de un DNS seguro.
- **Los errores ortográficos en nombres de hosts pueden filtrar credenciales.** Si un usuario escribe mal el nombre de un recurso compartido (ej. `DCC01` en lugar de `DC01`), Windows intentará resolverlo con LLMNR. Si un atacante responde primero, la víctima intentará autenticarse contra el servidor falso, enviando su hash NTLM.
- **Los hashes NTLMv2 pueden ser crackeables.** Si el atacante captura hashes NTLMv2, puede intentar crackearlos con herramientas como **hashcat** o **John the Ripper**. El uso de contraseñas seguras y la autenticación Kerberos en lugar de NTLM mitigan este riesgo.
- **Wireshark es clave para analizar ataques de red.** 
- **El tráfico SMB es un vector común de ataque en entornos Active Directory.** SMB es un objetivo frecuente para ataques como relay NTLM, Pass-the-Hash y Kerberoasting. Se recomienda reforzar la seguridad con SMB Signing y restringir accesos innecesarios.

## Recursos

- [Herramienta Wireshark](https://www.wireshark.org/)
- [Herramienta JohnTheRipper](https://github.com/openwall/john)
- [Análisis básico de ficheros PCAP](https://fwhibbit.es/analisis-basico-de-ficheros-pcap)
- [Procedimientos recomendados para proteger Active Directory](https://learn.microsoft.com/es-es/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory)