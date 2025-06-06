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

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(1).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(2).png)

### Task 2

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(3).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(4).png)

### Task 3

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(5).png)

Ejecutamos el comando:

```bash
utmpdump wtmp
```

Resultado:

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(6).png)

### Task 4

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(7).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(8).png)

### Task 5

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(9).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(10).png)

### Task 6

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(11).png)

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(12).png)

### Task 7

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(13).png)

Ejecutamos el comando:

```bash
grep "session 37" auth.log -A10
```

Este comando nos devolverá las siguientes 10 líneas de cada coincidencia que encuentre con la cadena "session 37".

Resultados:

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(14).png)

### Task 8

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(15).png)

Ejecutamos el comando:

```bash
grep "curl" auth.log
```

![image](https://github.com/eliferrob/CTFs/blob/main/assets/Brutus%20(16).png)
