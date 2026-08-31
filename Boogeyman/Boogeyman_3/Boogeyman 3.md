Para esta room estaré usando, como pide la room, elastic
## 1. Cual es el PID del proceso que ejecutó el payload fase 1?
Para esta analicé los correos enviados en el contexto y los contenidos:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260811224349.png))

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260811224403.png))

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260811224416.png))

Se ve como en el tipo de archivo es un html application en lugar de un pdf
Se ve como está en descargas, por lo que puedo llegar a intuir que se descargó de algún sitio de internet, creando un proceso para llevarlo a cabo por ejemplo.
Usando elastic:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260811224648.png))

Voy a probar con la extensión de html usando el comando de `file.extension=html` ya que el tipo de archivo es HTML Application

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260811224907.png))

Se ve como el ProcessId es 6392 en el único elemento en el que se resalta en un primer lugar la app html, ya que en los otros 4, parece que son posteriores en tiempo
## 2. El payload intentó, en la fase 1 , implantar un archivo en otra localización con un comando, cual es dicha linea de comando?
Aun con la misma linea de `file.extension=html`, no encontraba nada. 
Relacioné los eventos, en la anterior pregunta el PID era `6392` por lo que en el buscador de elastic, voy a buscar ese mismo id. Poniendo PID en el buscador, no es un comando como tal, pero lo lee bien

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827105926.png))

A parir de aqui, selecciono `process.pid` y añado `=6392` para buscar ese mismo id.
He añadido las columnas mas importantes para verlo mejor

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827110220.png))

Ahora, con los resultados, he encontrado una `process.command_line`:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827110324.png))

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827110342.png))

El nombre, objetivamente por ser un reto, es casi intuitivo. Sin embargo, he preferido guiarme por lo que hace el comando `"C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat` 
He descubierto que ese mismo comando lo que hace es copiar, usando xcopy.exe, el archivo review.dat, desde una carpeta "D" a pasar como un archivo temporal (temp) en "C"
## 3. El archivo previamente implantado fue al final usado por el payload fase 1, cual es la linea de comando completa?
Ya que se que el archivo es review.dat, voy a buscar en base a ese mismo archivo.

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827172511.png))

Nada mas buscarlo, me di cuenta de los comandos de whoami, los cuales se pueden dar cuando un atacante ya tiene o cree tener acceso a la máquina.
Voy a centrarme en buscar en las columnas de `process.parent.command_line` y `process.command_line`
También cabe añadir que el uso de dicho archivo .dat demuestra que si que se usó.

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827173316.png))

Se ve como, justo después (Encima en base al orden) del trastadar el .dat a temp, se ejecuta la linea:`"C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer` 
Adicionalmente, se ve como el nombre del proceso es rundll32.exe, una herramienta legítima de windows
## 4. El payload fase 1 estableció persistencia, cual es el nombre de la tarea programada creada por el script malicioso?
Desde el mismo punto que la pregunta anterior, justo en la fila de encima, se puede ver como se crea una nueva tarea:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827173656.png))

Lo que pone, ya que está cortado es: `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" $A = New-ScheduledTaskAction -Execute 'rundll32.exe' -Argument 'C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat,DllRegisterServer'; $T = New-ScheduledTaskTrigger -Daily -At 06:00; $S = New-ScheduledTaskSettingsSet; $P = New-ScheduledTaskPrincipal $env:username; $D = New-ScheduledTask -Action $A -Trigger $T -Principal $P -Settings $S; Register-ScheduledTask Review -InputObject $D -Force;`
Se ve como se crea una nueva tarea programada, para ejecutarse diariamente a las 6 am.
Se ve como en la parte de `Register-ScheduledTask`, se ve el nombre con el que se ha registrado dicha tarea: `Review`
## 5. La ejecución del archivo malicioso en la maquina inició una potencial conexión C2, cual es la ip y el puerto usados para esta conexión?
Voy a buscar lo que vi antes, lo de que se ejecute con rundll32.exe, proceso legítimo de windows, para ver las conexiones posiblemente llevadas a cabo con ese proceso y/o la tarea programada Review.
En la pregunta anterior, noté que también pasó a ejecutar directamente rundll32.exe, por eso pensé que al ser una herramienta legítima de windows, podría pasar más "desapercibida".
Tras buscarlo, noté que hay, evidentemente, muchas filas. Sin embargo, noté que, a partir de cierto punto, comenzaron a producirse conexiones todo el rato a la misma ip y puerto: `165.232.170.151:80`

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827182445.png))

Aparentemente, es la única conexión que aparece y el único puerto con rundll.exe

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827182614.png))

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260827182544.png))

## 6. El atacante descubrió que el acceso actual es de administrador. ¿Cual es el nombre del proceso usado por el atacante para ejecutar un bypass UAC?
He encontrado como el atacante parece enumerar grupos

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828102833.png))

Y también, antes de eso parece que intentó saber los usuarios, en la red parece ser, y buscó específicamente el grupo de administrators

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828102903.png))

