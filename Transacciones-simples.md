# Tutorial: ¿Cómo crear una transacción usando bitcoin-cli para hacer pruebas?
###### tags: `bitcoin` `transactions`
_En este tutorial aprendemos a usar la línea de comandos del cliente de "Bitcoin Core" para que con tu propio nodo puedas hacer transacciones de manera manual y así entender más claramente cómo se ejecutan las transacciones en el sistema de Bitcoin._

[TOC]

**Autor:** Entre Plancton y Ballenas. Twitter para correcciones, comentarios o sugerencias: [@entreplanctony1](https://twitter.com/Entreplanctony1).

El presente tutorial se realizó con el aprendizaje social del [Seminario socrático de Mastering Bitcoin](https://libreriadesatoshi.com/) a través de [@libreriadesatoshi](https://twitter.com/libdesatoshi).

En el siguiente enlace puedes encontrar la documentación de referencia:
[BitcoinDeveloper Transaction Tutorial](https://developer.bitcoin.org/examples/transactions.html)
:::success
:bulb: **¿Tienes un nodo de bitcoin?**
En este tutorial explicamos como usar la linea de comando de bitcoin core (bitcoin-cli), por lo que debes tener un nodo de bitcoin, estos pasos sirven igual para testnet o mainnet. Si no has instalado un nodo o no sabes como usar la testnet puedes hacerlo consultando los [Tutoriales sobre Nodos de Bitcoin](/jprsYssCTvuunqevvvzkZw).

NOTA: Los ejemplos de este tutorial se corren en testnet. También se asume que tienes una wallet inicializada y cargada. Si no sabes cómo crear una wallet puedes ver el tutorial para [Configurar una wallet en Bitcoin Core](/EvZ1uVQZTSiuZ9HCbp7eWQ)
:::

## Video Tutorial
:eye: Si lo prefieres, en el siguiente enlace puedes ver el [Tutorial: Cómo crear transacciones en la testnet de Bitcoin.](https://youtu.be/Zejek_nugWw)
## Ubica el balance que tienes dentro de tu wallet
El subcomando ***getbalance*** te muestra la cantidad total de satoshis disponibles para gastar en tu wallet, sumando todas los UTXO que tienes disponibles.

``` python
$ bitcoin-cli getbalance       
0.00008500
```
## ¿Cuánto vas a gastar?
Debes verificar la cantidad a gastar, para este ejemplo, quiero hacer un pago simple de **0.00008000** bitcoin. Por lo que debo identificar si hay alguna UTXO dentro de mi wallet en la que tenga el saldo suficiente para hacer este pago.

El subcomando **"listunspent"** nos despliega los UTXO que se tienen disponibles para gastar. 
``` json=
$ bitcoin-cli listunspent 
[
  {
    "txid": "b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6",
    "vout": 0,
    "address": "tb1qn0z6jwa0dvgf3my00f862wunjkrethm3qtjdsv",
    "label": "destino",
    "scriptPubKey": "00149bc5a93baf6b1098ec8f7a4fa53b93958795df71",
    "amount": 0.00004500,
    "confirmations": 2518,
    "spendable": true,
    "solvable": true,
    "desc": "wpkh([c062f9b4/0'/0'/3']024a89edfcfe9c294765c7d0128862cf5911977f8ab2e449807afa3973b0e12bba)#34zpfpsk",
    "safe": true
  },
  {
    "txid": "b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6",
    "vout": 1,
    "address": "tb1q76c6wv8v2k22682g9wdz73xxzm9pjg5cken3pu",
    "label": "p2sh-segwit",
    "scriptPubKey": "0014f6b1a730ec5594ad1d482b9a2f44c616ca192298",
    "amount": 0.00004000,
    "confirmations": 2518,
    "spendable": true,
    "solvable": true,
    "desc": "wpkh([c062f9b4/0'/0'/2']02f423b23f3ead98454eb19b9991c7b530aebb93a918aa78f63ed3ec7764f2b8d3)#wn6m9c0g",
    "safe": true
  }
]
```
De la salida resultante arriba, debemos identificar los siguientes campos para construir nuestra transacción:
- **"txid"**  es el identificador de nuestro UTXO [lineas 4 y 17]
- **"vout"**  es el índice que corresponde a la dirección que nos pertenece (la transacción pudo haber sido creada con más de un destinatario) [lineas 5 y 18]
- **"amount"**  es la cantidad disponible en cada UTXO [lineas 9 y 22]

En mi wallet tengo 2 UTXO la primera con **0.00004500** y la segunda con **0.00004000**, por lo que ninguna de las 2 tiene suficiente saldo para cubrir el pago que quiero hacer, sin embargo, la suma de ambas si puede cubrir la cantidad que necesito pagar **0.00008000**.

Ahora que conocemos las cantidades disponibles, podemos `declarar` las variables para las transacciones que necesitamos:
``` python=
$ utxo_txid=b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6
$ utxo1_vout=0
$ utxo1_monto=0.00004500
$ utxo2_vout=1
$ utxo2_monto=0.00004000
$ montopago=0.00008000
```
De la primera linea **utxo_txid** solo es el nombre que yo escogi para introducir un valor en la memoria de mi terminal (puedes darle a cada variable el nombre que tu quieras), el símbolo **=** indica el valor que va a tener la variable, y así sucesivamente en cada línea. En este caso ambos UTXO que tengo disponibles están declarados dentro del la misma transacción (**txid** en la salida de **listunspent**) por lo que sólo necesito declarar un txid dentro de mis variables y el resto de las variables solo son los montos que tiene cada UTXO y el monto a pagar.

En cualquier momento puedes verificar que tus variables tengan los valores correctos ejecutando el comando **"echo"** y añade el caracter **"$"** al nombre de tu variable.

``` shell=
$ echo $utxo_txid
b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6
$ echo $utxo1_vout
0
$ echo $utxo1_monto
0.00004500
$ echo $utxo2_vout
1
$ echo $utxo2_monto
0.00004000
$ echo $montopago
0.00008000
```




## Calcula los montos de salida de tu transacción

Debido a que los UTXO se deben "gastar" en su totalidad para poder ser utilizados, necesito consumir ambos UTXO que tengo en mi wallet para hacer el pago deseado, pero como la cantidad sumada no da la cantidad exacta para cubrir el pago, necesito calcular tanto mi "cambio" como el "fee" (la cuota) a pagar a la red para garantizar que mi transacción sea verificada por la red.

El cálculo de los fee es complejo, por lo que para este ejemplo estoy asumiendo que es algo simple y sólo quiero pagar una cuota de 0.00000300:

``` python
$ echo 0.00004500 + 0.00004000 - 0.00008000 - 0.00000300 | bc
0.00000200
$ cambio=0.00000200

```
- **echo** es un comando que despliega en la terminal lo que escribes frente a él.
- El caracter **|** (pipe) indica que a la salida resultante del comando de la izquierda se le va a aplicar el comando de la derecha. 
- **bc** solo es para indicar que quiero hacer una operacion con valores de numero flotante.
- **cambio=** Aqui guardo en memoria el valor obtenido.

En total necesito definir 2 montos de salida en mi transacción, una salida que contenga el valor definido en **montopago** y otra salida que será mi **cambio**, el **fee** de transacción aunque debemos siempre calcularlo para no cometer errores, no es necesario `declararlo`ya que en bitcoin el sobrante de tus operaciones de salida ya esta implicito que será tu pago a los mineros.

:::warning
:warning: Si no declaras en tu transacción un monto de salida para definir tu "cambio" el sistema asume que cualquier valor sobrante es lo que vas a pagar de **fee** a los mineros.
:::
Por último necesitamos las direcciones a las cuales vamos a depositar el monto de pago y el cambio (para este último estoy creando una nueva dirección dentro de mi wallet):

``` phyton
$ bitcoin-cli getnewaddress direcciondecambio
tb1qzrfj0w8ec7mjtyhtg4tcvy6zg53wh4335uu8x5
$ direcciondecambio= tb1qzrfj0w8ec7mjtyhtg4tcvy6zg53wh4335uu8x5
$ direcciondepago= tb1qpsjd66q4rwgfp6zju67wjyh6778ytj5ak200nt
```


## Crea tu transacción

El subcomando **createrawtransaction** es utilizado para crear las transacciones en crudo (en hexadecimal). Se debe introducir un arreglo JSON para listar los UTXOs que está gastando y otro para listar las salidas (destinatarios).

La estructura del comando podemos entenderla con el siguiente diagrama:

``` mermaid
classDiagram
ENTRADAS --> UTXO1
UTXO1 : txid
UTXO1 : vout
ENTRADAS --> UTXO2
UTXO2 : txid
UTXO2 : vout
ENTRADAS -- SALIDAS
SALIDAS --> PAGO
PAGO : direcciondepago
PAGO : montopago
SALIDAS --> CAMBIO
CAMBIO : direcciondecambio
CAMBIO : montocambio
```
Las entradas y las salidas se deben agrupar en un orden definido para que el comando las reconozca y deben contar con la siguiente sintaxis de objetos JSON, si usamos las variables que hemos definido previamente nos quedaría lo siguiente:

``` json=
% bitcoin-cli createrawtransaction 
''' [
     {
        "txid": "'$utxo_txid'","vout": '$utxo1_vout'
     }, 
     {
         "txid": "'$utxo_txid'","vout": '$utxo2_vout'
     }
] '''
''' [
     {
         "'$direcciondepago'": '$montopago'
     }, 
     {
         "'$direcciondecambio'": '$cambio'
     }
] '''
```
El comando **bitcoin-cli** utiliza un RPC JSON, por lo que la sintaxis del subcomando **createrawtransaction** podemos explicarla asi:
- Las 3 comillas **'''** y los simbolos **[ ]** en las lineas 2 y 9 representan el inicio y el final del arreglo JSON para las transacciones de entrada. Mientras que en las lineas 11 y 17 representan el inicio y el final del arreglo de salidas.
- Los corchetes **{ }** delimitan el inicio y final de cada uno de los valores de entrada y salida.
- La comas al final del corchete de cierre **},** solo indican que hay mas de un valor de entrada ó salida y no son necesarias cuando se tiene una sola entrada y una sola salida.
- Las comillas **" "** indican que estas pasandole un valor de texto al comando.

Con estas reglas de sintaxis para este comando es suficiente para definir nuestra transacción y la misma quedaría de la siguiente manera:

``` shell
$ bitcoin-cli createrawtransaction '''[{"txid": "b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6","vout": 0}, {"txid": "b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6","vout": 1}]''' '''[{"tb1qpsjd66q4rwgfp6zju67wjyh6778ytj5ak200nt": 0.00008000}, {"tb1qzrfj0w8ec7mjtyhtg4tcvy6zg53wh4335uu8x5": 0.00000200}]'''
```
Sin embargo, podemos también usar las variables que definimos anteriormente, ya que la idea es que tu puedas copiar y pegar estos comandos y experimentar con ellos con UTXOs que tengas en tu wallet. 

Para usar las variables cargadas en la memoria de nuestra terminal, necesitamos pasar otros caracteres para que el comando pueda leer lo que tenemos en nuestras variables, las comillas simples **' '** ejecutan lo que esta dentro de ellas y el **$** es para indicar que el texto frente a este símbolo es un apuntador a un valor que esta en memoria.

El siguiente comando crea la transacción usando nuestras variables y el valor hexadecimal resultante queda almacenado en una variable nueva llamada **txhex** :
``` shell
$ txhex=$(bitcoin-cli createrawtransaction '''[{"txid": "'$utxo_txid'","vout": '$utxo1_vout'}, {"txid": "'$utxo_txid'","vout": '$utxo2_vout'}]''' '''[{"'$direcciondepago'": '$montopago'}, {"'$direcciondecambio'": '$cambio'}]''')
$ echo $txhex
0200000002d69647239131124fce40f74432167679d28e6c1e372a71b1599e72c92cac67b40000000000ffffffffd69647239131124fce40f74432167679d28e6c1e372a71b1599e72c92cac67b40100000000ffffffff02401f0000000000001600140c24dd68151b9090e852e6bce912faf78e45ca9dc80000000000000016001410d327b8f9c7b72592eb45578613424522ebd63100000000
```
## Verifica y firma tu transacción

Usa el comando **decoderawtransaction** para verificar que todos los campos que acabas de construir con **createrawtransaction** estan correctos, notarás que el comando ha agregado varios campos por ti de manera automática.
```json=
$ bitcoin-cli decoderawtransaction $txhex
{
  "txid": "87bcc93f122a96e5b2c56de996f4efcd1f330eceff039146e329038c261afbc5",
  "hash": "87bcc93f122a96e5b2c56de996f4efcd1f330eceff039146e329038c261afbc5",
  "version": 2,
  "size": 154,
  "vsize": 154,
  "weight": 616,
  "locktime": 0,
  "vin": [
    {
      "txid": "b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967295
    },
    {
      "txid": "b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6",
      "vout": 1,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.00008000,
      "n": 0,
      "scriptPubKey": {
        "asm": "0 0c24dd68151b9090e852e6bce912faf78e45ca9d",
        "hex": "00140c24dd68151b9090e852e6bce912faf78e45ca9d",
        "address": "tb1qpsjd66q4rwgfp6zju67wjyh6778ytj5ak200nt",
        "type": "witness_v0_keyhash"
      }
    },
    {
      "value": 0.00000200,
      "n": 1,
      "scriptPubKey": {
        "asm": "0 10d327b8f9c7b72592eb45578613424522ebd631",
        "hex": "001410d327b8f9c7b72592eb45578613424522ebd631",
        "address": "tb1qzrfj0w8ec7mjtyhtg4tcvy6zg53wh4335uu8x5",
        "type": "witness_v0_keyhash"
      }
    }
  ]
}
```
:::warning
:warning: Verifica cuidadosamente que la sección de **vin** tenga los UTXO correctos, y que la sección de **vout** tenga las direcciones y los montos a pagar correctos, si cometes un error y gastas un UTXO equivocado o si depositas a una dirección incorrecta, y la transacción es aceptada por la red no hay forma de revertir los cambios en la cadena de bloques.

El comando **createrawtransaction** no calcula que los montos de entrada y salida sean correctos, solo valida que la sintaxis con la que la escribiste este correcta, por lo que aun podria ser rechazada tu transacción si cometiste errores de cálculo.
:::

Si en este punto todos los valores están correctos, entonces ahora puedes firmar la transacción con el comando **signrawtransaction**

``` json=
$ bitcoin-cli signrawtransactionwithwallet $txhex
{
  "hex": "02000000000102d69647239131124fce40f74432167679d28e6c1e372a71b1599e72c92cac67b40000000000ffffffffd69647239131124fce40f74432167679d28e6c1e372a71b1599e72c92cac67b40100000000ffffffff02401f0000000000001600140c24dd68151b9090e852e6bce912faf78e45ca9dc80000000000000016001410d327b8f9c7b72592eb45578613424522ebd63102473044022030d66c943170e5379d0c7a1e5c72074e8209909cbbace9ad3871ef167f670a660220678145bd2936ee098764394949a3feb29f1210368db75c87f644c733dd3a22d20121024a89edfcfe9c294765c7d0128862cf5911977f8ab2e449807afa3973b0e12bba0247304402205606745ac0af2ae456c8a3a6fc3ac27c1060068104b6b19e9598747bc6802fd60220725de7c9d9fb0bf7c33cfef774c8084bbf1429d3961da1fee8fe23ac84f14a5e012102f423b23f3ead98454eb19b9991c7b530aebb93a918aa78f63ed3ec7764f2b8d300000000",
  "complete": true
}
$ txfirmada= 02000000000102d69647239131124fce40f74432167679d28e6c1e372a71b1599e72c92cac67b40000000000ffffffffd69647239131124fce40f74432167679d28e6c1e372a71b1599e72c92cac67b40100000000ffffffff02401f0000000000001600140c24dd68151b9090e852e6bce912faf78e45ca9dc80000000000000016001410d327b8f9c7b72592eb45578613424522ebd63102473044022030d66c943170e5379d0c7a1e5c72074e8209909cbbace9ad3871ef167f670a660220678145bd2936ee098764394949a3feb29f1210368db75c87f644c733dd3a22d20121024a89edfcfe9c294765c7d0128862cf5911977f8ab2e449807afa3973b0e12bba0247304402205606745ac0af2ae456c8a3a6fc3ac27c1060068104b6b19e9598747bc6802fd60220725de7c9d9fb0bf7c33cfef774c8084bbf1429d3961da1fee8fe23ac84f14a5e012102f423b23f3ead98454eb19b9991c7b530aebb93a918aa78f63ed3ec7764f2b8d300000000
```
El comando devuelve un valor en hexadecimal llamado **"hex"** esta es tu transacción ya firmada con tu wallet. El valor **"complete"** si devuelve valor **true** significa que tu transacción no tiene errores (tu wallet si puede firmar la dirección del UTXO), si hay algún problema con tu transacción verás los mensajes de error aquí. El valor lo guardamos en una variable llamada **txfirmada**.

Puedes correr el comando **decoderawtransaction** y verás como ahora existen los campos de **"txinwitness"** dentro de tus **vin** , estos contienen las firmas de tu wallet.
``` shell
$ bitcoin-cli decoderawtransaction $txfirmada
```
:::info
:bulb: **Puedes firmar tu transacción sin conexión**
Como puedes ver hasta este punto la transacción no ha sido enviada a la red de bitcoin, por lo que el paso de crear y firmar la transacción pueden ejecutarse en una computadora disitnta o incluso sin conexión a internet para aumentar tu seguridad.
:::

## Envia tu transacción a la red

El ultimo paso es enviar la transacción a la red de **bitcoin**, para esto usamos el comando **sendrawtransaction**
``` shell
$ bitcoin-cli sendrawtransaction $txfirmada
c8493beae315f8c090aab8a348968c895a98534cd879496359561fb41ead7ea2
```
El comando devuelve un valor en hexadecimal que es el id de la transacción, el cual podemos verificar con **getrawtransaction**.
``` json=
% bitcoin-cli getrawtransaction c8493beae315f8c090aab8a348968c895a98534cd879496359561fb41ead7ea2 1
{
  "txid": "c8493beae315f8c090aab8a348968c895a98534cd879496359561fb41ead7ea2",
  "hash": "d2d217070fc053da1f92b7e10dae1f38d78156ea1a745eb64689cc12f7f78e12",
  "version": 2,
  "size": 370,
  "vsize": 208,
  "weight": 832,
  "locktime": 0,
  "vin": [
    {
      "txid": "b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "txinwitness": [
        "3044022030d66c943170e5379d0c7a1e5c72074e8209909cbbace9ad3871ef167f670a660220678145bd2936ee098764394949a3feb29f1210368db75c87f644c733dd3a22d201",
        "024a89edfcfe9c294765c7d0128862cf5911977f8ab2e449807afa3973b0e12bba"
      ],
      "sequence": 4294967295
    },
    {
      "txid": "b467ac2cc9729e59b1712a371e6c8ed27976163244f740ce4f123191234796d6",
      "vout": 1,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "txinwitness": [
        "304402205606745ac0af2ae456c8a3a6fc3ac27c1060068104b6b19e9598747bc6802fd60220725de7c9d9fb0bf7c33cfef774c8084bbf1429d3961da1fee8fe23ac84f14a5e01",
        "02f423b23f3ead98454eb19b9991c7b530aebb93a918aa78f63ed3ec7764f2b8d3"
      ],
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.00008000,
      "n": 0,
      "scriptPubKey": {
        "asm": "0 0c24dd68151b9090e852e6bce912faf78e45ca9d",
        "hex": "00140c24dd68151b9090e852e6bce912faf78e45ca9d",
        "address": "tb1qpsjd66q4rwgfp6zju67wjyh6778ytj5ak200nt",
        "type": "witness_v0_keyhash"
      }
    },
    {
      "value": 0.00000200,
      "n": 1,
      "scriptPubKey": {
        "asm": "0 10d327b8f9c7b72592eb45578613424522ebd631",
        "hex": "001410d327b8f9c7b72592eb45578613424522ebd631",
        "address": "tb1qzrfj0w8ec7mjtyhtg4tcvy6zg53wh4335uu8x5",
        "type": "witness_v0_keyhash"
      }
    }
  ],
  "hex": "02000000000102d69647239131124fce40f74432167679d28e6c1e372a71b1599e72c92cac67b40000000000ffffffffd69647239131124fce40f74432167679d28e6c1e372a71b1599e72c92cac67b40100000000ffffffff02401f0000000000001600140c24dd68151b9090e852e6bce912faf78e45ca9dc80000000000000016001410d327b8f9c7b72592eb45578613424522ebd63102473044022030d66c943170e5379d0c7a1e5c72074e8209909cbbace9ad3871ef167f670a660220678145bd2936ee098764394949a3feb29f1210368db75c87f644c733dd3a22d20121024a89edfcfe9c294765c7d0128862cf5911977f8ab2e449807afa3973b0e12bba0247304402205606745ac0af2ae456c8a3a6fc3ac27c1060068104b6b19e9598747bc6802fd60220725de7c9d9fb0bf7c33cfef774c8084bbf1429d3961da1fee8fe23ac84f14a5e012102f423b23f3ead98454eb19b9991c7b530aebb93a918aa78f63ed3ec7764f2b8d300000000",
  "blockhash": "000000000000002c8a723618df5b72e390df8d7f2d5caf8ea34762fd8ba71561",
  "confirmations": 1,
  "time": 1661204372,
  "blocktime": 1661204372
}
```
Al cabo de unos minutos si nuestros valores fueron correctos, podremos ver que la transacción ya esta confirmada (linea 62, número de confirmaciones) y en que bloque lo fue (linea 61 muestra el hash del bloque). También puedes visualizar la transacción en tu wallet con el comando **listtransactions**, y el saldo total en tu cartera deberá cambiar en **getbalance**.

:::success
:thumbsup:Felicidades, ya puedes usar tu nodo para hacer transacciones de prueba!
:::