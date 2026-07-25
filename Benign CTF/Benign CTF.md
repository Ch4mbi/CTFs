# Benign

https://tryhackme.com/room/benign

En el escenario, solo se pudo, por recursos limitados, extraer los logs de ejecución de procesos (ID de evento: 4688) y se introdujeron en splunk con el index `win_eventlogs`.
Departamentos:
- IT
	- James
	- Moin
	- Katrina
- HR
	- Haroon
	- Chris
	- Diana
- Marketing
	- Bell
	- Amelia
	- Deepak
## 1. Cuantos logs son del mes de marzo de 2022?
Para este caso, hay que establecer un margen de tiempo desde el inicio del mes en el año, hasta el final del mismo mes en splunk. Obviamente con el "conjunto" en el que se encuentra, es decir, escribir, aparte de especificar la fecha, `index="win_eventlogs"`
![[Pasted image 20260725113906.png]]
Se ve como hay 13959 eventos
## 2. Parece haber una alerta de un impostor en las alertas, cual es su nombre (usuario)?
Mirando la columna de la izquierda, decidí acceder a la sección de "UserName", en la cual hay 11 valores, habiendo en la empresa 9 empleados relacionados y el sistema que suele tender a generar eventos por si solo, dejando uno más
![[Pasted image 20260725114745.png]]
Seleccionando la opción de "rare values" se puede acceder a una gráfica en la que pone la cantidad de eventos de cada usuario, de los 11, y se ve como se "repite un usuario"
![[Pasted image 20260725115233.png]]
Ese usuario parece estar intentando psar desapercibido a ojos humanos, copiando un nombre, el de Amelia, por Amel1a
## 3. Que usuario del departamento de recursos humanos fue detectado ejecutando tareas programadas?
En un primer lugar, se especifica el departamento de recursos humanos, es decir, las personas implicadas son: Haroon (haroon), Chris (chris.fort) y Diana (Daiana).
Tras buscar en internet como es el comando para scheduled tasks , simplemente `schtasks`, añadiendolo al win_eventlog, sale este resultado a analizar, tras poner `index="win_eventlogs" AND schtasks`:
![[Pasted image 20260725120945.png]]
Sin embargo, al haber varios eventos, hay que correlacionarlos bien, de ahí, la lista del principio de cada uno, ya que especifica recursos humanos (HR). De ahí, en la sección de la izquierda, bajo a username y ahí puedo ver que el único de recursos humanos que ejecutó una schtask fue Chris (Chris.fort)
![[Pasted image 20260725121109.png]]
## 4. Que usuario de recursos humanos ejecutó un proceso del sistema (LOLBIN) para descargar un payload de un host de compartición de archivos?
Tras buscar en internet, se ve como el proceso LOLBIN corresponde a Living Off the Land Binary, el cual los atacantes pueden usar para evadir sistemas de seguridad/autenticación al contar con herramientas legitimas. Puedo ver como algunas de dichas herramientas, en windows suelen ser Powershell, certutil (para descargar archivos),MSHTA (ejecución de scripts HTML) o Rundll32.
Un detalle que he omitido es el hecho de que lo usó para descargar un payload. Accedí a LOLBAS (https://lolbas-project.github.io/) para buscar dicho archivo o método especifico para empezar a descartar/buscar procesos especificos. 
Hay varios que se pueden considerar comunes, por lo que voy a ir probando uno a uno a falta de un comando que pueda filtrar por eventos que entren en ese campo de LOLBIN
Escribiendo: `index="win_eventlogs" AND certutil' 
Solo aparece un evento:
![[Pasted image 20260725123557.png]]
Se ve como el único, que coincide con uno de recursos humanos, es haroon.
Cabe destacar que es un proceso lento, por lo que es más conveniente emplear otro método para sacar la respuesta más conveniente, por ejemplo, se pueden filtrar los resultados en base al Hostname, añadiendo `HostName="*HR*"`, lo cual, buscará todos los hostname que contengan HR, independientemente de lo que tengan delante o detrás, en este caso, haroon se llama HR_01, dando a entender que los otros miembros pueden tener un nombre similar
![[Pasted image 20260725173037.png]]
Aun así, se ven muchos eventos:
![[Pasted image 20260725173125.png]]
Por lo que se puede añadir otra "condición", por ejemplo, que se haya ejecutado una linea de comandos (CommandLine) para visualizar los resultados
![[Pasted image 20260725173300.png]]
Escribiendo `index="win_eventlogs" AND HostName="*HR*" | rare limit=50 CommandLine`
Lo añadido nuevo al comando muestra valores menos frecuentes (rare) del CommandLine, con un limite de los 50 valores más raros, y ahí se ve. De ahí se puede justificar la búsqueda de certutil
## 5. Qué proceso de sistema (lolbin) fue usado para descargar un payload de internet para pasar los controles de seguridad?
Desde el mismo punto de vista anterior, se ve como el processname es: ` C:\Windows\System32\certutil.exe` 
## 6. Cual fue la fecha en la que el binario fue ejecutado por el host infectado?
Desde la pregunta 4 se puede seguir viendo la fecha en los detalles de ese mismo evento, `2022-03-04T10:38:28Z`
![[Pasted image 20260725172626.png]]
## 7. A que sitio web third-party se accedió para descargar el payload malicioso?
En la command line del mismo evento, se puede ver la url que el comando usaba para descargar el archivo malicioso
![[Pasted image 20260725174824.png]]
Siendo `https://controlc.com/e4d11035`, es decir, que el sitio web es `controlc.com`
## 8. Cual es el nombre del archivo que fue descargado en la máquina host del servidor C2 durante la fase de post-explotación?
En la misma sección de la pregunta anterior se puede ver, `benign.exe`
## 9. El archivo malicioso descargado del servidor C2 contiene una flag
Decidí buscar la página web en un sitio seguro.
Una vez accedí a ella, se vió un flag.txt, el cual es el archivo malicioso, y su contenido, que es la flag: `THM{KJ&*H^B0}`
![[Pasted image 20260725175848.png]]
## 10. Cual es la URL a la que el host infectado se conectó?
Ya se mencionó la url completa antes a la que la victima se conectó al sitio malicioso, en la pregunta 8. El sitio web es: `https://controlc.com/e4d11035` 