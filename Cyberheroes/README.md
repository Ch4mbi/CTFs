# Cyberheroes

## 1

Acceder a la ip objetivo  
![CH1][Imagenes-CH/CH1]
Revisando , usando diferentes aplicación como burp suite, que no parece ser el caso, para confirmar que usar, 

## 2  
Acceder al login, y acceder al codigo de esa página. Para acceder al login, hay que usar gobuster   

![CH2][Imagenes-CH/CH2]

![CH3][Imagenes-CH/CH3] 
Y se debe analizar el código fuente(Ctrl+u)

## 3

Tras analizarlo, se encuentra esta sección  
![CH4][Imagenes-CH/CH4] 
En ella se pueden sacar datos como en la parte de username  h3ckerBoi y la contraseña 54321@terceSrepuS (SuperSecret@12345 por la “reverse String”) , indicando el código que si tenemos el a.value y el b.value correctos, nos dará la flag 

![CH5][Imagenes-CH/CH5]  
flag{edb0be532c540la150c3a7e85d2466e}
