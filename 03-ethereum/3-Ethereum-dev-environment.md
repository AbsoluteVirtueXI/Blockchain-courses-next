# Ethereum development environment

## introduction

Le développement de smart contracts est un processus qui peut paraitre pénible comparé à d'autres domaines comme du développement front-end ou backend:

- pas de feedback immédiat sur notre application, notre smart contract est déployé sur la blockchain Ethereum
- déployer notre smart contract sur un noeud local ou un testnet peut prendre du temps.
- utiliser notre smart contract sur un testnet prend du temps aussi.
- Il faut jongler avec les adresses des différents accounts qui vont effectuer les appels à notre smart contract
- Il faut de l'ether ou des tokens de tests
- pour interagir avec notre smart contract durant son développement il faudrait utiliser remix (à bannir) ou developper un frontend ou des scripts node.js
- tester notre smart contract de manière exhaustive manuellement, sur remix, ou via une interface graphique est **impossible** dès qu'il y a plusieurs variables d'états et plusieurs fonctions qui les manipulent
- il y a trop de cas possibles à tester: Les variables, les reverts, les events, les différentes conditions basées sur un timestamp ou un block number, etc...

Pour pallier à tout ces problèmes nous allons utiliser un environnement de développement qui nous facilitera notre processus de développement et qui proposera des outils pour automatiser certaines tâches.

## Smart contracts development process

### Write Solidity files

Les fichiers Solidity sont écrits via un éditeur de texte dans des fichiers avec l'extension `.sol`.  
Nous utiliserons VSCode pour cela avec l'extension [Solidity by Juan Blanco](https://marketplace.visualstudio.com/items?itemName=juanblanco.solidity) afin d'avoir du syntaxe `syntax highlighting`.
Nous utiliserons aussi un prettier pour formater notre code suivant les bonnes pratiques, ainsi qu'un linter pour bénéficier d'une analyse statique des erreurs durant l'édition de nos fichiers Solidity.

### Compile with the solc compiler

Nos smart contracts seront compilés avec le compilateur `solc`.

### Test

On test des smart contract écrit en Solidity mais avec des scripts Javascript.  
Ces scripts utiliseront les librairies `mocha` et `chai` pour effectuer des tests unitaires et la librairie `ethers.js` (web3) pour communiquer avec un noeud Ethereum local de test, le `Hardhat network` sur lequel seront déployés nos smart contracts à tester.  
Les tests sont automatisés par des scripts aucune interaction avec le développeur. Il faudra écrire les tests que nous souhaitons exécutés.

### Deploy

On déploie nos smart contracts sur un noeud local, un tesnet ou le mainnet Ethereum avec des scripts Javascript.  
Ces scripts utiliseront la librairie `ethers.js` pour communiquer avec la Blockchain.

### Share ABI and your smart contract addresses with your frontend or whatever

Si une application à besoin de communiquer avec avec nos smart contracts, généralement une frontend web ou mobile il faudra fournir aux développeurs l'ABI et les adresses de nos smart contracts.  
L'ABI (Application Binary Interface) est générée au moment de la compilation.  
L'adresse du smart contract est récupérée au moment du déploiement sur la Blockchain, mainnet, testnet ou réseau local.

## Hardhat

### Overview

https://hardhat.org/getting-started/#overview

