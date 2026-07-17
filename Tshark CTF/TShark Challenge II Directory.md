Al ser un pcap, pero en el lab no hay una aplicación propia para abrirlo directamente (wireshark por ejemplo), se puede usar la herramienta tshark en la carpeta
![Captura](img%20tshark/Pasted%20image%2020260716091200.png)
Usando: `ubuntu@ip-10-128-181-226:~/Desktop/exercise-files$ tshark -r directory-curiosity.pcap `
# 1. Nombre del dominio sospechoso
Para revisar una lista de dominios en el .pcap, se puede usar el comando `ubuntu@ip-10-128-181-226:~/Desktop/exercise-files$ tshark -r directory-curiosity.pcap -T fields -e dns.qry.name | uniq -c`
Siendo estos los resultados:

![Captura](img%20tshark/Pasted%20image%2020260716092018.png))

De los cuales, se puede empezar a descartar o a sospechar de algunos, por ejemplo, el único que llama la atención, ya que parece el menos legitimo a primera vista, es `jx2-bavuong.com`
## 2. Total de solicitudes HTTP al dominio malicioso
Dado que la pista indica que use la función de `http.request.full_uri` para poder buscar el numero de solicitudes del dominio, se va a usar. El comando es: `ubuntu@ip-10-128-181-226:~/Desktop/exercise-files$ tshark -r directory-curiosity.pcap -T fields -e http.request.full_uri | uniq -c`
![Captura](img%20tshark/Pasted%20image%2020260716092723.png))
Como se puede ver, con la url de `jx2-bavuong.com` hay 14 request
## 3. IP del dominio malicioso
Para poder ver la ip del dominio malicioso, se puede filtrar por el nombre del query (jx2-bavuong.com) y dns.flags.response == 1 para poder consultar las respuestas, que, en este caso, sirve para poder detectar la respuesta que ha recibido la victima del sitio web malicioso (el cual se ha filtrado/configurado para que se vea la ip del mismo). El comando final es: `ubuntu@ip-10-128-181-226:~/Desktop/exercise-files$ tshark -r directory-curiosity.pcap -Y "dns.flags.response==1 and dns.qry.name == jx2-bavuong.com"`
![Captura](img%20tshark/Pasted%20image%2020260716093425.png))
Se puede ver como la ip es: 141.164.41.174
## 4. Cual es la información del servidor del dominio
Para poder ver la información de los servidores , del .pcap, se puede filtrar por los campos y por el http.server usando el comando: `ubuntu@ip-10-128-181-226:~/Desktop/exercise-files$ tshark -r ~/Desktop/exercise-files/directory-curiosity.pcap -T fields -e http.server | uniq -c `
![Captura](img%20tshark/Pasted%20image%2020260716095510.png))
Se ve como las caracteristicas del servidor son: `Apache/2.2.11 (Win32) DAV/2 mod_ssl/2.2.11 OpenSSL/0.9.8i PHP/5.2.9`

## 5. Numero de archivos ASCII (TCP Stream en ASCII)
Para poder buscar el stream TCP en ASCII, se va a usar este comando: `tshark -r directory-curiosity.pcap -z follow,tcp,ascii,0 -q`. La parte de -z ... sirve para "seguir" el rastro del primer tcp ( stream 0) en el formato ASCII, legible, y -q para que no muestre datos "de más" reduciendo el bloque de información, haciendolo más facil de analizar. Tras analizar el resultado de la salida, se ven 3 archivos: 123.php, vlauto.exe, vlauto.php.
![Captura](img%20tshark/Pasted%20image%2020260716173734.png))
## 6. Nombre del primer archivo 
Como ya se vieron antes 3 archivos, hay que consultar el primero que se ve, el cual es 123.php:
![Captura](img%20tshark/Pasted%20image%2020260716174122.png))
## 7. Tras exportar los objetos http del .pcap, cual es el nombre del archivo descargado
Aunque se puede ver el .exe disponible para ddescargar en la "página" de las secciones anteriores (vlauto.exe), se va a proceder a exportar los archivos como se requiere.
Para poder exportar el archivo del .pcap, se va a usar este comando:
`tshark -r directory-curiosity.pcap --export-objects http,/home/ubuntu/Desktop -q`
Dicho comando, sirve para exportar los objetos http del pcap, en este caso, al escritorio (Desktop)
![Captura](img%20tshark/Pasted%20image%2020260716174900.png))
Se puede ver como hay un ejecutable (exe), el cual es vlauto.exe
## 8. SHA256 del archivo
Ya con el archivo en el escritorio, se puede usar la terminal para sacar su SHA256. Se usará el comando `sha256sum vlauto.exe` para obtener el sha256 del archivo
![Captura](img%20tshark/Pasted%20image%2020260716175235.png))
Dicho valor es: `b4851333efaf399889456f78eac0fd532e9d8791b23a86a19402c1164aed20de`
## 9. Valor de PEiD de virustotal (Del SHA256)
Para buscalo en virustotal, se usará el sha 256 para encontrarlo y poder obtener información sobre él. En este caso, el PEiD de virustotal. 
Voy a ir a la pestaña de detalles, en la cual suelen estar esta serie de datos o información importante. Tras buscalo, se ha hallado que el valor del PEiD packet es `.NET executable` en la parte de Basic Properties
![Captura](img%20tshark/Pasted%20image%2020260716175744.png))
## 10. En virustotal, como se clasifica el "Lastline Sandbox"?
Tras buscar un rato en virustotal, he encontrado esta sección, Display grouped sandbox reports
![Captura](img%20tshark/Pasted%20image%2020260716181032.png))
En ella,al bajar, se pueden ver varias "alertas" o notificaciones.
Tras bajar a "Dynamic Analysis Sandbox Detections", se ve que la ultima, la mas reciente, la "lastline" es un MALWARE TROJAN:
![Captura](img%20tshark/Pasted%20image%2020260716184109.png))
