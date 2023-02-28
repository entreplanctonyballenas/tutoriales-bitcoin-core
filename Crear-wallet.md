# Configurar una wallet en Bitcoin Core
###### tags: `bitcoin` `cli` `wallet`

_Ahora que ya tienes un nodo de bitcoin vas a aprender a usarlo para crear carteras_

**Tabla de contenido**
[TOC]

## Autor
EntrePlanctonyBallenas. 
Twitter para correcciones, comentarios o sugerencias: [@entreplanctony1](https://twitter.com/Entreplanctony1)

El presente tutorial fue elaborado para el [Seminario socrático de Mastering Bitcoin](https://libreriadesatoshi.com/) a través de [@libreriadesatoshi](https://twitter.com/libdesatoshi).

En el siguiente enlace puedes encontrar la documentación de referencia:
[Setting Up Your Wallet](https://github.com/BlockchainCommons/Learning-Bitcoin-from-the-Command-Line/blob/master/03_3_Setting_Up_Your_Wallet.md)

:::success
:bulb: **¿Tienes un nodo de bitcoin?**
En este tutorial explicamos como usar la linea de comando de bitcoin core (bitcoin-cli), por lo que debes tener un nodo de bitcoin, también se asume que sabes cómo usar el modo ++testnet++, si no has instalado un nodo o no sabes como usar la testnet puedes hacerlo consultando los [Tutoriales sobre Nodos de Bitcoin](/jprsYssCTvuunqevvvzkZw).

NOTA: Los ejemplos de este tutorial se corren en un nodo inicializado por configuracion (bitcoin.conf) en modo testnet.
:::

## ¿Qué es una wallet de Bitcoin? 
Una **wallet** (_billetera en español_) de Bitcoin es el equivalente digital de una billetera física en la red Bitcoin. Almacena información sobre la cantidad de bitcoins que te pertenecen y en que direcciones se encuentra, así como las formas en que puedes gastarlo. Gastar dinero físico es intuitivo, pero para gastar bitcoins, los usuarios deben proporcionar la clave privada correcta asociada a una clave pública. La información del par de claves se guarda en un archivo de la wallet, además de los datos sobre preferencias y transacciones. En su mayor parte, no tendrás que preocuparte por esa clave privada: el software la usará cuando sea necesario. Sin embargo, esto hace que el archivo sea extremadamente importante: si lo pierdes, pierdes tus claves privadas, y si pierdes tus claves privadas, ¡pierdes tus fondos! 

:::danger
:no_entry: Las wallets que puedes crear con _Bitcoin Core_ no usan frases semillas, en su lugar puedes usar un password para encriptarlas, por lo tanto aunque las wallets de _Bitcoin Core_ son funcionales y muy útiles para fines educacionales, ++no son recomendables para guardar bitcoin real++ en ellas.
:::
## Crear una wallet simple
Al inicializar el software de _Bitcoin Core_ no hay ninguna wallet cargada, por lo que debes crear una para poder interactuar con la red. 
El comando para crear wallets es: 
``` python
bitcoin-cli createwallet "nombre-de-tu-wallet"
```
Hay muchas opciones para crear wallets, por ahora solo crearemos una simple. Pero puedes explorer las opciones con el comando **bitcoin-cli help createwallet**

En el siguiente ejemplo estamos creando una wallet llamada ++"prueba1"++ yel resultado será el siguiente, si en el campo de "warning" no hay ningún mensaje de error, quiere decir que se creo correctamente
``` json
bitcoin-cli createwallet "prueba1"
{
  "name": "prueba1",
  "warning": ""
}
```
Con una wallet creada, ahora debemos crear una dirección pública que será donde recibiremos bitcoin, el comando es el siguiente:
``` python
bitcoin-cli getnewaddress "etiqueta" "Tipo_de_direccion"
```
Donde:
- "**etiqueta**" es un valor de texto opcional definido por nosotros para diferenciar esta dirección dentro de nuestra wallet.
- "**Tipo_de_direccion**" es un valor de texto que puede ser una de las siguientes 3 opciones: "**legacy**", "**p2sh-segwit**" y "**bech32**".

El siguiente ejemplo crea una dirección llamada "**faucet**" y que será de tipo "**bech32**", la cadena resultante será la dirección de depósito:
``` json
bitcoin-cli getnewaddress "faucet" "bech32"
tb1qnnvqdmuhkn77lm0jds3gvfhc5yqgjgfx6qukt8
```
Tenga en cuenta que esta dirección comienza con las letras "**tb1**" porque es una dirección **bech32** de **++testnet++**, si hubiéramos especificado "**legacy**" sería una "**m**" ( o, a veces, una "**n**" ) ó sería un "**2**" para una dirección "**P2SH**".
:::info
🔗 TESTNET vs MAINNET: La dirección principal equivalente comenzaría con un "**1**" ( para **Legacy** ), "**3**" ( para **P2SH** ) o "**bc1**" ( para **Bech32** ).
:::

## Enviar Testnet coins a tu wallet
Ahora que tienes una dirección de bitcoin, puedes ir a alguna _faucet_ que regale testnet coins de bitcoin como [https://bitcoinfaucet.uo1.net/](https://bitcoinfaucet.uo1.net/)

Solo copia la dirección testnet que acabas de crear, introducela en "_bitcoin address_" y da click en "_Send testnet bitcoins_"

## Consulta tu balance

Después de unos minutos, tu transacción deberá estar confirmada, por lo que ahora querrás saber, cómo confirmas que recibiste el saldo, los siguientes 3 comandos te dirán tu balance total, el que hay un tu cartera y las **txid** (++utxo++) que tienes para gastar respectivamente:
```json=
bitcoin-cli getbalance
0.00001000

bitcoin-cli getwalletinfo
{
  "walletname": "prueba1",
  "walletversion": 169900,
  "format": "bdb",
  "balance": 0.00001000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 1,
  "keypoololdest": 1677021603,
  "keypoolsize": 999,
  "hdseedid": "e82e328cb77dc2bba1fea97525a98a735361d3bb",
  "keypoolsize_hd_internal": 1000,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": false
}

bitcoin-cli listunspent
[
  {
    "txid": "fc2067320c7811079a826c80785f509769c20537a322457be0489c33662ddf4d",
    "vout": 0,
    "address": "tb1qnnvqdmuhkn77lm0jds3gvfhc5yqgjgfx6qukt8",
    "label": "faucet",
    "scriptPubKey": "00149cd806ef97b4fdefedf26c228626f8a100892126",
    "amount": 0.00001000,
    "confirmations": 1,
    "spendable": true,
    "solvable": true,
    "desc": "wpkh([cef220f5/0'/0'/0']02e2f07438f06cdb9ad240aff30cfa699e11d87732c6431d23ad91316bf14ed723)#y4a0g4pw",
    "safe": true
  }
]
```
## Activar y Desactivar una wallet

Cada vez que reinicies el software de *Bitcoin Core* tendrás que cargar una wallet creada ya que por default ninguna wallet se carga al inicializar (a menos que lo hayas definido al crear la wallet, mira el *help* de *createwallet*).

Y cada vez que hayas creado una wallet, se crea un directorio con el nombre de la wallet en la ruta de archivos que hayas definido en la instalación de *Bitcoin Core* y ese directorio contiene la información de tu wallet y las llaves que esta administra, para visualizar este directorio usa:
```
bitcoin-cli listwalletdir
{
  "wallets": [
    {
      "name": "prueba1"
    },
    {
      "name": "prueba2"
    }
  ]
}
```
Una vez localizado el nombre de tu wallet para activarla usa el comando:
```
bitcoin-cli loadwallet nombre-de-wallet
```
Si quieres desactivar una wallet (para crear una nueva) usa el comando:
```
bitcoin-cli unloadwallet nombre-de-wallet
```
Para listar el nombre de la wallet que tienes activa actualmente:
```
bitcoin-cli listwallets
```

