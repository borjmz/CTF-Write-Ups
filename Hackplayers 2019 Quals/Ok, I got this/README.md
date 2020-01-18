# Ok, I got this

### Descripción 

We have seen a boy with an antenna next to the garage door. In one of his hands it seemed to have a yardstick one. ¿Can you help us find out what the boy was trying to send?
### Análisis
Nos encontramos con un archivo wav captured.wav si lo abrimos con Audacity y ampliamos vemos rápidamente lo que parecen señales de una transmisión, si nos fijamos en la descripción del reto nos da la pista que puede ser un mando de garaje, estos suelen funcionar con el protocolo 433Mhz, la idea era convertir las ondas en 1s y 0s para posteriormente convertirlos a ASCII.


Para ello, mi compañero de equipo (@Gh0stpp7) hizo un pequeño script para esta labor.

```
import librosa

y, sr = librosa.load('captured.wav')

range = 10

counter = 0
final = []
for i in y[1:]:
    if (i + 0.2) > 1:
        counter += 1
        print('igual %g' % i)
    elif counter != 0:
        print('%g cambio a los %d' % (i, counter))
        if counter < range:
            final.append(1)
        else:
            final.append(0)
        counter = 0

print(''.join([str(i) for i in final]))
```
```
010101000110100001100101001000000110011001101100011000010110011100100000011010010111001100100000010010000010110101100011001100000110111001111011001100100011001100110010001101100110001101100110001100110011011001100010001110000011010000110111001100110110010000110110001100010011000101100100001101000011010000111001011001100011000100110001001101110110010000110000001110010011001100111001001110010110011001111100
```
Tras pasar de binario a ASCII el resultado obtenido por el script obtenemos la flag:

```
H-c0n{2326cf36b8473d611d449f117d09399f}
```


