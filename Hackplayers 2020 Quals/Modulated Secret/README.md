# Modulated Secret - Radio

### Descripción 

A radio amateur has approached us, very worried, saying that he has been able to capture a broadcast in which a secret was being shared. He had to leave because he had a jumping competition, so he sent us the capture of the broadcast. Can you help us recover the secret?
### Análisis

Nos encontramos con un archivo llamado "damn", tras analizarlo usamos la tool SigDigger https://github.com/BatchDrake/SigDigger de BatchDrake muy recomendada ;) 



Una vez cargamos el archivo seleccionamos FM como Demodulador, ponemos el sample rate a 1M y empezamos a oir la transmision como podemos ver en la imagen la transmision esta dividida en 3 canales. Tras escuchar la emision en ingles 300 veces y probar otras 300 veces conseguimos obtener la flag x"DDD 
```
h-c0n{4ed54c18599748c654f614806a645832}
```

