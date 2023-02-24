# Instalación de Bitcoin Core en Mac OS con línea de comandos
###### tags: `bitcoin` `cli` `nodo` `node` `macos`

_En este tutorial podrás aprender a instalar Bitcoin Core compilandolo desde el código fuente con Línea de comandos en una computadora con Mac OS._

**Tabla de contenido**
[TOC]

## Autores
EntrePlanctonyBallenas. 
Twitter para correcciones, comentarios o sugerencias: [@entreplanctony1](https://twitter.com/Entreplanctony1)

El presente tutorial fue elaborado para el [Seminario socrático de Mastering Bitcoin](https://libreriadesatoshi.com/) a través de [@libreriadesatoshi](https://twitter.com/libdesatoshi).

En el siguiente enlace puedes encontrar la documentación de referencia:
[https://github.com/bitcoin/bitcoin/blob/master/doc/build-osx.md](https://github.com/bitcoin/bitcoin/blob/master/doc/build-osx.md)

## Requisitos mínimos
Los nodos completos de Bitcoin Core tienen ciertos requisitos. 
Si intentas ejecutar un nodo en un hardware viejo, puede funcionar, pero probablemente pasarás más tiempo lidiando con problemas. 
Si puedes cumplir los siguientes requisitos, tendrás un nodo fácil de usar.

- Hardware de escritorio o portátil que ejecute versiones recientes de Windows, MAC OS o Linux.

- **540 gigabytes** de espacio **libre en disco**, accesible a una velocidad mínima de lectura/escritura de 100 MB/s. 
:::info
Si no tienes espacio en el disco de tu computadora, es posible usar un disco externo para guardar la cadena de bloques.
:::
- Al menos **2 gigabytes** de memoria (**RAM**)

- Una conexión a **Internet** de banda ancha con velocidades de subida de **al menos 50 kilobytes por segundo**

- Una conexión ilimitada, una conexión con límites de subida elevados o una conexión que monitorees regularmente para asegurarte de que no excedes tus límites de subida. Es común que los nodos completos en conexiones de alta velocidad utilicen 200 gigabytes de subida o más al MES. _El uso de descarga es de alrededor de 20 gigabytes al mes, más alrededor de 540 gigabytes adicionales la primera vez que inicia un nodo._

- 6 horas al día en las que se puede dejar funcionando el nodo completo. (Puedes hacer otras cosas con tu computadora mientras ejecutas un nodo completo.) Más horas serían mejores, y aún mejor sería si puedes ejecutar tu nodo continuamente. Sin embargo puedes apagar o suspender tu computadora de acuerdo tus necesidades, pero ==considera que cada vez que la apagues se deberan sincronizar los ultimos bloques desde el momento en que la apagaste.==

## Preparación
:::info
Los comandos de esta guía deben ser ejecutados en una aplicación de Terminal. MacOS viene con una Terminal incorporada, para abrirla ejecuta la secuencia de teclas **⌘+barra_espaciadora** e intruduce la palabra **terminal**.
:::

Antes de comenzar asegurate de conectar tu laptop o PC con un ==cable de red== directamente hacia tu módem de internet y desactiva la wifi, para tener una conexión a internet lo más veloz y estable posible.

Asegurate de que tu computadora este conectada a la corriente eléctrica y en las preferencias de sistema abre el _"Ahorro de energía"_ para **activar la función** de _"Impedir que la computadora entre en repos automáticamente"_, el IBD toma varios días y cada vez que se suspende tu computadora se interrumpirá la descarga.

![mac-sleep](https://i.imgur.com/9tJ0I7o.jpg)

Toma en cuenta que necesitas 540GB de espacio para almacenar toda la cadena de bloques completa, por lo que si no tienes este espacio disponible en el disco duro de tu sistema operativo, deberás conectar un disco duro externo, pero asegurate que dicho disco pueda ser montado como ==lectura / escritura== (los discos creados con particiones NTFS solo pueden ser montados como "read only" en MAC OS).
:::success
**TIP:** Formatea tu disco externo como **Ex-FAT** para poder usarllo en Windows y Mac OS. 
:::

![Ex-FAT](https://i.imgur.com/XRDRZ7P.jpg)

### Herramientas de línea de comando Xcode

Las herramientas de Línea de comandos Xcode son una colección de herramientas para macOS, y deben estar instaladas para poder instalar Bitcoin Core desde el código fuente.

Corre el siguiente comando desde tu terminal:
```gherkin
xcode-select --install
```

Al correr el comando debería aparecer un aviso. Haz click en `instalar` para continuar la instalación.

### Administrador de paquetes Homebrew

Homebrew es un gestor de paquetes para macOS que permite instalar paquetes desde la línea de comandos de manera fácil.

Para instalar [_Homebrew_](https://brew.sh), corre: 
```gherkin
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Nota: Si tienes problemas al instalar Homebrew o al descargar paquetes, consulta la [página de solución de problemas](https://docs.brew.sh/Troubleshooting).

### Dependencias Requeridas

Hay que descargar las dependencias requeridas. Estas dependencias son paquetes necesarios para hacer la instalación.

Ver [dependencies.md](https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md) para una lista completa.

Corre el siguiente comando en tu terminal:
```unix
brew install automake libtool boost pkg-config libevent
```
### Clonar el repositorio

`git` ya debería estar instalado en tu equipo. Ahora con todas las dependencias instaladas, hay que clonar el repositorio de _Bitcoin Core_ en un directorio de nuestra computadora. Todos los scripts para compilar y los comandos van a ejecutarse desde el directorio que elijas.

Ejemplo, si quiero crear un directorio llamado codigo-fuente y accesar al contenido de dicho directorio para que ahi se guarde la copia de _bitcoin core_ los comandos serían:
```unix
mkdir /Users/${USER}/codigo-fuente
cd /Users/${USER}/codigo-fuente
```
Y para clonar el repositorio:
```unix
git clone https://github.com/bitcoin/bitcoin.git
```
Ahora necesitas moverte al nuevo directorio que ha creado el comando git debajo de la ruta que tu creaste:
```unix
cd bitcoin
```
### Dependencias de la interfaz gráfica

Si quieres instalar la interfaz gráfica de _Bitcoin Core_ debes instalar el paquete qt@5. Si no quieres usar la interfaz gráfica puedes saltar este paso:
```unix
brew install qt@5
```
La interfaz gráfica también puede codificar direcciones en código QR, para usar esta función necesitas instalar el este paquete:
```unix
brew install qrencode
```
### Conjunto de pruebas (Test suite)
Hay un conjunto de pruebas que es muy útil para probar el código cuando se es desarrollador. Para poder correr estas pruebas (**recomendado**) hay que instalar python.
```unix
brew install python
```
### Archivo DMG
Si deseas que _bitcoin core_ cree un archivo con extensión **.dmg** para instalación en algún otro equipo con MacOS sin necesidad de compilar desde el código fuente debes correr el siguiente comando (solo funciona si ya tienes instalado python):
```unix
pip3 install ds_store mac_alias
```
## Ensamblando Bitcoin Core
### Configuración
Existen muchas maneras de configurar Bitcoin Core según las opciones que quieras instalar, aquí solo las más comunes, ambas instalaran **bitcoind** y **bicoin-cli** pero podrás controlar si se instalan las otras funcionalidades:

#### Solo Wallet e interfaz gráfica:
El siguiente comando habilita el soporte GUI y deshabilita el soporte para wallets legacy. Si no instalaste _qt_ (arriba) te mandará error. Si _sqlite_ (instalado por default en macOS) no esta instalado, la funcionalidad de wallet será desactivada.
```unix
./autogen.sh
./configure --without-bdb --with-gui=yes
```
#### Sin wallet ni interfaz gráfica:
```unix
./autogen.sh
./configure --without-wallet --with-gui=no
```
## Compilar
Después de configurar estas listo para compilar, corre el siguiente comando (dependiendo del hardware de tu computadora este proceso puede tardar un par de horas, ++mira la siguiente sección++):
``` unix
make        
```
### Para acelerar la Compilación
:bulb: En lugar de simplemente ejecutar `make` Puedes usar la opción "-j N" para compilar usando varios nucleos de procesador de manera paralela y reducir el tiempo que va a tardar la compilación (pero consumira más recursos, tómalo en cuenta por si estas realizando otras actividades), consulta el siguiente [enlace](https://www.blaess.fr/christophe/2012/01/14/parallelizing-compilations/) para tener más información al respecto, aquí un ejemplo:
``` unix
make -j 4       
```
### Para el conjunto de pruebas funcionales
Para poder correr las pruebas funcionales (ver Test Suite) si instalaste python.
```unix
make check
```
### Crear archivo DMG
Si quieres crear un archivo de instalación con extensión `.dmg` para que no tengas que volver a compilar si quieres instalar en otro equipo similar. Este DMG se creará con todas las opciones de configuración que escogiste previamente.
```unix
make deploy
```
## Ejecutar Bitcoin Core
El archivo ejecutable de Bitcoin Core se habrá creado en la _ruta relativa_ **`./src/bitcoind`**. Este archivo es el **daemon** (proceso en segundo plano) que necesitas correr si solo vas a utilizar la línea de comandos de _bitcoin core_.

El archivo **`./src/qt/bitcoin-cli`** es el ejecutable que utilizarás para interactuar con _Bitcoin Core_ y debes ejecutarlo solo después de haber ejecutado **`./src/bitcoind`**, de otra forma, el archivo mandará un error.

Y si compilaste también la interfaz gráfica la ruta será **`./src/qt/bitcoin-qt`**.

Siguiendo el ejemplo de la seccion _"Clonar el repositorio"_ que expusimos arriba, la _ruta absoluta_ de ambos archivos sería:
```unix
/Users/${USER}/codigo-fuente/bitcoin/src/bitcoind
/Users/${USER}/codigo-fuente/bitcoin/src/bitcoin-cli
/Users/${USER}/codigo-fuente/bitcoin/src/bitcoin-qt
```
:::warning
NOTA: no puedes ejecutar **`./src/qt/bitcoin-qt`** si **`./src/bitcoind`** ya está corriendo en el mismo equipo, para ejecutar cualquiera de los 2 comandos, no debe haber ninguna instancia de _Bitcoin Core_ activa en tu equipo.
:::
La primera vez que ejecutes el archivo bitcoind o bitcoin-qt, va a comenzar el proceso de IBD que puede tardar días (si quieres más información sobre este proceso y sus implicaciones consulta este [enlace](https://hackmd.io/kX3hKkP1ST-wPYmqv9AdEA#La-descarga-inicial-de-bloques-IBD)).

## Ubicación de los archivos de configuración
Por default, los archivos de **"wallets"** y **"la cadena de bloques"** se van a guardar en este directorio:
```unix
/Users/${USER}/Library/Application Support/Bitcoin/
```

Puedes monitorear el proceso de IBD y cualquier actividad que realiza el software de _Bitcoin Core_ en el archivo`debug.log`:
```unix
tail -f $HOME/Library/Application\ Support/Bitcoin/debug.log
```
## Antes de ejecutar Bitcoin Core por primera vez
:::success
:bulb: Existe un archivo de configuración llamado `bitcoin.conf` este archivo no se crea por default y a continuación puedes ver como crearlo y que es lo más recomendado que puedes hacer con el antes de inicializar tu nodo y comenzar la [IBD](https://hackmd.io/kX3hKkP1ST-wPYmqv9AdEA#La-descarga-inicial-de-bloques-IBD)
:::

### Si quieres cambiar el directorio de almacenamiento de default
Es posible que el equipo que estás usando no tenga el espacio suficiente en el directorio de default para almacenar toda la cadena de bloques (++500gb aprox++), pero puedes indicarle a _Bitcoin Core_ que la descargue en otro directorio o disco de tu computadora.

Si este es tu caso, antes de correr bitcoind, puedes crear un archivo vacío de configuración en la siguiente ubicación:
```unix
mkdir -p "/Users/${USER}/Library/Application Support/Bitcoin"
touch "/Users/${USER}/Library/Application Support/Bitcoin/bitcoin.conf"
chmod 600 "/Users/${USER}/Library/Application Support/Bitcoin/bitcoin.conf"
```
Con el editor de texto de tu preferencia debes modificar el archivo **bitcoin.conf** que acabas de crear para que contenga la siguiente linea:
```java
datadir=/Volumes/Mi_disco/bitcoin
```
Del ejemplo anterior tenemos:
**datadir** es una instrucción con la que le decimos al software que aquí guarde toda la información de nuestro nodo, bloques, índices, carteras, etc. 
**Volumes** es la ruta donde se montan los discos externos a tu sistema operativo (disco externo, pendrive, disco de red).
**Mi_disco** es el nombre de dispositivo de mi disco duro duro, el tuyo será otro que tu hayas definido o el nombre que tiene del fabricante. Es importante que macOS pueda leer y escribir en este disco (++mira arriba en la sección preparación de este tutorial++)
**bitcoin** es un directorio que yo cree para separar el contenido del nodo de bitcoin, de todo lo demás que hay en mi disco.

### Indexar las transacciones
La otra configruación básica del archivo `bitcoin.conf` se hace agregando la siguiente línea al archivo:
```java
txindex=1
```
Esta variable permite que el nodo cree un índice de todas las transacciones dentro de los bloques descargados y se usa para poder consultar cualquier transacción dentro de la base de datos de bitcoin.
### Habilitar la Testnet
Si vas a utilizar la red de pruebas de Bitcoin, por favor consulta: [Cómo correr un nodo en modo testnet](/3J6dkl0wTCOWJXuLB_kJyA)

## Ejecutar Bitcoin Core en segundo plano.
Si ejecutas **`bitcoind`** sin argumentos, no puedes cerrar la terminal en la que lo estes ejecutando, de otra forma el programa se cerrará junto con la ventana de terminal que estas utilizando, por este motivo si deseas poder cerrar la terminal sin parar la ejecución debes correr el comando de la siguiente manera desde una terminal (en este ejemplo usamos la ruta absoluta del archivo pero tu puedes usar la ruta relativa):
```unix
/Users/${USER}/codigo-fuente/bitcoin/src/bitcoind -daemon
```
Y para detener el programa de **`bitcoind`** puedes hacerlo usando:
```unix
/Users/${USER}/codigo-fuente/bitcoin/src/bitcoin-cli stop
```
De igual forma, si decides ejecutar desde una terminal el comando para lanzar la interfaz gráfica no podrás cerrar la terminal o _Bitcoin Core_ también se cerrará. Para abrir la interfaz gráfica desde línea de comando y que puedas cerrar la terminal:
```unix
/Users/${USER}/codigo-fuente/bitcoin/src/bitcoin-qt &
```