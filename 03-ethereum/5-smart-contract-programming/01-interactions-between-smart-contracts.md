# Interactions between smart contracts

## Smart contracts calling functions of other smart contracts.

### Principles

Un smart contract peut appeler les fonctions d'un autre smart contract.
Un smart contract `Caller` sera en mesure d'appeler un smart contract `Callee` si:

- `Caller` connait l'adresse de l'instance `Callee` déployée
- `Caller` possède l'interface ou le code de `Callee` afin de savoir comment communiquer avec ce dernier.

Sur Ethereum un smart contract est une adresse à laquelle est associée un espace de stockage, (nos variables de `storage`) et du `bytecode` qui correspond à tout le code exécutable de nos fonctions.  
Ce `bytecode` a une structure, et les fonctions sont identifiées par les 4 premiers bytes du hash `Keccak256` (SHA3) de leur signature.  
On appelle ce hash le [function selector](https://docs.soliditylang.org/en/v0.8.4/abi-spec.html#function-selector).  
Dans le `bytecode` ce `function selector` sera l'`entry point` pour accéder au code exécutable de la fonction.  
Techniquement lorsque `Caller` souhaitera appeler une fonction de `Callee`, il exécutera le `bytecode` correspondant au `Keccak hash` de la signature de la fonction, le `function selector`, à l'adresse de déploiement de `Callee`.  
Solidity est un langage de haut niveau, et nous n'avons pas à calculer nous même le hash des signatures des fonctions, ils sont calculés automatiquement à la compilation de notre smart contract, mais pour cela nous devrons lui fournir la signature de la fonction, c'est à dire son nom et le type des paramètres de cette fonction.  
C'est pour cela qu'un token est considéré comme un ERC20 si il implémente au minimum l'[EIP-20](https://eips.ethereum.org/EIPS/eip-20).  
Effectivement tous les ERC20 possèdent les mêmes signatures pour les fonctions définies dans l'`EIP-20`, donc les mêmes hash de fonctions, donc les même `function selectors`.

### Function signatures and function selectors

Sur Ethereum la signature d'une fonction est son nom suivit entre parenthèses du type de ses paramètres.
Un token est considéré comme un ERC20 si il implémente parfaitement EIP-20 donc qu'il possède au moins des fonctions qui ont les signatures suivantes:

```text
name()
symbol()
decimals()
totalSupply()
balanceOf(address)
transfer(address,uint256)
transferFrom(address,address,uint256)
approve(address,uint256)
allowance(address,address)
```

Pour être 100% ERC20 compliant il faut aussi implémenter les events `Transfer` et `Approval`.  
On retrouve la notion d'`interface` en Solidity, qui oblige le contract qui en hérite à implémenter les fonctions de cette interface.  
Par exemple l'implémentation d'un [ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol) par OpenZepplin hérite de [IERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol) qui est l'interface à implémenter.  
En Solidity hériter d'une interface oblige le développeur à implémenter toutes les fonctions de cette interface dans un smart contract.
En héritant de `IERC20` le smart contract `ERC20` doit implémenter toutes les fonctions déclarées, mais non définies.  
Comme on le remarque dans l'interface `IERC20` on peut récupérer les signatures des fonctions d'un smart contract avec seulement son interface, donc aussi ses `function selectors`.  
Donc depuis Solidity nous pouvons coder des appels de fonctions en connaissant au minimum l'interface du smart contract, `Callee`, que l'on souhaite appeler depuis notre smart contract `Caller`, ou au maximum le code complet de `Callee`.  
En Solidity `Caller` devra déclarer une variable de type `Callee` et lui associer l'adresse où a été déployé `Callee`.
Il faudra importer le smart contract `Callee` ou son interface dans `Caller` pour cela.

### Import a Contract / Interface and declare a contract type.

_IGreeter.sol_:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IGreeter {
    function sayHello() external pure returns (string memory);

    function sayGoodbye() external pure returns (string memory);
}
```

_Greeter.sol_:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./IGreeter.sol";

contract Greeter is IGreeter {
    function sayHello() public pure override returns (string memory) {
        return "Hello world!";
    }

    function sayGoodbye() public pure override returns (string memory) {
        return "Goodbye";
    }
}
```

_Say.sol_:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// import "./Greeter.sol" works too
import "./IGreeter.sol";

contract Say {
    IGreeter private _greeter;

    constructor(address greeterAddress) {
        _greeter = IGreeter(greeterAddress);
    }

    function hello() public view returns (string memory) {
        return _greeter.sayHello();
    }

    function bye() public view returns (string memory) {
        return _greeter.sayGoodbye();
    }
}
```

Dans les exemples précédents `Greeter` implémente l'`interface` `IGreeter`.  
Ce qui nous intéresse c'est comment le smart contract `Say` arrive à appeler les fonctions du smart contract `Greeter`.  
Dans le smart contract `Say`, nous importons le fichier `IGreeter.sol` qui contient l'interface `IGreeter`, le smart contract `Greeter` que nous souhaitons appeler implémente cette interface.  
Nous aurions aussi pût importer `Greeter.sol` dans notre cas.

```solidity
import "./IGreeter.sol";
```

Ensuite nous déclarons une variable de storage qui sera de type `IGreeter`:

```solidity
IGreeter private _greeter;
```

Nous initialisons ensuite cette variable dans le constructor. Le constructor a comme paramètre l'adresse du smart contract `Greeter` déjà déployé:

```solidity
constructor(address greeterAddress) {
    _greeter = IGreeter(greeterAddress);
}
```

Nous pouvons ensuite appeler les fonctions de `Greeter` directement via la variable de type `IGreeter`:

```solidity
_greeter.sayHello();
_greeter.sayGoodbye();
```

## Order of deployment and testing

On remarque que `Say` est dépendant de `Greeter`, car l'adresse de `Greeter` doit être connue avant que `Say` soit déployé. En effet l'adresse de `Greeter` doit être passée en paramètre du constructor de `Say`.
Cela s'applique également à nos tests. Nous pourrons tester `Greeter` en standalone car il n'a aucune dépendance, par contre pour tester `Say` il faudra aussi que l'on déploie `Greeter` dans nos [tests](https://github.com/BlockMagnet/interactions-contracts/blob/main/test/Say-test.js#L10).

## Exercice: ICO & Calculator

1. Créer un ERC20 en utilisant les librairies d'OpenZepplin.

2. Assigner la total supply à un utilisateur qui possédera tous les tokens, c'est l'owner.

3. Créer un smart contract `ICO` qui permettra à des utilisateurs d'acheter les tokens de l'owner.

   - 1 Token (1 x 10^18) pourra être obtenu pour 1 gwei.
   - Les utilisateurs auront 2 possibilités pour acheter ces tokens:
     - Envoyer directement les ethers qui seront réceptionnés par une fonction `receive` (déjà vu en cours)
     - Utiliser une fonction `buyTokens()` qui sera `payable` afin de supporter l'envoi d'ethers.
   - Le smart contract devra posséder un owner, vous pouvez utiliser `Ownable.sol` que nous avons déjà vu en cours, ou encore mieux utiliser [Ownable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol). Lire https://docs.openzeppelin.com/contracts/4.x/api/access#Ownable pour plus d'information.
   - L'ICO devra durer 2 semaines à partir du déploiement du smart contract. Après 2 semaines la vente de token sera clôturée.
   - L'owner pourra `withdraw` toute la balance du smart contract `ICO` lorsque l'ICO sera terminée, c'est à dire 2 semaines après le déploiement.
   - Ajouter des getters utilitaires pour obtenir le montant des `wei` gagnés, le nombre de tokens vendus, le prix en wei d'un token etc...
   - Ajouter des events.

4. Créer un smart contract `Calculator` qui nécessitera 1 de vos tokens pour effectuer l'une des 5 opérations arithmétiques d'addition, soustraction, multiplication, division et modulo. Ces fonctions devront retourner le résultat de l'opération et aussi émettre un `event` afin que l'on puisse récupérer ce résultat depuis notre Frontend.
   Par exemple la fonction `add` devra être déclarée ainsi:

```solidity
function add(int256 nb1, int256 nb2) public returns(int256) {
    // insérer votre code ici
}
```

Vous pouvez y ajouter des `modifier`s si c'est nécéssaire.  
Ce smart contract possédera un owner qui pourra `withdraw` les tokens accumulés par le smart contract.

5. Vous devrez fournir les tests unitaires qui justifieront du bon fonctionnement de vos smart contracts.
   **Attention à bien penser à `approve` le smart contract de l'ICO en tant qu'`owner` des tokens afin d'autoriser le smart contract de l'ICO à transférer les tokens aux acheteurs dans vos tests et aussi à `approve` le smart contract Calculator en tant qu'utilisateur pour que le smart contract Calculator puisse utiliser la fonction `transferFrom`**.

## Smart contracts deploying smart contracts

## Use cases

### ICO

### Decentralized Exchange
