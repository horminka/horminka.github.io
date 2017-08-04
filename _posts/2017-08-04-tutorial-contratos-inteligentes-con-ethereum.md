---
layout: post
title: "Tutorial Contratos Inteligentes con Ethereum"
date: 2017-08-04 10:26:04 -0500
ref: ethereum
lang: Español
categories: web
author: leonardo
image: http://i.imgur.com/wWQVbiu.png
---

## ¿Qué es Ethereum?

[Ethereum](https://es.wikipedia.org/wiki/Ethereum) es una plataforma de código abierto basada en
[Cadena de Bloques](https://es.wikipedia.org/wiki/Cadena_de_bloques) que
permite escribir y publicar [aplicaciones descentralizadas](https://ethereum.stackexchange.com/questions/383/what-is-a-dapp) usando [Contratos Inteligentes](https://en.wikipedia.org/wiki/Ethereum#Smart_contracts).

![Contrato Inteligente](http://i.imgur.com/wWQVbiu.png)

### Contratos Inteligentes

Los Contratos Inteligentes son piezas de software que
se escriben en una Cadena de Bloques y por lo tanto se ejecutan exactamente como
fueron programados sin ninguna posibilidad de censura, caída, fraude, o
interferencia de terceros.

Los Contratos Inteligentes pueden tomar decisiones, realizar transferencias
y leer o ejecutar otros contratos para facilitar intercambios de dinero, contenidos,
propiedad, acciones, o cualquier otro valor entre personas iguales de
manera segura, transparente y sin intermediarios.

En este tutorial se va a crear un contrato inteligente que lleva el
registro de votaciones en una elección con tres candidatos. Primero se
va a escribir y compilar el contrato paso por paso, y luego se hará
utilizando el framework [Truffle](http://truffleframework.com/).

## Entorno de trabajo

Instalar [editor de texto](http://www.vim.org/) y plugin para sintaxis de [Solidity](https://github.com/ethereum/solidity).

    $ sudo pacman -Sy ack ctags git rake gvim
    $ curl -L https://bit.ly/janus-bootstrap | bash
    $ mkdir .janus
    $ cd .janus/
    $ git clone https://github.com/tomlion/vim-solidity.git vim-solidity

Instalar [Node.js](https://nodejs.org/en/) utilizando [nvm](https://github.com/creationix/nvm).

    $ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
    $ nvm install node
    $ nvm use node


## Implementación local

Crear carpeta para la aplicación e instalar [testrpc](https://github.com/ethereumjs/testrpc)
y [web3](https://github.com/ethereum/web3.js/). **testrpc** permite correr una cadena de bloques
en el equipo local, y **web3** es un API de Ethereum en JavaScript.

    $ mkdir votaciones
    $ cd votaciones
    $ npm install ethereumjs-testrpc web3 solc

### Cuentas

Cada vez que ejecutamos testrpc obtenemos 10 cuentas Ethereum,
cada una con 100 ETH. El servicio queda escuchando en el puerto 8545.

    EthereumJS TestRPC v4.0.1 (ganache-core: 1.0.1)

    Available Accounts
    ==================
    (0) 0x025ec68ce2f762d8a174876af42b01b0e694fd39
    (1) 0xa4cca1611cd268771c3ddbe57921a9eb369b5559
    (2) 0xafb6c269dd94e4cc4fcda579dd98930a9e718b2a
    (3) 0x90e8ca57d4f3320d47fb93e00a38574dee628b31
    (4) 0x4b92fc85d5599a8f99b97dc8a2a6831ae2eca5a3
    (5) 0x201693a35db9c566579c3ed6918b681ee1ec0676
    (6) 0xe6143238392f487e06d74b59adaff1715dc4882f
    (7) 0x40f03fb282a94fde970683e3d06737d84406e6be
    (8) 0xb1743a5f97b03862b975a0fc618b59b25f8b0428
    (9) 0xf5cc7be45c74177c788d1dbccb08a2d609188587

    Private Keys
    ==================
    (0) dfccbc1c5df0cd253cdd3e38e95b7226d15cae300dab2e13e8ab86e588cadbb1
    (1) c910cb894f1f1eb7b402221edbd339b6ac499e39dbdf970ef5ecb49f9fd93c1c
    (2) f2f3dfe563f24a0b1eab8b329151500da60297dd8b4cb89d7d77294c6d712095
    (3) c1c935d22460fd58d68b122e1c2f3582ec3b6829906301de83280d5f9fbace60
    (4) 5cc7ba135396d5a2cd6b99a7c1d0175483acf21154b5db2f6e372120ce332ac5
    (5) dd286b8ab2e29d865f6582a5a4a7422a543e68059ebca64305553c34041ee42a
    (6) d2ca6db3bf9ff0878403f3e5c19fe95317e70f2aa84f64d9c3a07f4952fe4818
    (7) da2cc027ed84571654f4b678fa35da6fdfa0c1b51a7691089395c78503da796d
    (9) 1b7275ae26824d1aef5ee09d7bda06547741a3c97461ac4ca1bec3aa75262658

    HD Wallet
    ==================
    Mnemonic:      ahead forest plunge power boris kind entry depart potato pluck expect empower
    Base HD Path:  m/44'/60'/0'/0/{account_index}

    Listening on localhost:8545

### Contrato Inteligente

Creamos un contrato con dos funciones, una para leer el total de votos
de cada candidato, y otra para votar por un candidato. Los candidatos se
inician en el constructor del contrato.

    pragma solidity ^0.4.11;
    // Versión del compilador

    contract Votaciones {

      /* Hash que asocia los nombres de los candidatos (bytes32) con el
      número de votos recibidos (uint8) */
      mapping (bytes32 => uint8) public tablaDeVotaciones;

      // La lista de candidatos se almacena en un vector de tipo bytes32
      bytes32[] public listaDeCandidatos;

      // Constructor
      function Votaciones(bytes32[] nombresDeCandidatos) {
        listaDeCandidatos = nombresDeCandidatos;
      }

      // Esta función devuelve el total de votos que un candidato ha recibido hasta el momento
      function numeroDeVotosPara(bytes32 candidato) returns (uint8) {
        if (elCandidatoEsValido(candidato) == false) throw;
        return tablaDeVotaciones[candidato];
      }

      // Esta función incrementa el número de votos de un candidato
      function votarPor(bytes32 candidato) {
        if (elCandidatoEsValido(candidato) == false) throw;
        tablaDeVotaciones[candidato] += 1;
      }

      function elCandidatoEsValido(bytes32 candidato) returns (bool) {
        for(uint i = 0; i < listaDeCandidatos.length; i++) {
          if (listaDeCandidatos[i] == candidato) {
            return true;
          }
        }
        return false;
      }
    }

### Compilación

Para compilar el contrato se utiliza la librería `solc`, y para
interactuar con la cadena de bloques a través de RPC se utiliza la librería
`web3js`.

Realizamos el procedimiento desde una consola de Node. El servicio
testrpc debe estar en ejecución.

    $ node
    > Web3 = require('web3')
    > web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

Para probar que podemos comunicarnos con la cadena de bloques de testrpc,
consultamos las cuentas existentes desde la consola de Node con web3.

    > web3.eth.accounts

Debemos obtener el listado de las cuentas que creó testrpc cuando
inició.

Para compilar el contrato primero se convierte a una cadena de texto y
luego se compila utilizando `solc`.

    > codigo = fs.readFileSync('Votaciones.sol').toString()
    > solc = require('solc')
    > codigoCompilado = solc.compile(codigo)

### Publicar contrato en la cadena de bloques local

Para publicar el contrato primero creamos el objeto a partir de su
interfaz, y luego subimos los datos.

    > interfazContrato = JSON.parse(codigoCompilado.contracts[':Votaciones'].interface)
    > ContratoVotaciones = web3.eth.contract(interfazContrato)
    > codigoEnBytes = codigoCompilado.contracts[':Votaciones'].bytecode
    > contratoSubido = ContratoVotaciones.new(['Juan', 'Pedro', 'Luis'], { data: codigoEnBytes, from: web3.eth.accounts[0], gas: 4700000})

El contrato publicado queda alojado en una dirección de la cadena de
bloques:

    > contratoSubido.address
    '0x54f058e56a19d1e063c34709f3c40ad36db306b8'

La instancia del contrato nos sirve para interactuar con este:

    > instanciaContrato = ContratoVotaciones.at(ContratoSubido.address)

Para utilizar el contrato se necesita la interfaz y la dirección.

### Interactuar con el contrato utilizando Node

Podemos realizar llamados a las funciones del contrato desde la consola
de Node:

    > instanciaContrato.numeroDeVotosPara.call('Juan')
    { [String: '0'] s: 1, e: 0, c: [ 0 ] }
    > instanciaContrato.votarPor('Juan', { from: web3.eth.accounts[0] })
    '0x12383d4717c11ef9a996ae3260e6d934ff20a240a928a208774d5c2733ae3afe'
    > instanciaContrato.numeroDeVotosPara.call('Juan')
    { [String: '1'] s: 1, e: 0, c: [ 1 ] }
    > instanciaContrato.numeroDeVotosPara.call('Juan').toLocaleString()
    '1'

    > instanciaContrato.votarPor.call('Juan', { from: web3.eth.accounts[0] })
    []

Cada vez que se vota por un candidato se obtiene el id de la
transacción.

### Interactuar con el contrato utilizando una página web

De igual manera, podemos crear una interfaz web para realizar los
llamados a las funciones del contratos.

Creamos un archivo HTML con los nombres de los candidatos e invocamos
los comandos utilizados en la consola de Node.

Archivo `index.html`:

    <!DOCTYPE html>
    <html>
    <head>
      <title>Votaciones | HorMinka DApps</title>
      <link href='https://fonts.googleapis.com/css?family=Open+Sans:400,700' rel='stylesheet' type='text/css'>
      <link href='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css' rel='stylesheet' type='text/css'>
    </head>
    <body class="container">
      <h1>Votaciones HorMinka</h1>
      <div class="table-responsive">
        <table class="table table-bordered">
          <thead>
            <tr>
              <th>Candidato</th>
              <th>Votos</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>Juan</td>
              <td id="candidato-1"></td>
            </tr>
            <tr>
              <td>Pedro</td>
              <td id="candidato-2"></td>
            </tr>
            <tr>
              <td>Luis</td>
              <td id="candidato-3"></td>
            </tr>
          </tbody>
        </table>
      </div>
      <input type="text" id="candidato" />
      <a href="#" onclick="votarPorCandidato()" class="btn btn-primary">Votar</a>
    </body>
    <script src="https://cdn.rawgit.com/ethereum/web3.js/develop/dist/web3.js"></script>
    <script src="https://code.jquery.com/jquery-3.1.1.slim.min.js"></script>
    <script src="./index.js"></script>
    </html>

Archivo `index.js`:

    interfaz = '[{"constant":false,"inputs":[{"nombre":"candidato","type":"bytes32"}],"name":"numeroDeVotosPara","outputs":[{"name":"","type":"uint8"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"candidato","type":"bytes32"}],"name":"votarPor","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"","type":"uint256"}],"name":"listaDeCandidatos","outputs":[{"name":"","type":"bytes32"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"","type":"bytes32"}],"name":"tablaDeVotaciones","outputs":[{"name":"","type":"uint8"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"candidato","type":"bytes32"}],"name":"elCandidatoEsValido","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"inputs":[{"name":"nombresDeCandidatos","type":"bytes32[]"}],"payable":false,"type":"constructor"}]';
    direccion = '0x54f058e56a19d1e063c34709f3c40ad36db306b8';

    web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

    abi = JSON.parse(interfaz);

    ContratoVotaciones = web3.eth.contract(abi);
    instanciaContrato = ContratoVotaciones.at(direccion);
    candidatos = {"Juan": "candidato-1", "Pedro": "candidato-2", "Luis": "candidato-3"}

    function votarPorCandidato() {
      nombreCandidato = $("#candidato").val();
      instanciaContrato.votarPor(nombreCandidato, {from: web3.eth.accounts[0]}, function() {
        let div_id = candidatos[nombreCandidato];
        $("#" + div_id).html(instanciaContrato.numeroDeVotosPara.call(nombreCandidato).toString());
      });
    }

    $(document).ready(function() {
      nombresCandidatos = Object.keys(candidatos);
      for (var i = 0; i < nombresCandidatos.length; i++) {
        let nombre = nombresCandidatos[i];
        let votos = instanciaContrato.numeroDeVotosPara.call(nombre).toString()
        $("#" + candidatos[nombre]).html(votos);
      }
    });

## Implementación en testnet

### Geth

Instalamos [Geth](https://github.com/ethereum/go-ethereum), la implementación de Ethereum en Go que nos permite
ejecutar un nodo local de Ethereum, interactuar con la cadena de bloques y
administrar las cuentas de usuario.

    $ suco pacman -Sy geth

Iniciamos un nodo de Ethereum en la [tesnet Ropsten](https://github.com/ethereum/ropsten) y sincronizamos la cadena de
bloques rápidamente.

    $ geth --testnet --fast --cache=1024 console

### Truffle

Truffle es unn framework para desarrollo, prueba y publicación de
contratos inteligentes con Solidity y Javascript.

    $ npm install -g truffle

Iniciamos un proyecto basado en la [plantilla webpack](https://github.com/trufflesuite/truffle-init-webpack).

    $ mkdir votaciones-trfl
    $ cd votaciones-trfl
    $ npm install -g webpack
    $ truffle init webpack

Se eliminan los contratos de demostración y se copia el de votaciones.

A continuación se edita el archivo de migración `2_deploy_contracts.js`
para que publique nuestro contrato de votaciones correctamente:

    var Votaciones = artifacts.require("./Votaciones.sol");

    module.exports = function(deployer) {
      deployer.deploy(Votaciones, ['Hor', 'Min', 'Ka'], { gas: 290000 });
    };

Luego escribimos en `app/javascripts/app.js` el código JS para interactuar con el contrato:

    // Import the page's CSS. Webpack will know what to do with it.
    import "../stylesheets/app.css";

    // Import libraries we need.
    import { default as Web3} from 'web3';
    import { default as contract } from 'truffle-contract'

    /*
     * Cuando se compila y se publica el contrato, truffle guarda el abi y la dirección en un archivo json
     * en el directorio build. Esta información se utiliza para crear una abstracción del contrato, que después
     * utilizamos para crear un instancia del contrato e interactuar con él.
     */

    import votaciones_artifacts from '../../build/contracts/Votaciones.json'

    var Votaciones = contract(votaciones_artifacts);

    let candidatos = {"Hor": "candidato-1", "Min": "candidato-2", "Ka": "candidato-3"}

    window.votarPorCandidato = function(candidato) {
      let nombreCandidato = $("#candidato").val();
      try {
        $("#msg").html("Se ha recibido su voto. El número de votos se actualizará tan pronto se registre el voto en la cadena de bloques. Por favor espere.")
        $("#candidato").val("");

        /* Votaciones.deployed() devuelve una instancia del contrato.
         * Toda llamada en truffle devuelve una promesa
         */
        Votaciones.deployed().then(function(instanciaContrato) {
          instanciaContrato.votarPor(nombreCandidato, {gas: 140000, from: web3.eth.accounts[0]}).then(function() {
            let div_id = candidatos[nombreCandidato];
            return instanciaContrato.numeroDeVotosPara.call(nombreCandidato).then(function(v) {
              $("#" + div_id).html(v.toString());
              $("#msg").html("");
            });
          });
        });
      } catch (err) {
        console.log(err);
      }
    }

    $( document ).ready(function() {
      if (typeof web3 !== 'undefined') {
        console.warn("Using web3 detected from external source like Metamask")
        // Use Mist/MetaMask's provider
        window.web3 = new Web3(web3.currentProvider);
      } else {
        console.warn("No web3 detected. Falling back to http://localhost:8545. You should remove this fallback when you deploy live, as it's inherently insecure. Consider switching to Metamask for development. More info here: http://truffleframework.com/tutorials/truffle-and-metamask");
        // fallback - use your fallback strategy (local node / hosted node + in-dapp id mgmt / fail)
        window.web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
      }

      Votaciones.setProvider(web3.currentProvider);
      let nombresCandidatos = Object.keys(candidatos);
      for (var i = 0; i < nombresCandidatos.length; i++) {
        let nombre = nombresCandidatos[i];
        Votaciones.deployed().then(function(instanciaContrato) {
          instanciaContrato.numeroDeVotosPara.call(nombre).then(function(v) {
            $("#" + candidatos[nombre]).html(v.toString());
          });
        })
      }

Copiamos la página web del ejemplo anterior. Editamos para que agregue
el script `app.js` que acabamos de crear.

    <script src="app.js"></script>

### Cuentas y Ether

Para publicar un contrato en la testnet debemos tener una cuenta y algo
de ether. La cuenta la podemos crear con la consola de truffle.

Primero tenemos que asegurarnos que el tenemos activo el nodo de
Ethereum esperando conexiones en el puerto 8545

    $ geth --testnet --rpc --rpcapi db,eth,net,web3,personal --cache=1024 --rpcport 8545 --rpcaddr 127.0.0.1 --rpccorsdomain "*" console

A continuación iniciamos la consola de truffle y creamos una cuenta en
la testnet.

    $ truffle console
    truffle(development)> web3.personal.newAccount('strong password')
    '0x2d23aa9fc9191c56e45501e6a7955d4a8ab6505a'
    truffle(development)> web3.eth.getBalance('0x4d23aa9fc9191c56e45501e6a7955d4a8ab6505a')
    { [String: '0'] s: 1, e: 0, c: [ 0 ] }

Para obtener Ether, podemos utilizar un faucet, un servicio web al que
le solicitamos envíe ether a nuestra cuenta en la testnet.

    $ curl -X POST  -H "Content-Type: application/json" -d '{"toWhom":"0x2d23aa9fc9191c56e45501e6a7955d4a8ab6505a"}' https://ropsten.faucet.b9lab.com/tap
    {
      "txHash" : "0xcd738ec4815d04523373a97101fae7df4e87d3c01214fa0bc37fc8f39f9500c1"
    }

Luego de hacer la solicitud podemos verificar el nuevo saldo

    truffle(development)> web3.eth.getBalance('0x4d23aa9fc9191c56e45501e6a7955d4a8ab6505a')
    { [String: '1000000000000000000'] s: 1, e: 18, c: [ 10000 ] }

### Publicar contrato

Por defecto truffle utiliza la cuenta `accounts[0]` para publicar los
contratos en la cadena de bloques.

    truffle(development)> web3.eth.accounts
    [ '0x2d23aa9fc9191c56e45501e6a7955d4a8ab6505a' ]

Se puede establecer esta cuenta agregando el campo `from` en el archivo
de configuración `truffle.js`.

Por defecto, las cuentas en Geth están bloqueadas, lo que quiere decir
que no se pueden enviar transacciones desde ellas. Para publicar un contrato en la cadena de bloques debemos desbloquear la cuenta de forma temporal (3000 segundos) mientras subimos el contrato.

    truffle(default)> web3.personal.unlockAccount('0x2d23aa9fc9191c56e45501e6a7955d4a8ab6505a', 'strong password', 3000)
    true

Una vez desbloqueada la cuenta salimos de la consola de truffle y
publicamos el contrato en la cadena de bloques.

    truffle(development)> truffle migrate
    Using network 'development'.

    Running migration: 1_initial_migration.js
      Deploying Migrations...
      Migrations: 0xd93352496cf54ab358c6ea51afd801c7cdebb05f
    Saving successful migration to network...
    Saving artifacts...
    Running migration: 2_deploy_contracts.js
      Deploying Votaciones...
      Votaciones: 0x68b08e04924c8c75da8aa227574ff4dfe435c3a7
    Saving successful migration to network...
    Saving artifacts...

#### Posibles problemas

  * **transaction could not be found in next 50 blocks**: sincronizar el
 la cadena de bloques y verificar que no hayan transacciones pendientes
`eth.syncing`, `txpool.status`, `eth.pendingTransactions.length`.

  * **not enough gas**: agregar los parámetros `gas` y `gasPrice` al
    archivo de configuración `truffle.js`. Se puede tomar como
referencia la información del último bloque
`web3.eth.getBlock("latest").gasLimit`. Agregar el parámetro `gas` en el
contrato de publicación (`2_deploy_contracts.js`).

### Interactuar con el contrato

Votar por un candidato desde consola:

    truffle(development)> Votaciones.deployed().then(function(contractInstance) {contractInstance.votarPor('Hor').then(function(v) {console.log(v)})})
    undefined
    truffle(development)> { tx: '0x19bad3ba856e75439051e6ec14884cb93e97883abd9c5dea13c2854c37a6a2b5',
      receipt: 
       { blockHash: '0xa561577ccd3b241fb16dd19370e8c07a64e1baba7f506881c7fbd7957f63588f',
         blockNumber: 1411396,
         contractAddress: null,
         cumulativeGasUsed: 42939,
         from: '0x2d23aa9fc9191c56e45501e6a7955d4a8ab6505a',
         gasUsed: 42939,
         logs: [],
         logsBloom: '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
         root: '0x52610a22003b532f9d07fb1ab0de53c51d112324ca22e40cf9a914f5794d366d',
         to: '0x68b08e04924c8c75da8aa227574ff4dfe435c3a7',
         transactionHash: '0x19bad3ba856e75439051e6ec14884cb93e97883abd9c5dea13c2854c37a6a2b5',
         transactionIndex: 0 },
      logs: [] }

Consultar número de votos desde consola:

    truffle(development)> Votaciones.deployed().then(function(contractInstance) {contractInstance.numeroDeVotosPara.call('Hor').then(function(v) {console.log(v)})})
    undefined
    truffle(development)> { [String: '1'] s: 1, e: 0, c: [ 1 ] }

Utilizar la interfaz web

    $ npm run dev

Para realizar operaciones que impliquen transacciones debemos tener la
cuenta desbloqueada.

## Referencias

  * [Medium Post](https://medium.com/@mvmurthy/full-stack-hello-world-voting-ethereum-dapp-tutorial-part-1-40d2d0d807c2)
  * [Ethereum Wiki](https://github.com/ethereum/go-ethereum/wiki)
  * [Bitcoin Wiki](https://en.bitcoin.it/wiki/Main_Page)
  * [Truffle Gitter](https://gitter.im/ConsenSys/truffle)