Tras darle a la pista, por no saber avanzar, supe que debia de buscar tecnicas de bypass de UAC.
Encontré algunas:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828103550.png))

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828103608.png))

Curiosamente, en el punto 1, registry key manipulations, hay uno: `fodhelper.exe`, el cual me suena haber visto en los logs/registros de review.dat

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828103724.png))

En la propia página de elastic (`https://www.elastic.co/es/blog/exploring-windows-uac-bypasses-techniques-and-detection-strategies`) pude confirmar el uso de fodhelper.exe

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828103931.png))

También descubrí que fodhelper.exe es un archivo legitimo de windows, el cual puede servir para bypasear el UAC
## 7. Teniendo acceso privilegiado a la máquina, el atacante intentó introducir las credenciales en la maquina. ¿Cual es el enlace de github usado por el atacante para descargar una herramienta para volcar las credenciales?
*Nota antes de nada: En esta pregunta, plasmé un comando en obsidian, donde documento todo, sobre el posible script malicioso que podia contener dicho enlace a github, para poder visualizar completo y plasmar lo que posiblemente descubriera (No habia nada). El antivirus me lo detectó y me eliminó el .md entero. Luego caí en la cuenta de que una de las formas principales en las que leo los .md es con vs code, por lo que puede que lo haya detectado como un script malicioso real contra mi ordenador. Lección aprendida*
Ya que se que estoy buscando un enlace de github, voy a buscar eso mismo.
No hay resultados al poner en el buscador directamente "github", seguramente porque no esté configurado así y busque en base a ciertas columnas, entendible, por eso voy a probar otro enfoque.
Buscando en internet el funcionamiento más profundo de elastic, recordé, también por otra room que hice, el aplicar filtros correctamente. En este caso, el fallo fue mio. Solo me acordé de poner las "" y, en realidad, eso buscar coincidencias precisas, aparentemente en la columna de `process.name`, mientas que los ** se encargan de buscar cualquier valor que contenga lo que hay en ellos, en todas las celdas.

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828195437.png))

Gracias a la busqueda, pude ver el comando que intentaba descargar algo de github

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828195552.png))

Se puede ver que en el comando se usa powershell para descargar mimikatz de este link: `https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip`
## 8. Después de volcar exitosamente las credenciales en la máquina, el atacante usó las credenciales para ganar acceso a otra máquina, cual es el nombre de usuario y el hash?
Voy a volver a aplicar el mismo filtro de antes, ** , pero esta vez con mimikatz, para buscar el posible uso de esa herramienta. 

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260828202759.png))

Hay varios resultados, por lo que voy a empezar a buscsar, principalmente por las columnas de comandos `process.command_line` y `process.parent.command_line`
Tras buscar durante un rato, encontré un nombre de usuario (user) diferente al de `administrator`, `itadmin`, por lo que puede ser que esa sea la nueva máquina a la que el atacante se enfocó.

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260829111239.png))

Con su respectivo hash de lal contraseña: `F84769D250EB95EB2D7D8B4A1C5613F2` este hash parece ser un timpo de hash ntlm, solo de la contraseña
## 9. Usando las nuevas credenciales, el atacante intentó enumerar archivos compartidos.¿Cual es el nombre del archivo al que el atacante accedió remotamente?
Necesité ayuda externa en esta pregunta porque no sabia ya como buscarlo, ya que buscando por dicho nombre de usuario, aparecían demasiados resultados y me era imposible comprenderlos.
Descubrí que también se pueden buscar usos de mimikatz por medio de `*mimi*`.
Luego, tras buscar por un rato, encontré en internet que los atacantes pueden usar herramientas como Powerview para enumerar archivos (.ps1), por lo que me aferré a eso.
Así, pude encontrar el archivo:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260829115254.png))

`IT_Automation.ps1`
## 10. Después de obtener el contenido del archivo remoto, el atacante usó las nuevas credenciales para moverse lateralmente, ¿Cual es el nuevo set de credenciales descubierto por el atacante?
Volviendo a donde antes, `*mimikatz*`, probé a añadir más columnas para poder ver mejor los resultados:
- user.domain
- user.id
- user.name

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260829120246.png))

Tras un buen rato, tuve que recurrir a una guia, para obtener una pista para poder avanzar adecuadamente.
Descubrí que debia de buscar comandos de cmd.exe y/o powershell.exe , y filtrar los resultados por el proceso 1, el de winlog.event_id.
Tras un rato buscando encontré una alerta que contenia credenciales:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260829162737.png))

Es decir , `$credential`, en este caso, segun puedo ver,`QUICKLIGISTICS\allan.smith`

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260829162940.png))

Tras seguir buscando encontré algo más:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260829163026.png))

