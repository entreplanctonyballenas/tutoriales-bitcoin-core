# ¿Cómo crear una dirección multifirma con Bitcoin Core?
###### tags: `bitcoin` `cli` `tx`

_Lo que necesitas saber para crear, fondear y gastar una dirección multifirma usando tu nodo de *Bitcoin Core*_

**Tabla de contenido**
[TOC]

## Autor
EntrePlanctonyBallenas. 
Twitter para correcciones, comentarios o sugerencias: [@entreplanctony1](https://twitter.com/Entreplanctony1)

El presente tutorial fue elaborado para el [Seminario socrático de Mastering Bitcoin](https://libreriadesatoshi.com/) a través de [@libreriadesatoshi](https://twitter.com/libdesatoshi).

En el siguiente enlace puedes encontrar la documentación de referencia:
[Sending a Transaction with a Multisig](https://github.com/BlockchainCommons/Learning-Bitcoin-from-the-Command-Line/blob/master/06_1_Sending_a_Transaction_to_a_Multisig.md)
:::success
:bulb:**¿Tienes un nodo de bitcoin?**
En este tutorial explicamos como usar la linea de comando de bitcoin core (bitcoin-cli), por lo que debes tener un nodo de bitcoin, estos pasos sirven igual para testnet o mainnet. Si no has instalado un nodo o no sabes como usar la testnet puedes hacerlo consultando los [Tutoriales sobre Nodos de Bitcoin](/jprsYssCTvuunqevvvzkZw).

NOTA: Los ejemplos de este tutorial se corren en testnet. También se asume que tienes una wallet inicializada y cargada. Si no sabes cómo crear una wallet puedes ver el tutorial para [Configurar una wallet en Bitcoin Core](/EvZ1uVQZTSiuZ9HCbp7eWQ)
:::

## ¿Qué es una transacción multifirma?
*Es una transacción que permite que múltiples llaves privadas habiliten el gasto de un saldo.*

En una transacción típica de `P2PKH` ó `SegWit`, los _bitcoins_ se envían a una dirección basada en una clave pública, lo que a su vez significa que se requiere la clave privada relacionada para desbloquear la transacción, permitiendo reutilizar los fondos. Pero, ¿qué pasaría si pudiera bloquear una transacción con múltiples llaves privadas? Esto permitiría enviar fondos a un grupo de personas, donde todas esas personas tienen que aceptar reutilizar los fondos.

## Crear una dirección multifirma
*Las direcciones multifirma (o multisig) son creadas usando las llaves públicas de múltiples wallets.*

Para crear una dirección multifirma es necesario definir 2 variables:
- **n** que será el número total de _**llaves públicas que crearán**_ la dirección multisig.
- **m** que será el número total de _**firmas requeridas**_ del total de **n** llaves que se van a requerir _en un futuro_ para gastar el saldo almacenado en la dirección multisig.

:::info
Para este tutorial vamos a crear un ejemplo dónde _**n=3**_ y _**m=2**_, es decir que para gastar los fondos bloqueados en nuestra dirección multisg, se requerián sólo 2 de 3 firmas totales.
:::
### Obtener las llaves públicas para crear la multisig

Con el objetivo de aumentar la privacidad siempre será recomendable crear una nueva dirección y de esta nueva dirección obtener las llaves públicas que vamos a utilizar.

++Los siguientes comandos deben correrse en distintos nodos cada uno con una wallet inicializada y trabajando bajo la misma red, o en su defecto deben correrse en una wallet que permita extraer la llave pública de una nueva dirección de _bitcoin_.++

Crea una nueva dirección:
`bitcoin-cli getnewaddress`
Obtén la llave pública que corresponde a la dirección que acabas de crear:
`bitcoin-cli getaddressinfo dirección`

Ejemplo para 3 nodos distintos (solo se inidca el campo `pubkey` para abreviar).

*Nodo1 - wallet: prueba1*
``` gherkin
bitcoin-cli getnewaddress
tb1qn9dzxae5q8qf7uqa8746687q23fus7plqk55g9
bitcoin-cli getaddressinfo tb1qn9dzxae5q8qf7uqa8746687q23fus7plqk55g9
{
...
"pubkey": "03789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd51",
...
]
}
```
*Nodo2 - wallet: prueba2*
``` gherkin
bitcoin-cli getnewaddress
tb1qvfyre6axx43mxmjduf7kunkt66umxd8lsc2jwv
bitcoin-cli getaddressinfo tb1qvfyre6axx43mxmjduf7kunkt66umxd8lsc2jwv
{
...
"pubkey": "026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb",
...
]
}
```
*Nodo3 - wallet: prueba3*
``` gherkin
bitcoin-cli getnewaddress
tb1qfgzy3uk7g9284dmgqlw87gfrz6edggaek9v33g
bitcoin-cli getaddressinfo tb1qfgzy3uk7g9284dmgqlw87gfrz6edggaek9v33g
{
...
"pubkey": "037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea150",
...
]
}
```
### El comando createmultisig
En el software de *Bitcoin Core* el comando para crear una dirección multifirma se llama `createmulstsig` y el formato para el mismo fue el siguiente:

``` gherkin
bitcoin-cli createmultisig nrequired keys="[\"llave-1\",\"llave-2\", \"llave-3\",...,\"llave-n\"]" "Tipo_de_direccion"
```
Podemos explicar la sintaxis del comando **createmultisig** de la siguiente manera:
- "**nrequired**" es un valor de número entero que representa el valor **m** que es el número total de _**firmas requeridas**_ del total de **n** llaves que se van a requerir _en un futuro_ para gastar el saldo almacenado en la dirección multisig. 
- "**keys=**" indica que los valores a continuación son un arreglo JSON que define las llaves públicas utilizadas para crear la dirección multifirma.
- "**llave-n**" son el total de llaves que van a usarse para bloquear el saldo dentro de la dirección.
- "**Tipo_de_direccion**" es un valor de texto que puede ser una de las siguientes 3 opciones: "**legacy**", "**p2sh-segwit**" y "**bech32**". 
- Los simbolos **[ ]** representan el inicio y el final del arreglo JSON que define cuáles son las llaves públicas utilizadas.
- Las comillas **" "** indican que estas pasandole un valor de texto al comando.
- Los símbolos \ solo sirven para poder pasar las comillas dentro del arreglo JSON.

Con estas reglas de sintaxis para este comando es suficiente para crear nuestra dirección y la misma quedaría de la siguiente manera, con el resultado del comando:

``` gherkin=
bitcoin-cli createmultisig 2 "[\"03789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd51\",  \"026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb\",  \"037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea150\"]" "p2sh-segwit"
{
  "address": "2N5EN6faB92fg5cBTYXxzyWuL8d574HGGoJ",
  "redeemScript": "522103789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd5121026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb21037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea15053ae",
  "descriptor": "sh(wsh(multi(2,03789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd51,026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb,037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea150)))#6eu2t9wt"
}
```
Y podemos interpretar el resultado del comando de la siguiente forma:

- **address** es lo que se entregará a las personas que desean enviar fondos. Notarás que tiene un nuevo prefijo de **2**, que es una dirección P2SH-SegWit. Eso es porque, createmultisig en realidad está creando un tipo de dirección totalmente nuevo llamado dirección P2SH. Funciona exactamente como una dirección P2PKH estándar para enviar fondos, pero dado que esta ha sido construida para requerir múltiples direcciones, deberá usar más trabajo para gastarlas.

:::info
🔗 TESTNET vs MAINNET: En *testnet*, el prefijo para las direcciones **P2SH** es **2**, mientras que en *mainnet*, es **3**.
:::
- **redeemScript** es lo que necesita para gastar los fondos (junto con las claves privadas para "**m**" de las "**n**" direcciones).
- **descriptor** es la descripción estandarizada de una dirección y proporciona una forma de importar esta dirección a otra máquina, utilizando el RPC *importmulti*.

### Guarda la información
Para que _en un futuro_ puedas gastar el _bitcoin_ que quedo bloqueado dentro de la dirección multi firma debes guardar toda la salida del comando **createmultisig** (y también guarda la wallet a la que pertenece cada dirección).

Puedes recrear la salida del comando, pero requieres introducir las llaves privadas en el mismo orden que lo hiciste la primera vez, cualquier cambio en el orden producirá una dirección distinta a la original.

:::danger
:floppy_disk: Debido a que la información de una dirección multifirma no esta asociada a una sola llave privada, *Bitcoin Core* no tiene la habilidad de guardar en tu wallet la información que generaste con la salida del comando _**createmultisig**_, por lo que debes guardar esta información de manera manual.
:::
### Depositar a la dirección multifirma
A pesar de que la dirección creada con **createmultisig** es una **P2SH** se utiliza exactamente igual que cualquier dirección **P2PKH** y se puede enviar _bitcoin_ o _test-coins_ de la misma manera que a cualquier otra dirección.

## Gastar los fondos de una dirección multifirma
Los pasos para "_gastar_" los fondos que estan bloqueados en una dirección multifirma son más complejos que para una transacción normal, esto es en parte porque la información con la que se construyó la dirección no reside en una sola wallet. Primero hay que empezar desde el nodo que va a construir la transacción que "_gastará_" los fondos bloqueados.

### Importar la dirección multifirma
En el nodo en el cual vas a gastar el bitcoin bloqueado en la transacción multisig debes primero avisarle al nodo que vas a utilizar una dirección que no tiene listada en su wallet, para ello hay que importar la dirección utilizando la información del campo **descriptor** y el comando `importmulti`. El comando también requiere un arreglo de *JSON* y la sintaxis sería:

```gherkin
bitcoin-cli importmulti '[{"desc": "descriptor-dirección-multisig", "timestamp": "now", "watchonly": true}]'
```
- **desc** es para indicar que la cadena frente a los dos puntos **:** es el **`descriptor`** de la transacción a ser importada.
- **timestamp** es el sello de tiempo de la llave más vieja que se utilizó para crear la dirección, si se especifica un valor el nodo va a re-escannear la cadena de bloques hasta el punto señalado para conocer las transacciones que pertencen a esta dirección.
- **now** se utiliza para evitar el escaneo de la cadena de bloques, para llaves públicas que se sabe que nunca han sido utilizadas.
- **watchonly** Establece si solo se deben observar las salidas encontradas.

Entonces para el ejemplo mencionado anteriormente el comando y resultado serían:

```gherkin=
bitcoin-cli importmulti '[{"desc": "sh(wsh(multi(2,03789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd51,026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb,037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea150)))#6eu2t9wt", "timestamp": "now", "watchonly": true}]'
[
  {
    "success": true
  }
]
```
**`"success": true`** quiere decir que el comando no arrojó errores.

### Visualizar el balance de la dirección multifirma
Ya que la dirección fue importada a tu wallet puedes visualizar el saldo igual que una dirección **p2pkh**
```gherkin=
bitcoin-cli listunspent
[
  {
    "txid": "9e78a9e84103b99517dbc5f6a6371450ae9dd53094a4c2b7e7fec2824f05801c",
    "vout": 0,
    "address": "2N5EN6faB92fg5cBTYXxzyWuL8d574HGGoJ",
    "label": "",
    "redeemScript": "0020927b1f27643d85a185047f89c8b4bb700a83955dcd71a4c90b5dc22954b87449",
    "witnessScript": "522103789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd5121026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb21037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea15053ae",
    "scriptPubKey": "a9148377dd0506970bde7594e79e1cd6226aeb37922487",
    "amount": 0.00000889,
    "confirmations": 28,
    "spendable": false,
    "solvable": true,
    "desc": "sh(wsh(multi(2,[995a2377]03789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd51,[62483ceb]026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb,[4a0448f2]037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea150)))#ht8xhg04",
    "safe": true
  }
]
```
:::warning
No confundir el campo **redeemScript** en la salida de *listunspent* ya que en esta salida el verdadero redeemScript aparece listado como **witnessScript**
:::
### Define los valores que necesitas para construir la transacción

Para construir esta transacción necesitamos saber la dirección de depósito así como la cantidad a depositar, además debemos extraer los siguientes valores de la transacción en la que se hizo el fondeo a la dirección multifirma. Puedes consultar estos datos (excepto redeemScript que ya explicamos antes) usando el comando **getrawtransaction** que guardaremos en variables de una terminal de línea de comandos para abreviar las salidas:
```gherkin=
utxo_txid="9e78a9e84103b99517dbc5f6a6371450ae9dd53094a4c2b7e7fec2824f05801c"
utxo_vout=0
utxo_amount=0.00000889
redeem_script="522103789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd5121026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb21037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea15053ae"
utxo_spk="a9148377dd0506970bde7594e79e1cd6226aeb37922487"
receptor="tb1qqph0266fvykngkul8saumv0ay95zsudk3a9462"
amount=0.00000600
```
:::spoiler *utxo_spk*
Es el valor **hex** de **scriptPubKey** en el arreglo de **vout** de la transacción a gastar.
:::

Ahora puedes crear la transacción con todas las variables y almacenar la salida en una variable llamada **rawtxhex**:
```gherkin=7
rawtxhex=$(bitcoin-cli createrawtransaction '''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout' } ]''' '''{ "'$receptor'": '$amount'}''')
echo $rawtxhex
02000000011c80054f82c2fee7b7c2a49430d59dae501437a6f6c5db1795b90341e8a9789e0000000000ffffffff015802000000000000160014006ef56b49612d345b9f3c3bcdb1fd21682871b600000000
```
### Firmar la transacción
Para poder desbloquear los fondos hay que cumplir las condiciones para gastar la dirección multifirma, en este ejemplo es necesario firmar la transacción con **2** de **3** firmas posibles.

Entonces dentro de cada nodo hay que [activar la wallet](https://hackmd.io/EvZ1uVQZTSiuZ9HCbp7eWQ?view#Activar-y-Desactivar-una-wallet) con la que se creo cada dirección y obtener su llave privada.

:::danger
:no_entry_sign: Accesar tus llaves privadas directamente desde una terminal o consola es peligroso y debería hacerse con mucho cuidado si se está usando *bitcoin real*.
:::
*Nodo1 - wallet: prueba1*
``` json
bitcoin-cli dumpprivkey tb1qn9dzxae5q8qf7uqa8746687q23fus7plqk55g9
cUzaxbkhxjtKT2PCuyEQttCoZHPUApjVKpgds6CkoQGLUqGDNJZ3
```
*Nodo3 - wallet: prueba3*
``` json
bitcoin-cli dumpprivkey tb1qfgzy3uk7g9284dmgqlw87gfrz6edggaek9v33g
cUEoeJnbiuPi1MYjrA8XiYBnthQsUk6XwgyMR6YpRd76KYgYDadM
```
Firmamos la transacción en hexadecimal que tenemos guardada en la variable **rawtxhex** usando el comando `signrawtransactionwithkey`, el comando usa la siguiente sintaxis:
```sass
bitcoin-cli -named signrawtransactionwithkey hexstring=tx-en-hex prevtxs='''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout', "scriptPubKey": "'$utxo_spk'", "redeemScript": "'$redeem_script'", "amount": '$utxo_amount'} ]''' privkeys='["Lista_de_llaves_privadas"]'`
```
- **-named** es un argumento que podemos usar en cualquier comando de bitcoin-cli para indicar el nombre de los campos que el comando requiere, se usa si vas a cambiar el orden de los campos requeridos por el comando.
- **hexstring** es la salida de nuestra nueva transacción de gasto codificada en hexadecimal.
- **prevtxs** es un arreglo de *JSON* que define la transacción de entrada, estos campos son necesarios ya que el comando requiere esta información para saber si el *redeem_script* es válido.
- **privkeys** es una lista de las llaves privadas que van a firmar la transacción.

En este ejemplo asumimos que estamos en el nodo1 por lo que solo firmamos con la ++primera llave privada++:
``` json
bitcoin-cli -named signrawtransactionwithkey hexstring=$rawtxhex prevtxs='''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout', "scriptPubKey": "'$utxo_spk'", "redeemScript": "'$redeem_script'", "amount": '$utxo_amount'} ]''' privkeys='["cUzaxbkhxjtKT2PCuyEQttCoZHPUApjVKpgds6CkoQGLUqGDNJZ3"]'
{
  "hex": "020000000001011c80054f82c2fee7b7c2a49430d59dae501437a6f6c5db1795b90341e8a9789e0000000023220020927b1f27643d85a185047f89c8b4bb700a83955dcd71a4c90b5dc22954b87449ffffffff015802000000000000160014006ef56b49612d345b9f3c3bcdb1fd21682871b604004730440220195bdc4f812960c2ed3d716632a109bc5742dc90e0fb819f9bde0e4789a2a15d02202fd7f04ce059cb550e9f81514944d757014e4cd8e6aaddcf88357a21ae6db59f010069522103789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd5121026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb21037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea15053ae00000000",
  "complete": false,
  "errors": [
    {
      "txid": "9e78a9e84103b99517dbc5f6a6371450ae9dd53094a4c2b7e7fec2824f05801c",
      "vout": 0,
      "witness": [
        "",
        "30440220195bdc4f812960c2ed3d716632a109bc5742dc90e0fb819f9bde0e4789a2a15d02202fd7f04ce059cb550e9f81514944d757014e4cd8e6aaddcf88357a21ae6db59f01",
        "",
        "522103789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd5121026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb21037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea15053ae"
      ],
      "scriptSig": "220020927b1f27643d85a185047f89c8b4bb700a83955dcd71a4c90b5dc22954b87449",
      "sequence": 4294967295,
      "error": "CHECK(MULTI)SIG failing with non-zero signature (possibly need more signatures)"
    }
  ]
}
``` 
:::warning
En este punto la salida arroja un mensaje de **error** esto es algo normal ya que el script de desbloqueo aún no esta completo.
:::
El campo **hex** de la salida resultante de `signrawtransactionwithkey` lo enviamos al segundo firmante (y lo guardamos en **sigrawtxhex1**), para nuestro ejemplo será el nodo3, el nodo3 ahora añade una nueva firma con su llave privada:
``` json
bitcoin-cli -named signrawtransactionwithkey hexstring=$sigrawtxhex1 prevtxs='''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout', "scriptPubKey": "'$utxo_spk'", "redeemScript": "'$redeem_script'", "amount": '$utxo_amount'} ]''' privkeys='["cUEoeJnbiuPi1MYjrA8XiYBnthQsUk6XwgyMR6YpRd76KYgYDadM"]'
{
  "hex": "020000000001011c80054f82c2fee7b7c2a49430d59dae501437a6f6c5db1795b90341e8a9789e0000000023220020927b1f27643d85a185047f89c8b4bb700a83955dcd71a4c90b5dc22954b87449ffffffff015802000000000000160014006ef56b49612d345b9f3c3bcdb1fd21682871b604004730440220195bdc4f812960c2ed3d716632a109bc5742dc90e0fb819f9bde0e4789a2a15d02202fd7f04ce059cb550e9f81514944d757014e4cd8e6aaddcf88357a21ae6db59f0147304402200fd5ad162d9b3c87951c3efeb8e2217c438f9244a766932fcff226b3256e3ce0022034db6a20a39c2f4cd26f2e3fab42cfb9680003c47ca9cfe8e8fc763345c1a9420169522103789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd5121026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb21037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea15053ae00000000",
  "complete": true
}

``` 
La salida ahora indica **"complete": true**, ahora el segundo firmante entonces va a obtener el campo **hex** de la salida resultante de `signrawtransactionwithkey` y esta es la transacción en hexadecimal que podemos enviar a la red (la guardamos en la variable **signedtx**).


### Enviar la transacción
Cualquiera que tenga la transacción en hexadecimal con las firmas requeridas puede publicar la transacción a la red.

Verificamos que los campos de la transacción sean válidos y enviamos la transacción a la red:
``` json
bitcoin-cli decoderawtransaction $signedtx
{
  "txid": "cf1e77641084a1df7ad88336fc0c9cb6e9d6e38750856203f18ec08c9398a9be",
  "hash": "aae2a31f89d49f8b87c8dd712942a2218c22145a4ee9b7e8b6d613462bfa759e",
  "version": 2,
  "size": 371,
  "vsize": 181,
  "weight": 722,
  "locktime": 0,
  "vin": [
    {
      "txid": "9e78a9e84103b99517dbc5f6a6371450ae9dd53094a4c2b7e7fec2824f05801c",
      "vout": 0,
      "scriptSig": {
        "asm": "0020927b1f27643d85a185047f89c8b4bb700a83955dcd71a4c90b5dc22954b87449",
        "hex": "220020927b1f27643d85a185047f89c8b4bb700a83955dcd71a4c90b5dc22954b87449"
      },
      "txinwitness": [
        "",
        "30440220195bdc4f812960c2ed3d716632a109bc5742dc90e0fb819f9bde0e4789a2a15d02202fd7f04ce059cb550e9f81514944d757014e4cd8e6aaddcf88357a21ae6db59f01",
        "304402200fd5ad162d9b3c87951c3efeb8e2217c438f9244a766932fcff226b3256e3ce0022034db6a20a39c2f4cd26f2e3fab42cfb9680003c47ca9cfe8e8fc763345c1a94201",
        "522103789e08453a347f882eeb40351021f8c7e50d13b091c31f5af98d0f705831dd5121026a41e38659a849b1f465b76305422bcb392ab4a941d828563217e369c587e0cb21037b215b4f61201b61851444dfb72fce522e13f485abaa19f5d7e24dbca29ea15053ae"
      ],
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.00000600,
      "n": 0,
      "scriptPubKey": {
        "asm": "0 006ef56b49612d345b9f3c3bcdb1fd21682871b6",
        "hex": "0014006ef56b49612d345b9f3c3bcdb1fd21682871b6",
        "address": "tb1qqph0266fvykngkul8saumv0ay95zsudk3a9462",
        "type": "witness_v0_keyhash"
      }
    }
  ]
}

bitcoin-cli sendrawtransaction $signedtx
cf1e77641084a1df7ad88336fc0c9cb6e9d6e38750856203f18ec08c9398a9be
```
En este ejemplo se envió un total de 600sats y se pagaron 289sats de fee.