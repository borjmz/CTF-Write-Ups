# Machine - boot2root || brute2root

### Análisis
Nos encontramos con un cms Made Simple version 2.2.5 tras buscar un poco encontramos una vulnerabilidad de inyeccion SQL, tras lanzar el exploit de [exploitdb](https://www.exploit-db.com/exploits/46635) obtenemos el hash de la password de Admin en este caso la password era "lalala"


Tras entrar en el panel de administracion, procedemos a lanzar otro exploit de [exploitdb](https://www.exploit-db.com/exploits/44976) pero modificandolo primero ya que han modificado el parametro de CSRF por defecto a  ```_sk_``` una vez esto y con la webshell subida accedemos a ```/home/prequal/backup/id_rsa``` y nos copiamos la privada para acceder por ssh al usuario ```prequals```

En el path ```/home/prequals/user.txt``` se encuentra la flag de user :

```
H-c0n{3ab7568bdae26ac11f6b9e14cad546f9}
```

Una vez dentro del ssh analizamos el sistema para poder elevar privilegios y encontramos en ```/var/backups``` el shadow

```
root:$6$dzULlI9m$PFRv56FDSBt3lwtzzS6Pwk1Zje9.BpW7LZfWK30Wviak5d9oChjUYOJpeOr9ZnrqU5bonapYkCpc55fjG.nop1:17800:0:99999:7::: daemon:*:17779:0:99999:7::: 
bin:*:17779:0:99999:7::: 
sys:*:17779:0:99999:7::: 
sync:*:17779:0:99999:7::: 
games:*:17779:0:99999:7:::
man:*:17779:0:99999:7::: 
lp:*:17779:0:99999:7::: 
mail:*:17779:0:99999:7::: 
news:*:17779:0:99999:7::: 
uucp:*:17779:0:99999:7::: 
proxy:*:17779:0:99999:7::: 
www-data:*:17779:0:99999:7::: 
backup:*:17779:0:99999:7::: 
list:*:17779:0:99999:7::: 
irc:*:17779:0:99999:7::: 
gnats:*:17779:0:99999:7::: 
nobody:*:17779:0:99999:7::: 
systemd-timesync:*:17779:0:99999:7:::
systemd-network:*:17779:0:99999:7::: 
systemd-resolve:*:17779:0:99999:7::: 
systemd-bus-proxy:*:17779:0:99999:7::: 
_apt:*:17779:0:99999:7::: 
messagebus:*:17779:0:99999:7::: 
sshd:*:17779:0:99999:7::: 
mysql:!:17779:0:99999:7::: 
statd:*:17779:0:99999:7::: 
mongodb:!:17828:0:99999:7::: 
postgres:*:18249:0:99999:7:::
prequal:$6$aU.n0NTc$UPmUWgSkn4o2uDAL63UvWokvo0PrKZsz0VKB4FyfgpQ3OHtGydGgaiTvXhZbIBNamMA0VVnKXb0foX/DEzbY41:18249:0:99999:7::: 
tomcat8:*:18249:0:99999:7:::
```
Ahora toca crackear el hash de root, para ello hay que usar el diccionario [Kaonashi](https://github.com/kaonashi-passwords/Kaonashi)


```
$6$dzULlI9m$PFRv56FDSBt3lwtzzS6Pwk1Zje9.BpW7LZfWK30Wviak5d9oChjUYOJpeOr9ZnrqU5bonapYkCpc55fjG.nop1:lp0520
```

Una vez obtenemos la password hacemos ```su root``` y introducimos la password de root y leemos el archivo root.txt donde se encuentra la flag de root.
