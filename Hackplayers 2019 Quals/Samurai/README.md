# Samurai - Stego

### Descripción 

"The general who is skilled in defense hides in the most secret recesses of the earth"

Sun Tzu's Art of War
### Análisis

Nos encontramos con un archivo llamado samurai.png.



Le tiramos la herramienta binwalk y extraemos un archivo llamado wind.wav si nos llevamos el archivo a Sonic Visualizer y vemos el espectro del mismo vemos la palabra "Shinobi" como se puede ver en la imagen a continuacion.



Tras analizar el LSB de la imagen nos damos cuenta que esta cifrado con fernet y utilizamos esta tool [Stegpy](https://github.com/dhsdshdhk/stegpy) para descifrarlo con la password que hemos visto en la imagen anterior "SHINOBI" tras lanzar el script nos devuelve la flag valida:
```
H-c0n{3899dcbab79f92af727c2190bbd8abc5}
```
# Samurai - Stego

### Descripción 

"The general who is skilled in defense hides in the most secret recesses of the earth"

Sun Tzu's Art of War
### Análisis

Nos encontramos con un archivo llamado samurai.png.



Le tiramos la herramienta binwalk y extraemos un archivo llamado wind.wav si nos llevamos el archivo a Sonic Visualizer y vemos el espectro del mismo vemos la palabra "Shinobi" como se puede ver en la imagen a continuacion.



Tras analizar el LSB de la imagen nos damos cuenta que esta cifrado con fernet y utilizamos esta tool [Stegpy](https://github.com/dhsdshdhk/stegpy) para descifrarlo con la password que hemos visto en la imagen anterior "SHINOBI" tras lanzar el script nos devuelve la flag valida:
```
H-c0n{3899dcbab79f92af727c2190bbd8abc5}
```

