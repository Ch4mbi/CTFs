En el laboratorio del ctf me permiten usar estas herramientas:
- Brim
- Wireshark
- NetworkMiner
![Captura](Warzone_1_img/Pasted%20image%2020260718092422.png)

## 1. Cual fue la firma de la alerta para "Malware command and control activity detected"?
Ya que me pide hallar un indicador de dicho ataque C2, se puede usar brim para detectarlo. Seleccionando la app de Brim y seleccionando el archivo de Zone1.pcap
![Captura](Warzone_1_img/Pasted%20image%2020260718092758.png)
Poniendo el siguiente comando en el buscador se puede filtrar para alertas (para descartar "ruido" en base al archivo original):
`alert_type=="alert"`, dando como resultado:
![Captura](Warzone_1_img/Pasted%20image%2020260718093147.png)
Sin embargo, se puede "filtrar por la pregunta", en este caso, añadiendo "Malware command and control activity detected" para especificar el tipo de alerta, en este caso de C2, quedando así el comando: `alert_type=="alert" | "Malware Command and Control Activity Detected"`. Así, se puede ver en la columna de alert.signature la "firma": ET MALWARE MirrorBlast CnC Activity M3
![Captura](Warzone_1_img/Pasted%20image%2020260718093507.png)
## 2. IP de origen de dicha alerta? 
Desde el punto de partida de la anterior preguna (la 1) se puede ver la ip de origen, 172.16.1.102
![Captura](Warzone_1_img/Pasted%20image%2020260718093703.png)
## 3. IP de destino?
De igaul manera que la anterior pregunta, se puede ver la ip a la que se ha hecho la solicitud: 169.239.128.11
![Captura](Warzone_1_img/Pasted%20image%2020260718093706.png)
Esto, al ser un ataque C2, puede haber sido que la victima (172.16.1.102) envie solicitudes al atacante o al dominio del mismo (169.239.128.11)
## 4. En virustotal, en community, a que grupo se le atribuye dicha IP?
La teoría anterior se puede confirmar al ver que la ip del sitio web (169.239.128.11) es presuntamente paliciosa segun virustotal:
![Captura](Warzone_1_img/Pasted%20image%2020260718094537.png)
Se puede ver que, en la sección de community, se indica el tipo de grupo APT (TA505)
## 5. Familia de malware?
Desde la misma pregunta anterior se puede ver, en virustotal, la familia de malware, la cual parece ser: MirrorBlast
![Captura](Warzone_1_img/Pasted%20image%2020260718094537.png)
Sin embargo, también se puede ver en brim, en la columna de alert.metadata.malware_family
![Captura](Warzone_1_img/Pasted%20image%2020260718095040.png)
## 6. En virustotal, respecto al dominio,  cual fue la mayoria de archivos listados bajo en titulo de Communicating files?
Desde virustotal, aun con la misma ip (169.239.128.11), en la sección de relations, se puede ver la sección de Communicating Files, zona la cual puede ayudar a saber que archivos suele comunicar dicho dominio, viendo que la mayoria de tipo de archivos son Win32 EXE. Sin embargo, tiene varias filas, por lo que se va a buscar en ellas
![Captura](Warzone_1_img/Pasted%20image%2020260718095752.png)
Se ve que, a pesar de la mayoria de archivos Win32 EXE, hay también archivos Windows Installer
## 7. Tras inspeccionar el tráfico de red bajo la ip, cual es el user-agent del tráfico? 
Para inspeccionar el tráfico se puede usar wireshark con un filtro que "seleccione" la ip específica: `ip.adrr == 169.239.128.11`
![Captura](Warzone_1_img/Pasted%20image%2020260718101327.png)
Ya con el filtro aplicado, se puede seleccionar la opción de seguir dicho flujo tcp, lo cual llevaría a esta sección:
![Captura](Warzone_1_img/Pasted%20image%2020260718101706.png)
Aquí, se puede ver una sección User-agent: REBOL View 2.7.8.3.1
## 8. Había varias ip relacionadas con el ataque (2), cuales eran?
Desde brim, se pueden filtrar por paquetes http con el comando `_path=="http" | uniq`, lo cual filtra por paquetes http
![Captura](Warzone_1_img/Pasted%20image%2020260718103747.png)
Se pueden ver varias ip adicionales diferentes a la del atacante. Sin embargo, hay debo de saber cuales son las que estan relacionadas con el ataque.
Es filtro aplicado se le puede añadir un filtro adicional que muestra columnas especificas, en este caso, puede ser buena idea seleccionar columnas que:
- Miestren la ip de la victima
- Ip del sitio web q responde a las solicitudes 
- Tal vez el user agent puede ser util para identificar las ip maliciosas (por virustotal antes)
Por lo que el filtro puede quedar así: `_path=="http" | uniq | cut id.origin_h, id.resp_h, user_agent`
Al bajar, se puede ver esto:
![Captura](Warzone_1_img/Pasted%20image%2020260718104318.png)
Si se contrasta con los resultados de la ip principal en virustotal, se puede ver como hay 2 archivos Windows Installer, los cuales tienen la "ip relacionadas":
- 192.36.27.92
- 185.10.68.235
## 9. Cuales eran los archivos descargados? (De las IP)
Para ver los archivos descargados se puede usar networkiner sobre el .pcap , el cual dependiendo de la versión los puede presentar al usuario.
Debo de buscar las IP en la sección de Files
![Captura](Warzone_1_img/Pasted%20image%2020260718104829.png)
Ahí, me debo de centrar en las Ip mencionadas antes (192.36.27.92, 185.10.68.235).
He hallado esto respecto a dichas ip, y sus archivos descargados:
- 185.10.68.235 - filter.msi
- 192.36.27.92 - 10opd3r_load.msi
## 10. Tras inspeccionar el trafico en busca de los archivos mencionados, 2 archivos fueron guardados en el mismo directorio, cuales eran?
Dada la pregunta, se puede buscar 1 solo para hallar la respuesta de ambas rutas. Ya que se sabe la ip y el archivo (en este caso voy a usar 185.10.68.235 - filter.msi), puedo usar wireshark para ver las propiedades del paquete/s. Usando el comando/filtro `ip.addr==185.10.68.235` para ver dichos paquetes con esa ip
![Captura](Warzone_1_img/Pasted%20image%2020260718172913.png)
Es importante recalcar que uno debe de fijarse en las comunicaciones victima - servidor, no las respuestas del servidor a la victima, es decir, que la columna destination sea 185.10.68.235.
También seleccioné el paquete cuya información incluia http, lo cual puede ser el método de descarga de dicho archivo que se está buscando
![Captura](Warzone_1_img/Pasted%20image%2020260718173528.png)
Tras darle a follow http stream, busco una ruta.
En dicha ruta encuentro una, siendo `C:\ProgramData\001\arab.bin001`
![Captura](Warzone_1_img/Pasted%20image%2020260718173703.png)
Sin embargo, son 2 rutas, o mas bien 2 archivos. Encima del marcado se puede ver otro, misma ruta, pero diferente archivo, arab.exe.
Quedando como respuesta ambas rutas `C:\ProgramData\001\arab.bin001 y C:\ProgramData\001\arab.exe`
## 11. De igual manera, pero con el otro archivo .pcap, cual es la ruta?
Se debe de hacer lo mismo, pero esta vez con el otro archivo, el de la ip 192.36.27.92, con este comando `ip.addr==192.36.27.92`
![Captura](Warzone_1_img/Pasted%20image%2020260718174405.png)
Tras darle a follow al flujo http, se abre una ventana similar a la de antes.
![Captura](Warzone_1_img/Pasted%20image%2020260718174720.png)
Siendo las rutas `C:\ProgramData\Local\Google\rebol-view-278-3-1.exe y C:\ProgramData\Local\Google\exemple.rb`