[HardHat](https://hardhat.org/) est un `Ethereum development environment for professionals`.
`Hardat` est qualifié de:

- Flexible: On peut utiliser tout ou juste une partie de Hardhat selon nos besoins.
- Extensible: Hardhat utilise un système de plugins que l'on peut importer en fonction de nos besoins.
- Fast: Accélère le processus de développement, automatise certaines tâches et leur noeud local utilisé pour effectuer nos tests, le `Hardhat network` est bien plus rapide à exécuter nos transactions qu'une instance de ganache.

Hardhat est utilisable via la ligne de commande.  
Nous y exécutons des `tasks`, certaines sont fournies par défaut, d'autres viennent de `plugins` que nous devrons installer (via `yarn` toujours).  
Les `tasks` les plus communes sont la compilation de nos smart contracts, l'éxecution de scripts de tests ou de déploiements.

### Hardhat vs truffle vs Embark

Jusqu'a l'arrivée de `Hardhat` la seule alternative était `Truffle` et `Embark`.  
`Truffle` reste très utilisé, nous l'utiliserons aussi dans le cadre de cette formation pour que vous puissiez au moins une fois setup un projet `Truffle`.  
Néanmoins `HardHat` possède un [plugin](https://hardhat.org/plugins/nomiclabs-hardhat-truffle5.html) qui permet la compatibilité de tous les scripts de tests écrits pour `Truffle`, afin qu'ils puissent aussi être utilisés dans notre projet `Hardhat`.

### Installation

https://hardhat.org/getting-started/#installation

Hardhat devra être installé localement pour chacun de vos projets.
Hardhat est une suite d'outils Node.js, il faudra donc installer Hardhat via le package manager Node.js, donc dans un dossier qui a été initialisé avec `yarn init`.

```zsh
mkdir first-hardhat-project
cd first-hardhat-project
yarn init -y
yarn add hardhat --dev
```

Ouvrir avec VSCode.

On se retrouve avec un projet Hardhat vide non initialisé.  
C'est comme on peut le remarquer aussi un projet Node.js (présence d'un répertoire _node_modules/_ contenant tous les packages nécéssaires à une installation par défaut de Hardhat et d'un fichier _package.json_)

### Setup an Hardhat project

https://hardhat.org/getting-started/#quick-start

Maintenant que Hardhat est installé nous pouvons setup notre nouveau projet avec:

```zsh
npx hardhat
```

A l'invite de la ligne de commande choisir:

- `Create a sample project` afin de tester Hardhat avec un simple exemple de projet créé par défaut par Hardat.
- Garder le répertoire par défaut pour le `Hardhat project root` qui devrait être votre répertoire courant, celui de votre nouveau projet.
- sélectionner `y` pour générer un fichier _.gitignore_
- sélectionner `y` pour installer des dépendances par défaut pour gérer la compatibilité de tests écrits avec une autre librairie

Dans l'explorateur de fichier vous pouvez remarquer que de nouveaux répertoires et fichiers sont présents.
Ils constituent les éléments essentiels à un projet Hardhat

### Running tasks

https://hardhat.org/getting-started/#quick-start

Les `tasks` sont toutes les fonctionnalités que vous offre le client Hardhat.  
Les `tasks` les plus communes sont accessibles par défaut d'un projet Hardhat. Les plugins peuvent fournir des `tasks` supplémentaires à leur installation et à leur activation, vous pouvez aussi créer votre propres `tasks`

Pour avoir la list de toutes les `tasks` disponibles:

```zsh
npx hardhat
Hardhat version 2.3.0

Usage: hardhat [GLOBAL OPTIONS] <TASK> [TASK OPTIONS]

GLOBAL OPTIONS:

  --config           	A Hardhat config file.
  --emoji            	Use emoji in messages.
  --help             	Shows this message, or a task's help if its name is provided
  --max-memory       	The maximum amount of memory that Hardhat can use.
  --network          	The network to connect to.
  --show-stack-traces	Show stack traces.
  --tsconfig         	Reserved hardhat argument -- Has no effect.
  --verbose          	Enables Hardhat verbose logging
  --version          	Shows hardhat's version.


AVAILABLE TASKS:

  accounts	Prints the list of accounts
  check   	Check whatever you need
  clean   	Clears the cache and deletes all artifacts
  compile 	Compiles the entire project, building all artifacts
  console 	Opens a hardhat console
  flatten 	Flattens and prints contracts and their dependencies
  help    	Prints this message
  node    	Starts a JSON-RPC server on top of Hardhat Network
  run     	Runs a user-defined script after compiling the project
  test    	Runs mocha tests

To get help for a specific task run: npx hardhat help [task]
```

Dans le sous dossier _contracts/_ contiendra tous nos smart contracts.  
Avec le setup par défaut, il contient un smart contract _Greeter.sol_.

### Running `compile` task

https://hardhat.org/getting-started/#compiling-your-contracts

Pour compiler tous nos smart contracts il faut lancer la task `compile`:

```zsh
npx hardhat compile
Downloading compiler 0.7.3
Compiling 2 files with 0.7.3
Compilation finished successfully
```

On peut remarquer que 2 smart contracts ont été compilés, et que la version du compilateur `solc` n'est pas la dernière.  
Pour changer la version du compilateur il faut modifier le fichier de configuration du projet Hardhat _hardhat.config.js_ et choisir la dernière version du compilateur connue. Dans notre cas la `0.8.4`.  
A la fin du fichier _hardhat.config.js_ modifier la version du compilateur `solc`

```js
module.exports = {
  solidity: '0.8.4',
}
```

Il faudra aussi modifier la version de `pragma` du fichier _Greeter.sol_ pour qu'elle match la version de notre compilateur. Au passage profitons en pour réecrire le smart contract `Greeter` avec les bonnes conventions recommandées:

```solidity
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "hardhat/console.sol";


contract Greeter {
  string private _greeting;

  constructor(string memory greeting_) {
    console.log("Deploying a Greeter with greeting:", greeting_);
    _greeting = greeting_;
  }

  function setGreeting(string memory greeting_) public {
    console.log("Changing greeting from '%s' to '%s'", _greeting, greeting_);
    _greeting = greeting_;
  }

  function greeting() public view returns (string memory) {
    return _greeting;
  }
}
```

Ensuite `clean` le cache et tous les `artificats` générés par la dernière compilation et recompiler:

```zsh
npx hardhat clean
npx hardhat compile
Solidity 0.8.4 is not fully supported yet. You can still use Hardhat, but some features, like stack traces, might not work correctly.

Learn more at https://hardhat.org/reference/solidity-support"

Downloading compiler 0.8.4
Compiling 2 files with 0.8.4
Compilation finished successfully
```

Si la compilation de nos smart contracts se passe sans erreurs 2 nouveaux répertoires sont générés:

- _cache/_: répertoire interne à Hardhat. Il contient des informations relatives à notre projet.
- _artifacts/_: Ce repertoire contient les `artifacts` de nos smart contracts. Ce sont des fichiers `JSON` contenant des metadonnées et surtout l'ABI et le bytecode de nos smart contract compilés. Les artificats de chacun de nos smart contracts sont disponibles dans _artifacts/contracts/_.

### Running `test` task

https://hardhat.org/getting-started/#testing-your-contracts

Tous nos scripts de tests se trouvent dans le répertoire _test/_.  
Généralement un smart contract doit posséder un fichier de script de tests.  
Une fois exécuté, ce script lance une batterie de tests qui simuleront un déploiement de notre smart contract, ainsi que des transactions et nous pourrons ainsi vérifier son exécution en comparant les résultats de ces exécutions à des valeurs ou comportements déterminés à l'avance dans nos tests.  
Rien de magique il faudra écrire nos propres tests pour chacun de nos smart contracts.  
Pour cela il faudra utiliser la librairie [mocha](https://mochajs.org/) qui est un test framework sur node, couplé à [chai](https://www.chaijs.com/) qui est une `BDD / TDD assertion library for node`.  
Pour que notre code Javascript de test, mais pas que, sache communiquer avec la blockchain Ethereum (ou un testnet / réseau local) on utilise la librairie [ethers.js](https://docs.ethers.io/v5/).

Afin de lancer nos tests sur nos smart contracts il faut absolument que ces derniers soient compilés avant car les scripts de tests ont besoin des `artificats` des contrats qu'ils doivent tester. Si ce n'est pas le cas le comportement par défaut de la `task` `test` est de compiler les smart contracts avant d'exécuter les scripts de test.  
Avec le setup par défaut de notre projet Hardhat un script de test _test/sample-test.js_ est fourni pour tester le smart contract `Greeter` du fichier _contracts/Greeter.sol_
Afin de suivre de meilleure convention renommer le fichier _test/sample-test.js_ en _test/Greeter-test.js_.  
Modifier le fichier _test/Greeter-test.js_ pour qu'il teste correctement notre smart contract `Greeter` modifié précédemment:

```js
const { expect } = require('chai')

describe('Greeter', function () {
  it("Should have 'Hello, world!' at deployement", async function () {
    const Greeter = await ethers.getContractFactory('Greeter')
    const greeter = await Greeter.deploy('Hello, world!')
    await greeter.deployed()
    expect(await greeter.greeting()).to.equal('Hello, world!')
  })

  it("Should return the new greeting once it's changed", async function () {
    const Greeter = await ethers.getContractFactory('Greeter')
    const greeter = await Greeter.deploy('Hello, world!')
    await greeter.deployed()
    await greeter.setGreeting('Hola, mundo!')
    expect(await greeter.greeting()).to.equal('Hola, mundo!')
  })
})
```

Pour exécuter la `task` `test` on utilise le client Hardhat en ligne de commande:

```zsh
npx hardhat test


  Greeter
Deploying a Greeter with greeting: Hello, world!
    ✓ Should have 'Hello, world!' at deployement (1083ms)
Deploying a Greeter with greeting: Hello, world!
Changing greeting from 'Hello, world!' to 'Hola, mundo!'
    ✓ Should return the new greeting once it's changed (94ms)


  2 passing (1s)
```

Le script `Greeter-test.js` contient 2 tests `Should have 'Hello, world!' at deployement` et `Should return the new greeting once it's changed` qui sont passés avec succès.  
Les tests de nos smart contract sont exécutés sur une instance du `Hardhat network` non persistante.

### Running `run` task for deploying smart contracts.

https://hardhat.org/getting-started/#deploying-your-contracts

Les scripts de déploiement sont dans le répertoire _scripts/_.  
Il n'existe pas de `task` `deploy` pour déployer un smart contract consiste à exécuter un script qui récupérera le bytecode du smart contract dans son artificat associé et qui enverra ensuite une instruction de création de smart contract à la Blockchain.  
Pas de surprise c'est la librairie `ethers.js` qu'on utilise pour effectuer cela depuis un script Javascript.  
Avec le setup par défaut de notre projet Hardhat un script de déploiement _test/sample-test.js_ est fourni pour déployer le smart contract `Greeter` du fichier _contracts/Greeter.sol_  
Afin de suivre de meilleure convention renommer le fichier _scripts/sample-test.js_ en _scripts/deploy-Greeter.js_.
Pour effectuer un déploiement on utilise la `task` `run` en lui passant comme arguments le script de déploiement à exécuter en argument de la ligne de commande.

```zsh
npx hardhat run scripts/deploy-Greeter.js
Deploying a Greeter with greeting: Hello, Hardhat!
Greeter deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

Par défaut notre smart contract est déployé sur une instance du `Hardhat network` non persistante.  
Il n'y a pas vraiment d'interêt (sauf dans le cadre de nos tests unitaires).  
Pour déployer sur une instance persistante du `Hardhat network` il faut lancer un noeud local:

```zsh
npx hardhat node
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/
```

Il faudra ensuite déployer notre smart contract en spécifiant le network sur lequel on souhaite déployer, le `Hardhat network` est par défaut sur localhost:

```zsh
npx hardhat run scripts/deploy-Greeter.js --network localhost
Greeter deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

Nous pouvons ensuite, si nous le souhaitons nous interfacer manuellement avec notre smart contract via la `Hardhat console`.  
Il faudra spécifier sur quel network nous souhaitons lancer notre `Hardhat console`.  
Pour envoyer une transaction à notre smart contract `Greeter` déployé sur le `Hardhat network` à l'adresse `0x5FbDB2315678afecb367f032d93F642f64180aa3`, il faudra spécifier le network `localhost`.

```zsh
npx hardhat console --network localhost
Welcome to Node.js v15.1.0.
Type ".help" for more information.
> const Greeter = await ethers.getContractFactory("Greeter")
> const greeter = await Greeter.attach("0x5FbDB2315678afecb367f032d93F642f64180aa3")
> const message = await greeter.greeting()
> console.log(message)
Hello, Hardhat!
> await greeter.setGreeting("Hola, mundo!")
> const newMessage = await greeter.greeting()
> console.log(newMessage)
Hola, mundo!
```

En général pour préférerons utiliser des scripts pour s'interfacer avec notre smart contract déployé plutôt que de le faire manuellement avec la `Hardhat console`.

### Architecture of an Hardhat project

## Setup an Ethereum development environment

https://hardhat.org/tutorial/creating-a-new-hardhat-project.html

### VSCode extensions

Install the following extensions:
[EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)  
[ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)  
[Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)  
[solidity by Juan Branco](https://marketplace.visualstudio.com/items?itemName=JuanBlanco.solidity)

### Create a new project

Create a new directory and setup a new project with yarn and git:

```zsh
mkdir CoinCoin-Token
cd CoinCoin-Token
yarn init -y
git init
```

Don't forget to create a _.gitignore_ file with at least the following content:

```zsh
node_modules

#Hardhat files
cache
artifacts

#VSCode files
.vscode
```

### Install and configure prettiers and linters

Install prettiers, linters and configure their configurations files as follow:

```zsh
yarn add prettier prettier-plugin-solidity solhint eslint eslint-config-standard eslint-plugin-import eslint-plugin-mocha-no-only eslint-plugin-node eslint-plugin-promise eslint-plugin-standard --dev
```

_.prettierrc_:

```json
{
  "overrides": [
    {
      "files": "*.sol",
      "options": {
        "printWidth": 120,
        "tabWidth": 4,
        "useTabs": false,
        "singleQuote": false,
        "bracketSpacing": false,
        "explicitTypes": "always"
      }
    },
    {
      "files": "*.js",
      "options": {
        "tabWidth": 2,
        "useTabs": false,
        "semi": true,
        "singleQuote": true,
        "arrowParens": "always",
        "bracketSpacing": true
      }
    }
  ]
}
```

_.eslintrc_:

```json
{
  "extends": ["standard", "plugin:promise/recommended"],
  "plugins": ["mocha-no-only", "promise"],
  "env": {
    "browser": true,
    "node": true,
    "mocha": true,
    "jest": true
  },
  "globals": {
    "artifacts": false,
    "contract": false,
    "assert": false,
    "web3": false,
    "usePlugin": false,
    "extendEnvironment": false
  },
  "rules": {
    // Strict mode
    "strict": ["error", "global"],

    // Code style
    "array-bracket-spacing": ["off"],
    "camelcase": ["error", { "properties": "always" }],
    "comma-dangle": ["error", "always-multiline"],
    "comma-spacing": ["error", { "before": false, "after": true }],
    "dot-notation": ["error", { "allowKeywords": true, "allowPattern": "" }],
    "eol-last": ["error", "always"],
    "eqeqeq": ["error", "smart"],
    "generator-star-spacing": ["error", "before"],
    "indent": ["error", 2],
    "linebreak-style": ["error", "unix"],
    "max-len": ["error", 120, 2],
    "no-debugger": "off",
    "no-dupe-args": "error",
    "no-dupe-keys": "error",
    "no-mixed-spaces-and-tabs": ["error", "smart-tabs"],
    "no-redeclare": ["error", { "builtinGlobals": true }],
    "no-trailing-spaces": ["error", { "skipBlankLines": false }],
    "no-undef": "error",
    "no-use-before-define": "off",
    "no-var": "error",
    "object-curly-spacing": ["error", "always"],
    "prefer-const": "error",
    "quotes": ["error", "single"],
    "semi": ["error", "always"],
    "space-before-function-paren": ["error", "always"],

    "mocha-no-only/mocha-no-only": ["error"],

    "promise/always-return": "off",
    "promise/avoid-new": "off"
  },
  "parserOptions": {
    "ecmaVersion": 2018
  }
}
```

_.solhint.json_:

```json
{
  "extends": "solhint:recommended",
  "rules": {
    "func-order": "off",
    "mark-callable-contracts": "off",
    "no-empty-blocks": "off",
    "compiler-version": "off",
    "private-vars-leading-underscore": "error",
    "reason-string": "off",
    "func-visibility": ["error", { "ignoreConstructors": true }]
  }
}
```

_.editorconfig_:

```toml
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = false
max_line_length = 120

[*.sol]
indent_size = 4

[*.js]
indent_size = 2

[*.adoc]
max_line_length = 0
```

### Install Hardhat and create an empty project

Install Hardhat and setup a new HardHat project:

```zsh
yarn add hardhat --dev
npx hardhat
```

Choose `Create an empty hardhat.config.js` for a new empty Hardhat project.

Install [ethers.js](https://docs.ethers.io/v5/), [waffle](https://ethereum-waffle.readthedocs.io/en/latest/matchers.html) and [chai](https://www.chaijs.com/api/bdd/) librairies and plugins:

```zsh
yarn add @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-solhint --dev
```

Configure `hardhat.config.js` for enabling `hardhat-waffle` and `hardhat-solhint` plugins and set your compiler version

```js
require('@nomiclabs/hardhat-waffle')
require('@nomiclabs/hardhat-solhint')

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
  solidity: '0.8.4',
}
```

By enabling the `hardhat-solhint` plugin, `npx hardhat check` task will run `solhint` on your project.

### Install usefull plugins

#### hardhat-docgen

[hardhat-docgen](https://hardhat.org/plugins/hardhat-docgen.html) nous permet de générer de la documentation depuis nos commentaires NatSpec.
Installer hardhat-docgen:

```bash
yarn add --dev hardhat-docgen
```

Charger le plugin dans _hardhat.config.js_ et configurer le plugin sous la clef `docgen`:

```js
require('@nomiclabs/hardhat-waffle')
require('@nomiclabs/hardhat-solhint')
require('hardhat-docgen')

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
  solidity: '0.8.4',
  docgen: {
    path: './docs',
    clear: true,
    runOnCompile: true,
  },
}
```

Ainsi à chaque compilation de notre projet Hardhat un dossier `docs/` contenant la documentation de nos smart contracts selon leurs commentaires NatSpec sera généré.

### Deploy on public networks

Pour déployer sur un réseau public comme le mainnet ou un des testnets nous devons configurer des points d'entrée dans le fichier de configuration _hardhat.config.js_. Les points d'entrée peuvent aussi bien être l'url d'une api ou d'un noeud Ethereum.  
Pour cela nous allons utiliser le service [infura](https://infura.io/) ou [alchemy](https://www.alchemy.com/). L'avantage est que ces services pointent sur plusieurs nodes Ethereum nous offrant ainsi une meilleure qualité de service que si nous étions dépendant que d'un unique noeud Ethereum.

#### Infura

Créer un compte et un nouveau projet sur [infura](https://infura.io/)
Dans le dashboard de votre projet, vous avez accès aux `SETTINGS` qui contiennent le `PROJECT ID` de votre projet.
Vous avez désormais des `endpoints` disponibles sur l'ensemble des réseaux Ethereum qui sont des urls que vous devrez utiliser pour communiquer avec la blockchain Ethereum pour effectuer les déploiements des smart contracts.

#### dotenv and Private key

Maintenant que nous avons accès à la blockchain Ethereum via l'api `infura` il nous faut configurer un account qui puisse effectuer des transactions de déploiements (pas obligatoirement que des déploiements).  
Pour cela il nous faut la clef privée de cet account, et que cet account possède également de l'Ether afin de payer les frais de gas sur le réseau.  
Néanmoins il ne faudra jamais que cette clé privée se retrouve publiée et visible, particulièrement sur Github!!!
Dans de nombreux tutoriels sur Hardhat les clés privées sont écrites directement dans le fichier _hardhat.config.js_, **IL NE FAUT SURTOUT PAS ECRIRE VOS CLEFS PRIVEES DANS LE FICHIER _hardhat.config.js_**.  
Il est recommandé d'écrire les données sensibles dans un fichier _.env_ que l'on ajoutera dans le fichier _.gitignore_ afin qu'il soit ignoré lors du push de votre projet sur Github.  
Ensuite dans le fichier _hardhat.config.js_ nous utiliserons `dotenv` afin de récupérer les valeurs du fichier _.env_, le `INFURA_PROJECT_ID` et la `PRIVATE_KEY` qui nous permettront de configurer _hardhat.config.js_ pour effectuer des transactions de déploiements sur la blockchain Ethereum.

#### Setup for deploying on public network

Installer `dotenv`:

```zsh
yarn add dotenv --dev
```

A la racine de votre projet Hardhat créer un fichier _.env_ qui contient les informations suivantes:
_.env_:

```zsh
INFURA_PROJECT_ID="ENTER INFURA PROJECT ID HERE"
PRIVATE_KEY="ENTER YOUR PRIVATE KEY HERE"
```

Ajouter le fichier _.env_ dans le fichier _.gitignore_:

```zsh
node_modules

#Hardhat files
cache
artifacts

#dotenv
.env
```

Configurer _hardhat.config.js_ afin de récupérer l'`INFURA_PROJECT_ID` et la `PRIVATE_KEY` de l'account qui effectuera les déploiements et rajouter les points d'entrée réseaux comme ci dessous:

```js
require('@nomiclabs/hardhat-waffle')
require('@nomiclabs/hardhat-solhint')
require('hardhat-docgen')

require('dotenv').config()
const INFURA_PROJECT_ID = process.env.INFURA_PROJECT_ID
const PRIVATE_KEY = process.env.PRIVATE_KEY

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
  solidity: '0.8.4',
  networks: {
    mainnet: {
      url: `https://mainnet.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [`0x${PRIVATE_KEY}`],
    },
    ropsten: {
      url: `https://ropsten.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [`0x${PRIVATE_KEY}`],
    },
    kovan: {
      url: `https://kovan.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [`0x${PRIVATE_KEY}`],
    },
    rinkeby: {
      url: `https://rinkeby.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [`0x${PRIVATE_KEY}`],
    },
    goerli: {
      url: `https://goerli.infura.io/v3/${INFURA_PROJECT_ID}`,
      accounts: [`0x${PRIVATE_KEY}`],
    },
  },
  docgen: {
    path: './docs',
    clear: true,
    runOnCompile: true,
  },
}
```

la clef `accounts` est un tableau de clefs privées. Par défaut la première clef privée, `accounts[0]`, est utilisée dans nos scripts.  
la clef `accounts` peut également être une seed phrase: https://hardhat.org/config/#hd-wallet-config  
Si plusieurs accounts sont définis et que nous souhaitons utiliser un account particulier pour nos scripts, il faudra comme pour nos tests unitaires récupérer la liste de tous les signers et appliquer un destructuring assignement:

```js
const [account1, account2, account3] = await ethers.getSigners()
```

Bien qu'il ne soit pas indiqué il existe toujours par défaut un network `localhost` qui correspond au Hardhat network lancé depuis la commande `npx hardhat node`.

#### Write a deployment script

Lorsque nos smart contracts compilent sans erreurs et que tous les tests unitaires sont passés avec succès nous pouvons écrire des scripts de déploiements pour nos smart contracts.  
Le nom des scripts de déploiement doivent suivre cette convention: _deploy-NomDuContract.js_  
Tous les scripts de déploiements doivent être dans le dossier _scripts/_ et doivent suivre la structure ci dessous:
_scripts/deploy-Greeter.js_:

```js
const hre = require('hardhat')

async function main() {
  // Hardhat always runs the compile task when running scripts with its command
  // line interface.
  //
  // If this script is run directly using `node` you may want to call compile
  // manually to make sure everything is compiled
  // await hre.run('compile');

  // Optionnel car l'account deployer est utilisé par défaut
  const [deployer] = await ethers.getSigners()
  console.log('Deploying contracts with the account:', deployer.address)

  // We get the contract to deploy
  const Greeter = await hre.ethers.getContractFactory('Greeter')
  const greeter = await Greeter.deploy('Hello, Hardhat!')

  // Attendre que le contrat soit réellement déployé, cad que la transaction de déploiement
  // soit incluse dans un bloc
  await greeter.deployed()

  // Afficher l'adresse de déploiement
  console.log('Greeter deployed to:', greeter.address)
}

// We recommend this pattern to be able to use async/await everywhere
// and properly handle errors.
main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error)
    process.exit(1)
  })
```

#### deploy

Exécuter le script de déploiement en spécifiant l'un des networks configurés dans _hardhat.config.js_:

```zsh
npx hardhat run scripts/deploy-Greeter.js --network rinkeby
```

Il faut récupérer l'adresse affichée via un `console.log` depuis le scripts de déploiements!!!
Pour une meilleure solution voir ci dessous.

#### deployed.json

Idéalement il faudrait ajouter du code à nos scripts de déploiements afin de garder les informations suivantes dans un fichier:

- nom du smart contract déployé
- nom du réseau sur lequel le smart contract a été déployé
- l'adresse du smart contract déployé

**Exercice**:
Voici la structure attendue d'un exemple de fichier _deployed.json_:

```json
{
  "Greeter": {
    "rinkeby": {
      "address": "0x5FbDB2315678afecb367f032d93F642f64180aa3"
    },
    "kovan": {
      "address": "0xfe0035DD6677D6aA6b876F442f4A4F7dF7a550dd"
    }
  }
}
```

Ecrire du code, et idéalement une fonction, qui après chaque déploiement réussi effectue une mise à jour du fichier _deployed.json_ avec les nouvelles informations.  
le nom du réseau de déploiement est accessible depuis nos scripts de déploiement via la variable `hre.network.name`.

### hardhat-project-template

Utiliser le template: https://github.com/BlockMagnet/hardhat-project-template
N'oubliez d'exécuter la commande `yarn` afin d'installer tous les packages et de créer un fichier `.env` qui suit ce format:
_.env_:

```zsh
INFURA_PROJECT_ID="ENTER INFURA PROJECT ID HERE"
PRIVATE_KEY="ENTER YOUR PRIVATE KEY HERE"
```
