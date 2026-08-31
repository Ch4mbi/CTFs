# CTFs — Ch4mbi
Recopilación de retos y rooms completados en tryhackme, con enfoque principal en **análisis forense digital (DFIR)**.  
Cada carpeta contiene el writeup del reto: escenario, herramientas utilizadas, proceso de análisis, etc.

## Retos completados
| Reto | Plataforma | Categoría | Descripción |
|---|---|---|---|
| [Disk Analysis & Autopsy](https://github.com/Ch4mbi/CTFs/blob/main/Disk-analysis-&-Autopsy%20-THM/Disk-analysis-&-Autopsy.md) | TryHackMe | Forense de disco | En este CTF tuve la función de encontrar lo que le pasó a una máquina con diversos usuarios reconstruyendo los eventos, centrandome en qué pasó, como posiblemente ocurrió y otras preguntas que ayudaron a la investigación |
| [Investigating Windows](https://github.com/Ch4mbi/CTFs/blob/main/Investigating%20Windows/Investigating-Windows.md) | TryHackMe | Forense Windows / IR | Anallicé un dispositivo el cual estaba llevando a cabo conexiones remotas con el atacante con el objetivo de robar la información del usuario. Mi función consistió en detectarlo y responder ciertas preguntas qye se me iban haciendo |
| [Cyberheroes](https://github.com/Ch4mbi/CTFs/blob/main/Cyberheroes/Cyberheroes.md) | TryHackMe | Web / Misc | CTF sencillo en el que debí de pasar un login en un entorno controlado |
| [Summit CTF](https://github.com/Ch4mbi/CTFs/blob/main/Summit%20CTF%20THM%20Writeup/Summit.md) | TryHackMe | Respuestas / Análisis | Simulación más teórica que práctica en la que debí de gestionar diferentes ataques/malware que se me pidió analizar y como bloquearlos de diferentes maneras, con el fierwall, hashes, conexiones, solicitudes DNS, etc. |
| [Tshark Challenge II](https://github.com/Ch4mbi/CTFs/blob/main/Tshark%20CTF/TShark%20Challenge%20II%20Directory.md) | TryHackMe | Análisis/Busqueda | Simulación de análisis de un archivo `.pcap` en el que debia identificar diferentes aspectos de las conexiones de dicho archivo de red usando la herramienta tshark |
| [Warzone 1](https://github.com/Ch4mbi/CTFs/blob/main/Warzone%201%20CTF/Warzone%201%20CTF%20TryHackMe.md) | TryHackMe | Análisis multiple | Laboratorio en el que, usando Wireshark, Brim y NetworkMiner, debí de analizar una serie de conexiones sospechisas C2 |
| [Shadow Trace](https://github.com/Ch4mbi/CTFs/blob/main/Shadow%20trace%20CTF/Shadow%20Trace.md) | TryHackMe | Análisis estático y decodificación | Laboratorio en el que debí de analizar un ejecutable sospechoso sin ejecutarlo (estáticamente) con diferentes herramientas |
| [Invite only CTF](https://github.com/Ch4mbi/CTFs/blob/main/Invite%20Only%20CTF/Invite%20only.md) | TryHackMe | Análisis de archivos (hashes) | Ejercicio en el que me suministraban una ip y un hash y debí de descifrar fuentes de dicho malware, ip maliciosa, y buscar reportes gracias a virustotal |
| [ItsyBitsy](https://github.com/Ch4mbi/CTFs/blob/main/ItsyBitsy%20CTF/ItsyBitsy.md) | TryHackMe | Analisis de eventos con elastic | Entorno en el que, usando Elastic, debí de localizar y analizar un evento de un host comprometido y un servidor atacante C2 |
| [Benign CTF](https://github.com/Ch4mbi/CTFs/blob/main/Benign%20CTF/Benign%20CTF.md) | TryHackMe | Análisis de logs/eventos con splunk | Reto en el que debí de usar splunk para comprender una serie de eventos relacionados con un servidor C2 y los recursos humanos de la empresa |
| [Tempest CTF](https://github.com/Ch4mbi/CTFs/blob/main/Tempest%20CTF/Tempest.md) | TryHackMe | Análisis de incidente de seguridad | Reto en el que tuve que analizar evidencias de un incidente de seguridad con algunas técnicas persistencia y escalada de privilegios usando diversas herramientas |
| [CTFs de Boogeyman](https://github.com/Ch4mbi/CTFs/tree/main/Boogeyman) | TryHackMe | Análisis de incidentes de seguridad | Serie de retos en los que tuve que desentrañar diferentes situaciones de un presunto mismo atacante. En los retos estaban:<br>• [Boogeyman 1](https://github.com/Ch4mbi/CTFs/blob/main/Boogeyman/Boogeyman_1/Boogeyman%201%20CTF.md): En este ctf debí de investigar un email phishing y usar diferentes herramientas como `lnkparse` u organización de los archivos al visualizarlos<br>• [Boogeyman 2](https://github.com/Ch4mbi/CTFs/blob/main/Boogeyman/Boogeyman_2/Boogeyman%202.md): En este CTF debí de usar volatility y olevba para analizar otro incidente de seguridad, empezado por un correo de phishing<br>• [Boogeyman 3](https://github.com/Ch4mbi/CTFs/blob/main/Boogeyman/Boogeyman_3/Boogeyman%203.md): En este terccer u último CTF de la linea de Boogeyman, debí de usar la herramienta Elastic (SIEM) para analiar eventos y poder seguir un ataque que ncluye movimiento lateral entre usuarios/máquinas, descargas, volcados de credenciales y hashes, etc. |

## Herramientas

`Autopsy` `Event Viewer` `TryHackMe` `CMD` `Task Scheduler` `Wireshark` `NetworkMiner` `Brim` `Tshark` `Virustotal` `pestudio` `Timeline explorer`

Acceso a mi perfil: [Ch4mbi](https://github.com/Ch4mbi) 