He encontrado: `(ConvertTo-SecureString 'Tr!ckyP@ssw0rd987' -AsPlainText -Force`. Esto parece ser la contraseña, la cual estaría dentro del usuario, siendo todo junto: `QUICKLIGISTICS\allan.smith:Tr!ckyP@ssw0rd987`
## 11. ¿Cual es el hostname de la máquina lab del atacante en su intento de movimeinto lateral?
Ya que antes el atacante se movió lateralmente con las credenciales `QUICKLIGISTICS\allan.smith:Tr!ckyP@ssw0rd987`, puedo ver en esa misma linea de comandos, el nombre del ordenador, `WKSTN-1327`, justo después de ComputerName, en la imagen anterior se ve justo debajo de la contraseña `Tr!ckyP@ssw0rd987`
## 12. Usando el comando malicioso ejecutado por el atacante de la primera máquina para moverse lateralmente, cual es el parent process name del comando malicioso ejecutado en la segunda máquina comprometida?
Ya se que la máquina es WKSTN-1327, voy a probar a filtrar por ese parámetro, principalmente porque ese es el intento de movimiento lateral del atacante, esa es la primera máquina, y quiero descubrir el "proceso padre".

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830115300.png))

Aplicando el filtro directamente de manera simple, de `host.name` + `is` + `WKSTN-1327` para buscar los resultados en un primer lugar de esa máquina, aparecen varios resultados

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830115332.png))

Por lo que ya que es un proceso malicioso, tal vez haya usado el event.id 1, que en windows consiste en creación de procesos, en sysmon
Usaré `event.code:1` 

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830115610.png))

Hay ahora menos resultados haciendo más posible la búsqueda. Estoy buscando un parent process name (última columna) y, habiendo seguido los procesos/pasos del ataque, puedo intentar encontrar más facilmente el nombre.
Al investigar, descubrí un proceso que no reconocía, de los del día a dia:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830121136.png))

`wsmprovhost.exe`
Dicho proceso, según he encontrado buscando sobre él, he hallado que sirve como host en segundo plano para poder gestionar conexiones de powershell a través de la red mediante el protocolo winrm (Windows Remote Management) y WS-Management, o sea, que puede servir como herramienta de posible movimiento lateral.
Cabe destacar que no estaba muy seguro de que fuera esa la respuesta, por lo que tuve que contrastar con guías.
## 13. El atacante volcó los hashes en la segunda máquina, ¿cual es el nombre de usuario y el hash de las nuevas credenciales volcadas?
Puedo probar a volver a buscar mimikatz con el event id 1 , de creación de procesos, en este caso, de volcado en una nueva máquina

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830125234.png))

Buscando, encontré este:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830131753.png))

Este mismo contenía algunas cosas que me llamaban la atención:
- Otra vez uso de NTLM (De las 20, no todas lo usan)
- Uso de mimikatz, evidentemente
- Usuario de administrator, el resto que usan NTLM son itadmin u otros
Esas cosas me hicieron sospechar de que el atacante consigió acceso a la cuenta de admin con el volcado de las credenciales en ntlm, siendo la respuesta: `administrator:00f80f2538dcb54e7adc715c0e7091ec`
## 14. Tras ganar acceso al controlador de dominio, el atacante intentó volcar los hashes por medio de un ataque DCSync. Aparte de la cuanta de administrador, ¿que cuenta también volcó?
Aun con la misma búsqueda de la anterior pregunta, cronológicamente en orden, vi un usuario que no me sonaba de nada, `backupda`

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830132933.png))

Sin embargo, aunque por algun casual esa era la respuesta, creo que no era la manera correcta de encontrarla.
Buscando en internet, descubrí que los ataques dcsync, se pueden filtrar con solo `*dsync*` en elastic

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830133222.png))

Se ven solo 3 eventos, y el primero y el último son del usuario `administrator`, por lo que se puede descartar para esta pregunta.
1 de los eventos, por otro lado,contiene las características del nuevo usuario sobre el que se volcaron los hashes, `backupda`

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830200622.png))

## 15. Tras volcar los hashes, el atacante intentó descargar otro archivo remoto para ejecutar un ransomware. ¿Cual es el link usado para descargar dicho ransomware?
Partiendo del mismo punto anterior, el volcado a backupda no es el evento más reciente, sino al otro pc, como admin. Me acabo de fijar de que el host.hostname no es el de WKSTN-1327, o sea, ese es el de más abajo, el último, después vino el de backupda en el host.hostname DC01 según veo, y, el más reciente, a admin, pero aun al host.hostname DC01, por lo que voy a partir de esa máquina con el mismo event id 1 de posible creación de un proceso para descargar y posiblemente ejecutar dicho ransomware de un enlace.

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830202839.png))

Tras un rato buscando con ctrl + f, ya que intuía que iba a ser un link http, encontré esto:

![img_captura_de_pantalla](img_boogeyman_3/Pasted%20image%2020260830203158.png))

Se puede ver como usa powershell para descargar un posible ransomware (casi evidente por el nombre) por medio de este link: `http://ff.sillytechninja.io/ransomboogey.exe`. A demás, la máquina atacada parece ser `WKSTN-132`, la misma que se vio antes, y usa iwr (Invoke web request) para descargar el rasomware posiblemente de un servidor del atacante/un servidor malicioso. Veo también como fue hacia el nuevo usuario itadmin, visto previamente, ya que lo guarda en `C:\Users\itadmin\ransomboogey.exe` 
