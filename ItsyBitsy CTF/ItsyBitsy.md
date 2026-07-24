Para poder acceder a este laboratorio, debo de poner en el buscador, tras activar la vpn correspondiente y encender el entorno preparado, debo de acceder a la ip de la máquina en el buscador (con http).

El enunciado remarca una potencial conexión c2. Dichos logs del periodo de tiempo están en la sección de `connection_logs`

![img](img_itsybitsy/Pasted%20image%2020260724191535.png)

![img](img_itsybitsy/Pasted%20image%2020260724191712.png)

## 1. Número de eventos (Mes de marzo 2022)
Hay que poner el margen de tiempo esperado, es decir, lo que se plantea, desde el 1 de marzo de 2022 a las 00:00 (O aproximado) hasta el 31 de marzo de 2022 hasta las 23:59 (o aproximado).

![img](img_itsybitsy/Pasted%20image%2020260724191943.png)

Se puede ver el numero de eventos en ese margen de tiempo: 1482
## 2. Ip asociada al usuario sospechoso
Los eventos para este laboratorio se pueden ver en el margen de tiempo establecido.
Para ver la ip asociada, se puede filtrar por la source_ip (Columna de la izquierda)

![img](img_itsybitsy/Pasted%20image%2020260724192245.png)

Se puede ver una ip que tiene casi todos los eventos del mes (99,6%) y otra que solo tiene apenas 2 (0,4%), lo cual, en el contexto, puede encajar con un comportamiento de servidor C2.

![img](img_itsybitsy/Pasted%20image%2020260724192558.png)

Por lo que parece que la ip maliciosa es `192.166.65.54`
## 3. Nombre del binario (legítimo de windows) que la máquina del usuario usó para descargar el archivo del servidor C2
Analizando los 2 eventos malicosos, en profundidad, se ve como en la sección del user_agent, se puede ver que el usado es bitsadmin.

![img](img_itsybitsy/Pasted%20image%2020260724211951.png)

El user agent suele aportar navegador/versián o herramienta , sistema operativo y versión, dispositivo, ... Esto, en análisis de logs sirve para identificar herramientas, correlaciones, filtrado de eventos,... Aunque hay que destacar que en escenarios reales es "fácil" de falsificar
## 4. La máquina del usuario, ya infectada, se conectó con un sitio web para compartir archivos, actuando a la vez como servidor C2 usado por el malware. Cual es el nombre del sitio web?
Desde el mismo punto que la anterior pregunta se puede responder, desde la misma opción de visualización de "características" del paquete. En la opción de host, siendo pastebin.com

![img](img_itsybitsy/Pasted%20image%2020260724212656.png)

## 5. URL completa del servidor C2 con el que el usuario se conectó
Con el dominio previamente sacado no es suficiente para concluir con todo el dominio, necesito toda la dirección. Para hallarlo, puedo consultar la uri para descubrir el resto de la url. 

![img](img_itsybitsy/Pasted%20image%2020260724212930.png)

En este caso, la uri es `/yTg0Ah6a`, siendo la url completa: `pastebin.com/yTg0Ah6a`
## 6. Se accedió a un archivo en el sito web de compartición de archivos, cual es el nombre?
Para poder ver las características de la url, como los archivos de la misma, se puede directamente buscar. **Siempre en un entorno controlado y/o aislado, no es recomendable acceder a estos sitios maliciosos en general**
Al buscar `pastebin.com/yTg0Ah6a`, sale esto:

![img](img_itsybitsy/Pasted%20image%2020260724213503.png)

Mostrando que el nombre del archivo es secret.txt
## 7. Flag dentro del archivo?
Desde el mismo punto de vista anterior, se puede ver el contenido del .txt, el cual es: `THM{SECRET__CODE}`


