Se puede usar la herramienta que aporta el laboratorio de tryhackme, pero, en teoría, funciona de manera similar a virustotal por ejemplo, plataforma la cual puede servir para dar más realismo al entorno también.
La room me aporta:
- IP: `101.99.76.120`
- SHA-256 del archivo: `5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f`
## 1. Nombre del archivo identificado con el SHA256
Introduciendo el hash en virustotal este es el resultado:

![img](img_invite_only/Pasted%20image%2020260723085645.png)

Parece ser que el archivo, el nombre, es syshelpers.exe
## 2. Tipo de archivo (file type) asociado con el hash sha 256
Tras bajar, en la sección de detalles, se ve el tipo de archivo

![img](img_invite_only/Pasted%20image%2020260723090026.png)

File type--Win32 EXE `executable` `windows` `win32` `pe` `peexe` 
## 3. Ejecutables padres del hash remarcado
En la sección de relations, al bajar, se puede ver un apartado de "execution parents"

![img](img_invite_only/Pasted%20image%2020260723090138.png)

Se ve como son:
- 361GJX7J
- installer.exe
## 4. Nombre del archivo que se descarga
Un poco más abajo de la sección de execution parents de la pregunta anterior, está la sección de dropped files

![img](img_invite_only/Pasted%20image%2020260723090956.png)

El nombre del archivo es AClient.exe, y su hash `dd02c105809e4ca41a5489e585ba025eddb89a91703b73a566c9903e6406a08c`
## 5. Buscar el segundo hash marcado de la pregunta 3, listar los primeros 4 archivos maliciosos
Dandose la situación de que pide los hashes de la pregunta 3, los voy a plasmar:
- 047c5eec0445746862710d20e50a5dd04510b7e625fa5c1f5d48ce078001c0de (361GJX7J)
- `fa102d4e3cfbe85f5189da70a52c1d266925f3efd122091cdc8fe0fc39033942` (installer.exe)
Esto lo he obtenido volviendo a la sección de execution parents, y arriba a la derecha de la misma, dándole a la opción de Export Identifiers
Ahora bien, pide buscar el segundo, por lo que se va a hacer:

![img](img_invite_only/Pasted%20image%2020260723091559.png)

En la sección de dropped files se puede ver:

![img](img_invite_only/Pasted%20image%2020260723092214.png)

A pesar de que pide los maliciosos, el nombre dle primer parece haber cambiado, pero , son el hash, se puede consultar nombres anteriores del mismo:

![img](img_invite_only/Pasted%20image%2020260723092302.png)

Quedando como resultado los siguientes nombres:
- searchHost.exe
- syshelplers.exe
- nat1.vbs
- runsys.vbs
## 6. Analizar los archivos relacionados con la ip marcada, cual es la familia de malware
Se busca la ip en virustotal

![img](img_invite_only/Pasted%20image%2020260723092709.png)

No parece haber informacion de la familia de malware relacionada, por lo que se va a consultar una seccion menos oficial, la parte de community.
En esa sección, en uno de los comentarios, se ve eso:

![img](img_invite_only/Pasted%20image%2020260723093305.png)

Más detalles sobre dicha ip, en este comentario, se ve una seccion de "malpedia", que especifica que la familia de malware es AsyncRAT
## 7. Titulo del reporte original donde la ip y el hash estaban mencionados antes?
Para esto voy a recurrir al comentario anterior

![img](img_invite_only/Pasted%20image%2020260723164735.png)

Se ve que el primer reporte se llamó:
`From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery`
## 8. Que herramienta usaron los atacantes para robar las cookes de chrome?
Ya que se sigue refiriendo al título anterior, voy a buscar dicho reporte inicial

![img](img_invite_only/Pasted%20image%2020260723164927.png)

Siendo el link este: https://research.checkpoint.com/2025/from-trust-to-threat-hijacked-discord-invites-used-for-multi-stage-malware-delivery/
Se ve en el apartado de Key takeaways, el ultimo punto, habla de como los atacantes se saltaban los logins con el robo de cookies usando ChromeKatz
## 9. Qué tecnica de phishing usan los atacantes?
Voy a buscar en el reporte ya mencionado indicadores de phishing o técnicas que el grupo usaba. Aunque, también lo mencionan al principio, otra vez en la sección de key takeaways, siendo la técnica de phishing una llamada ClickFix , es decir, una técnica la cual usan los atacantes para que las victimas ejecuten código malicioso en sus dispositivos. Funciona de manera diferente al descargar archivos, e incitan a la victima a copiar y pegar un comando oculto en el sistema (el cual descarga el malware), aunque el phishing viene del engaño principal a la victima, alarmándola de un fallo o de un malware inexistente.
## 10. Nombre de la plataforma usada para redirigir al usuario al servidor malicioso?
Al continuar leyendo el reporte, se ve como los atacantes se comunicaban o entablaban conversaciones con ellas por medio de Discord (Plataforma de chat)

![img](img_invite_only/Pasted%20image%2020260723170230.png)
