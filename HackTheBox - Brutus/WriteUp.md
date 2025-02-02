# HackTheBox - Brutus

## Información General

- **Nombre del desafío:** Brutus (Sherlock)
- **Categoría:** Análisis de logs (DFIR)
- **Dificultad:** Muy Fácil
- **Fecha:** 31/01/2025

## Descripción del Desafío

En este desafío de Sherlock, te familiarizarás con los registros **auth.log** y **wtmp** de Unix. Exploraremos un escenario en el que un servidor **Confluence** fue víctima de un ataque de fuerza bruta a través de su servicio **SSH**.

Después de obtener acceso al servidor, el atacante realizó actividades adicionales, las cuales podemos rastrear usando **auth.log**. Aunque **auth.log** se usa principalmente para el análisis de ataques de fuerza bruta, profundizaremos en todo el potencial de este artefacto en nuestra investigación, incluyendo aspectos de **escalada de privilegios, persistencia e incluso visibilidad en la ejecución de comandos**.

## Solución

### Task 1

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%201.png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%201%20(1).png)

### Task 2

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%202.png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%202%20(1).png)

### Task 3

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%203.png)

Ejecutamos el comando:

```bash
utmpdump wtmp
```

Resultado:

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%203%20(1).png)

### Task 4

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%204.png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%204%20(1).png)

### Task 5

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%205.png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%205%20(1).png)

### Task 6

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%206.png)

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%206%20(1).png)

### Task 7

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%207.png)

Ejecutamos el comando:

```bash
grep "session 37" auth.log -A10
```

Este comando nos devolverá las siguientes 10 líneas de cada coincidencia que encuentre con la cadena "session 37".

Resultados:

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%207%20(1).png)

### Task 8

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%208.png)

Ejecutamos el comando:

```bash
grep "curl" auth.log
```

![image](https://github.com/eliferrob/CTFs/blob/main/HackTheBox%20-%20Brutus/assets/Task%208%20(1).png)

## Lecciones Aprendidas

- **Los ataques de fuerza bruta pueden explotar credenciales débiles.** Si los servicios permiten intentos ilimitados de autenticación, un atacante puede probar listas de contraseñas hasta encontrar la correcta. Implementar protección contra fuerza bruta, como **bloqueo de cuentas tras múltiples intentos fallidos** y uso de **CAPTCHAs**, puede ser una buena defensa contra este tipo de ataques.
- Conseguir credenciales de un usuario con bajos privilegios puede ser suficiente para escalar dentro de la red.
- En **`/var/log/auth.log`**, un gran número de intentos fallidos seguidos de diferentes nombres de usuario puede indicar un ataque de fuerza bruta en SSH. Será necesario configurar alertas para detectar patrones sospechosos y bloquear IPs maliciosas con **Fail2Ban** o reglas de firewall.

## Recursos

- [Documentación utmpdump](https://man7.org/linux/man-pages/man1/utmpdump.1.html)
- [Cómo leer/visualizar archivos utmp, wtmp y btmp en Linux](https://toquecanela.blogspot.com/2015/05/como-leervisualizar-archivos-utmp-wtmp.html)
