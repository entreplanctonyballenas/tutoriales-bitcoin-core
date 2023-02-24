# Cómo correr un nodo en modo testnet
###### tags: `bitcoin` `cli` `nodo` `node` `test`

_Si ya tienes instalado bitcoin core, con estas instrucciones puedes usar el modo testnet para hacer pruebas sin importar si instalaste en modo cli o con gui_

**Tabla de contenido**
[TOC]

## Autores
EntrePlanctonyBallenas. 
Twitter para correcciones, comentarios o sugerencias: [@entreplanctony1](https://twitter.com/Entreplanctony1)

El presente tutorial fue elaborado para el [Seminario socrático de Mastering Bitcoin](https://libreriadesatoshi.com/) a través de [@libreriadesatoshi](https://twitter.com/libdesatoshi).

En el siguiente enlace puedes encontrar la documentación de referencia:
[https://developer.bitcoin.org/examples/testing.html](https://developer.bitcoin.org/examples/testing.html)

## ¿Qué es la testnet?
El modo **testnet** de bitcoin contiene todas las funciones de la red principal, creación de wallets, la red p2p, minería, sin embargo los bitcoins de prueba (test-coins) están creados para no tener valor y la dificultad de minado se mantiene muy baja para que los bitcoins de esta red nunca tengan valor. Se usa para aprender a utilizar transacciones de bitcoin sin gastar bitcoins reales, también es un entorno de pruebas seguro en el cual los desarrolladores pueden probar actualizaciones sin afectar la red de bitcoin.
## IBD
La primera vez que entres en modo testnet, tu nodo va a pasar por el proceso de IBD. Pero la cadena de bloques de la red **testnet** de bitcoin es mucho más ligera mide menos de **40gb** por lo que tardará mucho menos tiempo en descargarla y validarla, en menos de 1 dia deberia estar listo tu nodo para poder ser utilizado en modo testnet.

### Interrupción del IBD
Igual que en modo normal de bitcoin (mainnet) el software de _Bitcoin Core_ puede ser detenido en cualquier momento, y al arrancarlo nuevamente este va a comenzar el IBD desde el punto en que se dejó. Esto quiere decir que si primero instalaste y cofiguraste tu nodo en modo **mainnet** e interrumpes el proceso para arrancar en modo **testnet** la información que ya descargaste no se pierde, solo se añade ahora información de la red de testnet a lo que ya tenías. 

Puedes trabajar en modo testnet y cuando termines volver a poner tu nodo en modo **mainnet** cuando lo necesites.

## Activar el modo testnet
:::info
Lo primero a mencionar es que al activar el modo testnet se agregaran archivos debajo del path donde has configurado que se guarde la información de carteras y bloques por lo que en dicho disco debes tener espacio suficiente (**40gb libres aprox**).
:::
Si en tu computadora estas ejecutando el software de _Bitcoin Core_ debes detenerlo:
#### Detener Bitcoin Core en linea de comando
```sass
bitcoin-cli stop
```

#### Detener Bitcoin Core en interfaz gráfica
Simplemente cierra la ventana de la interfaz gráfica en tu sistema operativo y ++espera a que el aviso de no apagar tu equipo desaparezca++
### Modo testnet desde linea de comandos
Localiza el archivo ejecutable de **bitcoind** (esta ubicación varia según tu sistema operativo y de si lo instalaste desde archivo ejecutable o si lo compilaste desde código fuente).
:::spoiler Excepción para macOS
Si instalaste la interfaz gráfica de Bitcoin Core desde el archivo **.dmg** descargable, este no incluye el ejecutable para bitcoind, por lo que deberás ++descargar e instalar++ usando las [instrucciones de bitcoin.org](https://bitcoin.org/en/full-node#osx-daemon)
:::

Ya que localizaste el archivo **bitcoind** en tu computadora, abre una terminal, y ejecuta el comando con el argumento **-testnet**, esto sirve para indicarle a _Bitcoin Core_ que va ejecutarse en modo testnet, pero sólo de manera momentánea.
#### Para Linux y macOS:

```sass
bitcoind -testnet -daemon
```
#### Para Windows:
```sass
bitcoind.exe -testnet -daemon
```

:::warning
:warning: Todos los sub-comandos que corras en modo testnet a partir de este punto, deben llevar el argumento **-testnet** de otra forma mandarán un error de ejecución, algunos ejemplos:
bitcoin-cli -testnet getmempoolinfo
bitcoin-cli -testnet stop
:::

### Modo testnet desde interfaz gráfica
Abre una terminal, localiza el archivo ejecutable de bitcoind (esta ubicación varia según tu sistema operativo y si lo compilaste o instalaste el ejecutable) y ejecuta para **Linux** y **macOS**:
```sass
bitcoin-qt -testnet
```
Para **Windows**:
```sass
bitcoin-qt.exe -testnet
```


## Cambios en bitcoin.conf
 
Para hacer que tu nodo levante siempre en modo testnet, debes agregar la siguiente linea en el archivo de configuración bitcoin.conf:
 
```gherkin
testnet=1
```
Con este cambio evitarás tener que introducir el argumento **-testnet** con cada subcomando al usar **bitcoin-cli**.

Si en algún momento quieres volver a habilitar el modo "**mainnet**" comenta la linea anteponiendo un "**#**" al inicio de la línea o borrala completamente. Después de hacer el cambio, deberás re-iniciar el cliente de _Bitcoin Core_.

## Blockchaininfo
Ahora que tienes la posibilidad de cambiar entre modo testnet y mainnet quizá se te olvide en que modo arrancaste la ultima vez (sobre todo si modificaste el archivo `bitcoin.conf`), para que confirmes esto abre una terminal y ejecuta `getblockchaininfo` en el campo de `chain` *(línea 3)* de la siguiente salida, verás **testnet** ó **mainnet**.
```gherkin=
$ bitcoin-cli getblockchaininfo
{
  "chain": "test",
  "blocks": 1088,
  "headers": 139999,
  "bestblockhash": "0000000063d29909d475a1c4ba26da64b368e56cce5d925097bf3a2084370128",
  "difficulty": 1,
  "mediantime": 1337966158,
  "verificationprogress": 0.001644065914099759,
  "chainwork": "0000000000000000000000000000000000000000000000000000044104410441",
  "pruned": false,
  "softforks": [

```