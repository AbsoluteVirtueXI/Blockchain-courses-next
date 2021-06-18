# Smart contract

Précédemment nous avons vu tous les éléments que pouvait posséder un fichier Solidity.
Les éléments les plus intéressants sont évidement les smart contracts qui composent ce fichier.
Ils sont définit avec le keyword `contract`.

## Layout order in a smart contract

Comme précisé dans le [style guide](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout) l'ordre des éléments dans un smart contract suit une convention.

Il faudra déclarer les différents éléments d'un smart contract avec l'ordre suivant:

1. Library usage
2. Type declarations
3. State variables
4. Events
5. constructor
6. Function modifiers
7. Functions

## SmartWallet

Les smart contracts ci dessous utilisent tous les éléments que vous pouvez utiliser dans un smart contract.  
Néanmoins ces smart contracts n'utilisent pas d'`enum` et de `struct`.

_Ownable.sol_:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract Ownable {
    address private _owner;
    constructor(address owner_) {
        _owner = owner_;
    }

    modifier onlyOwner() {
        require(msg.sender == _owner, "Ownable: Only owner can call this function");
        _;
    }

    function owner() public view returns(address) {
        return _owner;
    }
}
```

_SmartWallet.sol_:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

// Pour remix il faut importer une url depuis un repository github
// Depuis un project Hardhat ou Truffle on utiliserait: import "@openzeppelin/contracts/utils/Address.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol";
import "./Ownable.sol";

contract SmartWallet is Ownable {
    // library usage
    using Address for address payable;

    // State variables
    mapping(address => uint256) private _balances;
    mapping (address => mapping (address => uint256)) private _allowances;
    mapping(address => bool) private _vipMembers;
    uint256 private _tax;
    uint256 private _profit;
    uint256 private _totalProfit;

    // Events
    event Deposited(address indexed sender, uint256 amount);
    event Withdrew(address indexed recipient, uint256);
    event Transfered(address indexed sender, address indexed recipient, uint256 amount);
    event VipSet(address indexed account, bool status);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    // constructor
    constructor(address owner_, uint256 tax_) Ownable(owner_) {
        require(tax_ >= 0 && tax_ <= 100, "SmartWallet: Invalid percentage");
        _tax = tax_;
    }

    // modifiers
    // Le modifier onlyOwner a été défini dans le smart contract Ownable


    // Function declarations below
    receive() external payable {
        _deposit(msg.sender, msg.value);
    }

    fallback() external {

    }

    function deposit() external payable {
        _deposit(msg.sender, msg.value);
    }


    function withdraw() public {
        uint256 amount = _balances[msg.sender];
        _withdraw(msg.sender, amount);
    }

    function withdraw(uint256 amount) public {
        _withdraw(msg.sender, amount);
    }

    function approve(address spender, uint256 amount) public {
        require(spender != address(0), "SmartWallet: approve to the zero address");
        _allowances[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
    }

    function transfer(address recipient, uint256 amount) public {
        require(_balances[msg.sender] >= amount, "SmartWallet: Not enough Ether to transfer");
        require(recipient != address(0), "SmartWallet: transfer to the zero address");
        _balances[msg.sender] -= amount;
        _balances[recipient] += amount;
        emit Transfered(msg.sender, recipient, amount);
    }

    function transferFrom(address sender, address recipient, uint256 amount) public {
        require(_allowances[sender][msg.sender] >= amount, "SmartWallet: transfer amount exceeds allowance");
        require(_balances[sender] >= amount, "SmartWallet: Not enough Ether to transfer");
        require(recipient != address(0), "SmartWallet: transfer to the zero address");
        _allowances[sender][msg.sender] -= amount;
        _balances[sender] -= amount;
        _balances[recipient] += amount;
    }


    function withdrawProfit() public onlyOwner {
        require(_profit > 0, "SmartWallet: can not withdraw 0 ether");
        uint256 amount = _profit;
        _profit = 0;
        payable(msg.sender).sendValue(amount);
    }

    function setTax(uint256 tax_) public onlyOwner {
        require(tax_ >= 0 && tax_ <= 100, "SmartWallet: Invalid percentage");
        _tax = tax_;
    }

    function setVip(address account) public onlyOwner {
        _vipMembers[account] = !_vipMembers[account];
        emit VipSet(account, _vipMembers[account]);
    }


    function balanceOf(address account) public view returns (uint256) {
        return _balances[account];
    }

    function allowance(address owner_, address spender) public view returns (uint256) {
        return _allowances[owner_][spender];
    }

    function total() public view returns (uint256) {
        return address(this).balance;
    }

    function tax() public view returns (uint256) {
        return _tax;
    }

    function profit() public view returns(uint256) {
        return _profit;
    }

    function totalProfit() public view returns(uint256) {
        return _totalProfit;
    }

    function isVipMember(address account) public view returns (bool) {
        return _vipMembers[account];
    }

    function _deposit(address sender, uint256 amount) private {
        _balances[sender] += amount;
        emit Deposited(sender, amount);
    }

    function _withdraw(address recipient, uint256 amount) private {
        require(_balances[recipient] > 0, "SmartWallet: can not withdraw 0 ether");
        require(_balances[recipient] >= amount, "SmartWallet: Not enough Ether");
        // version de john avec ternaire
        // uint256 fees = _vipMembers[recipient] ? 0 : _calculateFees(amount, _tax);
        uint256 fees = 0;
        if(_vipMembers[recipient] != true) {
            fees = _calculateFees(amount, _tax);
        }
        uint256 newAmount = amount - fees;
        _balances[recipient] -= amount;
        _profit += fees;
        _totalProfit += fees;
        payable(msg.sender).sendValue(newAmount);
        emit Withdrew(msg.sender, newAmount);
    }


    function _calculateFees(uint256 amount, uint256 tax_) private pure returns (uint256) {
        return amount * tax_ / 100;
    }
}
```

## `using` library

Dans Solidity une `library` s'applique à un type.
Une `library` permet de rajouter des fonctionnalités à un type.

```solidity
using Address for address;
```

Désormais toutes les variables de type `address payable` pourront utiliser les fonctionnalités de la librairie `Address` importée depuis _Address.sol_.

Ensuite nous pourrons utiliser la méthode `sendValue`, méthode accessible à toutes les `address payable` de notre smart contract. Si une `address` n'est pas `payable` il faudra la convertir pour profiter de cette nouvelle fonctionnalités:

```solidity
payable(msg.sender).sendValue(amount);
```

la fonction `sendValue` est définie dans le fichier `Address.sol` que l'on importe:  
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L53

## New type declarations

Avec Solidity nous pouvons déclarer nos propres types pour nos variables.  
Ces nouveaux types sont des énumérations ou des structures.

### enum

#### Principle

Certains langages permettent l'utilisation d'énumérations.
Contrairement à JavaScript, nous pouvons utiliser des énumération en Solidity.  
TypeScript aussi permet la déclaration d'énumérations.  
C'est un concept très pratique lorsque nous connaissons toutes les valeurs possibles que pourrait contenir une variable et de limiter les valeurs que peut prendre une variable .
Par exemple les directions dans un jeu: `Up`, `Down`, `Left`, `Right` ou bien des états possibles d'un programme: `Success`, `Error`, `Loading`.
Les énumérations sont des outils pour rendre le code plus lisible et plus élégants, afin d'exprimer avec des mots des valeurs, et dans le cas particulier de Solidity des `uint8`.

#### Comparison with JavaScript

Pour que notre programme puisse avoir du sens à la lecture en JavaScript nous devrions écrire une gestion de direction de cette manière.

```js
// 0, 1, 2, 3 sont des codes/valeurs renvoyés par la manette de jeu
// en fonction de la direction
const Up = 0
const Down = 1
const Left = 2
const Right = 3

let currentDirection = getCurrentDirectionFromGamePad()

switch (currentDirection) {
  case Up:
    moveUp()
    break
  case Down:
    moveDown()
    break
  case Left:
    moveLeft()
    break
  case Right:
    moveRight()
    break
  default:
    throw new Error('Bad direction')
}
```

Nous associons un nombre (le code retourné par la manette de jeu) à un nom que nous pouvons utiliser ensuite pour référencer la direction.  
Mais nous pouvons écrire du code qui n'aurait aucun sens pour notre application mais qui serait du JavaScript valide comme: `currentDirection = 777`
Avec un type `enum` nous aurions la certitude que `currentDirection` ne pourra prendre que des valeurs connues d'avance, donc cohérente comme `Up`, `Down`, `Left`, `Right`.

#### enum in Solidity

En Solidity `enum` est un type de données qui consiste en un ensemble de valeurs constantes. Ces différentes valeurs représentent différents cas.  
Nous devons déclarer notre nouveau type d'abord pour qu'il soit ensuite disponible dans l'ensemble de notre smart contract:

```Solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract Game {
    // Types declarations
    enum Direction { Up, Down, Left, Right }

    // Storage variables
    // default value of _currentDirection will be Direction.Up
    Direction private _currentDirection;

    function setDirection(Direction direction) public {
        _currentDirection = direction;
    }

    function getDirection() public view returns(Direction) {
        return _currentDirection;
    }
}
```

Notre variable de storage `_currentdirection` est de type `Direction`.  
La fonction `setDirection` prend comme paramètre une variable de type `Direction`.  
La fonction `getDirection` retourne une variable de type `Direction`.  
En réalité à la compilation les variables d'un type `enum` sont converties en `uint8` avec le premier élément de l'enum qui est à 0, le suivant à 1, etc...
Mais dans notre smart contract il faudra absolument utiliser le type de l'`enum` lorsqu'il est attendu pour une assignation avec l'opérateur `=`, un passage en paramètre d'une fonction ou pour la valeur de retour d'une fonction.  
Nous pouvons vérifier cela en observant l'abi du smart contract `Game`:

```json
[
  {
    "inputs": [],
    "stateMutability": "nonpayable",
    "type": "constructor"
  },
  {
    "inputs": [],
    "name": "getDirection",
    "outputs": [
      {
        "internalType": "enum Game.Direction",
        "name": "",
        "type": "uint8"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "enum Game.Direction",
        "name": "direction",
        "type": "uint8"
      }
    ],
    "name": "setDirection",
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  }
]
```

#### Passing and returning enum types from functions.

Comme remarqué précédemment, dans nos smart contracts il faudra absolument que le type enum soit respecté pour nos appels de fonctions, ainsi dans notre code Solidity il faudra utiliser le type `enum` pour le type des arguments à passer à une fonction qui attend une `enum` comme paramètre. Cela s'applique aussi a des fonctions d'un smart contract distant:

```solidity
pragma solidity ^0.8.0;
import "./Game.sol";

contract ExternalContract {
    Game private _game;
    constructor(address gameAddress) {
        _game = Game(gameAddress);
    }

    function setGameDirection(Game.Direction direction) public {
        _game.setDirection(direction);
    }

    function getGameDirection() public view returns(Game.Direction){
        return _game.getDirection();
    }
}
```

Pour des appels offchain/web3 c'est différent.
Avec `ethers.js` il faudra passer un nombre (pas un Big number) pour appeler des fonctions qui prennent une `enum` comme paramètre, et la valeur de retour d'une fonction qui retourne une enum sera convertie en nombre aussi coté web3.

```js
let direction = await game.getDirection()
console.log(typeof direction) // print "number"
await game.setDirection(3) // set _direction à Direction.Down
direction = await game.getDirection() // direction == 3
```

#### conversion

Les conversions implicites ne sont pas possibles, les codes suivants entraineront une erreur à la compilation:

```solidity
Direction dir1 = _currentDirection + 1;
Direction dir2 = _currentDirection + Direction.Up;
Direction dir3 = Direction.Up + Direction.Down;
setDirection(3); // Le nombre 3 est différent de Direction.Down
```

Si nous voulons effectuer des opérations arithmétiques sur les variables d'un type enum, si cela à du sens dans notre code, il faudra convertir cette variable explicitement en nombre:

```solidity
uint8 code1 = uint8(_currentDirection) + 1;
2 == uint8(Direction.Left); // true
```

La conversion inverse, un nombre vers l'élément d'une `enum` est possible aussi:

```solidity
Direction dir1 = Direction(0) // dir1 == Direction.Up
Direction dir2 = Direction(1) // dir2 == Direction.Down
Direction dir3 = Direction(2 + 1) // dir3 == Direction.Right
```

### struct

L'autre moyen de créer ses propres types sont les `struct`.  
Les `struct` servent à grouper un ensemble de variables qui ont du sens, pour ainsi donner des propriétés à une donnée que l'on souhaite exprimer en code.  
Une `struct` se rapproche d'une `class` en JavaScript, mais contrairement à une `class` JavaScript, une `struct` en Solidity ne possède pas explicitement de constructor et ne possède pas de méthodes.

#### struct declaration

Le keyword `struct` sert à déclarer un nouveau type, donc comme pour une `enum` il faut d'abord déclarer notre nouveau type avant de l'utiliser:

```solidity
// Type declarations
enum Course {Frontend, Backend, Blockchain, Security}

struct Student {
    Course course;
    bool isActive;
    bool isCertified;
    uint256 certificationDate;
    uint256 certificationDateLimit;
    uint256 age;
    uint256 registrationDate;
    string firstName;
    string lastName;
    string githubUrl;
}

// Storage variable
mapping(address => Student) private _students;
```

`Student` est un nouveau type que nous pouvons désormais utiliser dans notre smart contract.
Il possède 10 propriétés:

- `course` qui est de type `Course` qui correspond au module suivit par un étudiant
- `isActive` qui est de type `bool` pour déterminer si un étudiant est en cours de formation
- `isCertified` qui est de type `bool` pour déterminer si un étudiant a été certifié
- `certificationDate` qui est de type `uint256` qui correspond à la date de certification de l'étudiant
- `certificationDateLimit` qui est de type `uint256` qui correspond à la date limite à laquelle un étudiant pourra passer la certification
- `age` qui est de type `uint256` qui correspond à l'âge de l'étudiant
- `registrationData` qui est de type `uint256` qui correspond à la date d'inscription de l'étudiant
- `firstName` qui est de type `string` qui correspond au prénom de l'étudiant
- `lastName` qui est de type `string` qui correspond au nom de famille de l'étudiant
- `githubUrl` qui est de type `string` qui correspond à l'url du Github de l'étudiant

Puisque nous pourrions avoir plusieurs étudiants, et qui sont tous différents, une variable de storage pertinente serait un `mapping(address => Student)` afin de lier dans notre smart contract l'adresse Ethereum d'un étudiant à une structure `Student` qui contient toutes les informations le concernant.

```solidity
mapping(address => Student) private _students;
```

#### Using struct types

Maintenant que notre type `Student` est déclaré nous pouvons définir des étudiants, pour ensuite les stocker dans la variable de storage `_students`.  
Créons une fonction `register` qui permet à un nouvel étudiant de s'enregistrer:

```solidity
function register(
    string memory firstName,
    string memory lastName,
    string memory githubUrl,
    uint256 age,
    Course course
) public {
    _students[msg.sender] = Student({
        firstName: firstName,
        lastName: lastName,
        githubUrl: githubUrl,
        age: age,
        course: course,
        isActive: true,
        isCertified: false,
        certificationDate: 0,
        certificationDateLimit: block.timestamp + 200 days,
        registrationDate: block.timestamp
    });
}

function getStudent(address account) public view returns (Student memory) {
    return _students[account];
}
```

Il existe 2 autres façons de déclarer une variable de type struct:

    Course course;
    bool isActive;
    bool isCertified;
    uint256 certificationDate;
    uint256 certificationDateLimit;
    uint256 age;
    uint256 registrationDate;
    string firstName;
    string lastName;
    string githubUrl;

```solidity
// 2nd way: Passer les arguments dans le même ordre que les propriétés de la struct
_students[msg.sender] = Student(
    course,
    true,
    false,
    0,
    block.timestamp + 200 days,
    age,
    block.timestamp,
    firstName,
    lastName,
    githubUrl
);

// 3rd way: Passer par une variable initialisé avec des valeurs par défauts.
// Pour ensuite initialiser les propriétés qui nous intéressent une par une.
// Cette méthode est obligatoire si notre struct contient un mapping
Student memory student;
student.firstName = firstName;
student.lastName = lastName;
// etc...
_students[msg.sender] = student;
```

#### Using struct variables

Toutes les propriétés d'une struct sont accessibles par un `.`.
Nous pouvons ainsi lire ou modifier les éléments d'une structure:

```js
function isCertified(address account) public view returns(bool) {
    return _students[account].isCertified;
}

function certify(address account) public OnlyInstructor {
    _students[account].isCertified = true;
}
```

#### Passing and returning enum types from functions.

Comme pour une `enum`, dans nos smart contracts il faudra absolument que le type de la structure soit respecté pour nos appels de fonctions, ainsi dans notre code Solidity il faudra utiliser le type de la structure pour le type des arguments à passer à une fonction qui attend une structure comme paramètre. Cela s'applique aussi a des fonctions d'un smart contract distant.

La particularité est pour les appels offchain/web3.
Si nous analysons la partie de l'abi qui correspond à la fonction `getStudent` qui retourne un struct nous obtenons:

```json
{
  "inputs": [
    {
      "internalType": "address",
      "name": "addr",
      "type": "address"
    }
  ],
  "name": "getStudent",
  "outputs": [
    {
      "components": [
        {
          "internalType": "enum School.Course",
          "name": "course",
          "type": "uint8"
        },
        {
          "internalType": "bool",
          "name": "isActive",
          "type": "bool"
        },
        {
          "internalType": "bool",
          "name": "isCertified",
          "type": "bool"
        },
        {
          "internalType": "uint256",
          "name": "certificationDate",
          "type": "uint256"
        },
        {
          "internalType": "uint256",
          "name": "certificationDateLimit",
          "type": "uint256"
        },
        {
          "internalType": "uint256",
          "name": "age",
          "type": "uint256"
        },
        {
          "internalType": "uint256",
          "name": "registrationDate",
          "type": "uint256"
        },
        {
          "internalType": "string",
          "name": "firstName",
          "type": "string"
        },
        {
          "internalType": "string",
          "name": "lastName",
          "type": "string"
        },
        {
          "internalType": "string",
          "name": "githubUrl",
          "type": "string"
        }
      ],
      "internalType": "struct School.Student",
      "name": "",
      "type": "tuple"
    }
  ],
  "stateMutability": "view",
  "type": "function"
}
```

On remarque que l'`internal type` de la valeur de retour est bien `struct School.Student`.  
Dans une abi l'`internal type` correspond au type côté smart contract.
Mais on remarque le que le `type` est un `tuple`, donc une liste.
Dans une abi le `type` est comment les call extérieur devront interpréter ce type.

Vérifions avec `ethers.js` comment est récupéré cette structure `Student` lors d'un appel à `getStudent``

```js
const [deployer, student1] = await ethers.getSigners()
const SchoolMagnet = await ethers.getContractFactory('SchoolMagnet')
const schoolMagnet = await SchoolMagnet.connect(deployer).deploy()
await schoolMagnet.deployed()

// Register student1 as a student
await schoolMagnet
  .connect(student1)
  .register('Sofiane', 'Akermoun', 'https://github.com/AbsoluteVirtueXI', 39, 3)
// get student1 information
const student1Info = await schoolMagnet.getStudent(student1.address)
console.log(student1Info)
```

output:

```js
[
  3,
  true,
  false,
  BigNumber { _hex: '0x00', _isBigNumber: true },
  BigNumber { _hex: '0x61d3ef26', _isBigNumber: true },
  BigNumber { _hex: '0x27', _isBigNumber: true },
  BigNumber { _hex: '0x60cc4326', _isBigNumber: true },
  'Sofiane',
  'Akermoun',
  'https://github.com/AbsoluteVirtueXI',
  course: 3,
  isActive: true,
  isCertified: false,
  certificationDate: BigNumber { _hex: '0x00', _isBigNumber: true },
  certificationDateLimit: BigNumber { _hex: '0x61d3ef26', _isBigNumber: true },
  age: BigNumber { _hex: '0x27', _isBigNumber: true },
  registrationDate: BigNumber { _hex: '0x60cc4326', _isBigNumber: true },
  firstName: 'Sofiane',
  lastName: 'Akermoun',
  githubUrl: 'https://github.com/AbsoluteVirtueXI'
]
```

Si une fonction Solidity retourne une structure, on pourra donc au choix interprété la valeur de retour comme un tableau, avec tous les élements dans le même ordre que les propriétés de la strucutre, ou sinon un objet littéral qui possède les mêmes propriétés que notre structure.

#### Few words on optimization

## Storage: state variables

Les variables d'états sont persistantes et sont stockées de manière permanente dans le `storage` du smart contract.
Le `storage` est un espace mémoire que l'on pourrait comparer à un disque dur.
A leur déclaration Les variables d'états se composent:

- d'un type
- un attribut pour déclarer la variable `constant` ou `immutable` si nécéssaire
- une visibilité (`public`, `private` ou `internal`)
- un identifier (un nom)
- et optionnellement cette déclaration est suivie d'une initialisation avec l'opérateur `=` pour initialiser la variable dés sa déclaration.

### variable types

Pour l'instant nous ne travaillons qu'avec des `uint256`, des `bool`, des `address` et des `mapping`.

### constant and immutable variables

#### `constant`

Les variables déclarées comme constantes doivent absolument être initialisées au moment de la compilation.
La valeur d'une variable constante doit être connue avant le déploiement du smart contract.
La valeur d'une variable constante ne pourra jamais être modifiée.

```solidity
uint256 constant private MAX_PLAYERS = 255; // obligation d'initiliaser la variable
```

Une variable constante est le seul type de variables qui peut exister à l'extérieur d'une smart contract.  
Comme dans de nombreux languages, la convention pour nommer une constante est de l'écrire entièrement en majuscule et avec `_` qui séparent les mots qui la compose.

#### `immutable`

Les variables déclarées comme `immutable` sont un peu moins restrictives que les variables constantes.
A la différence des variables constantes elles n'ont pas l'obligation d'être initialisées au moment de la compilation, mais doivent absolument l'être dans le constructor du smart contract.  
Une fois initialisée une variable `immutable` ne pourra plus jamais être modifiée.

```solidity
uint256 immutable private _cap;
/**
  * @dev Sets the value of the `cap`. This value is immutable, it can only be
  * set once during construction.
  */
  constructor (uint256 cap_) {
    require(cap_ > 0, "ERC20Capped: cap is 0");
    _cap = cap_;
  }
```

Une variable `immutable` est utile lorsqu'on ne connaitra sa valeur qu'au moment de la construction du smart contract, valeur qui sera passée en paramètre du constructeur et qui initialisera la variable `immutable`.

### identifier

Une variable possède un nom qui doit obligatoirement commencer par une lettre, un signe `$` ou un `_` et peut contenir des nombres après le premier symbole.

### Visibility of state variables

Nous respectons la convention d'[OpenZepplin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/GUIDELINES.md#solidity-code):  
**Toutes les variables de storage doivent être `private` ou `internal`, elles devront toutes commencer par un `_` et si les valeurs doivent être accessibles depuis l'extérieur du smart contract il faudra créer des getter associés.**

```solidity
uint256 private _tax;

function tax() public view returns (uint256) {
  return _tax;
}
```

### Variable initialization

Une variable peut être initialisée lors de sa déclaration, dans le constructeur ou obtenir une valeur par défaut en fonction de son type si elle n'est pas initialisé avec l'opérateur `=`:

- `bool`: `false`
- `int` et `uint`: `0`
- `string`: `""`
- `enum`: premier élement de l'enum
- `address`: `0x0`, il faut utiliser `address(0)` pour exprimer cette adresse. On l'appelle `zero-address`
- `mapping(key => value)`: Par défaut les clefs du mapping seront associés à la valeur par défaut du type de `value`
- `struct`: Tous les membres de la structure auront pour valeur par défaut celui de leur type.
- dynamically-sized array: `[]`
- fixed-sized array: un tableau dont tous les éléments seront initialisés à la valeur par défaut du type du tableau.

## Event

Les events permettent d'écrire dans le journal de l'EVM.  
Ils sont utiles pour des applications off-chain qui souhaitent être notifiées d'événements qui se produisent dans notre smart contract ou dans la Blockchain en général.  
Les fonctions qui modifient les variables d'états ne peuvent pas communiquer avec des applications off-chain grâce à leurs valeurs de retour, pour cela elles doivent utiliser des `event`.  
Emettre des `event` permettra aussi à des applications de parcourir le journal pour des événements particuliers et consulter ainsi l'historique de tous les événements d'un type particulier émit par un smart contract.  
Un filtre sur des événements peut être appliqués sur des arguments de l'`event` qui sont déclarés comme `indexed`.

Pour déclarer un `event`:

```solidity
event Deposited(address indexed sender, uint256 amount);
```

`Desposited` est un event qui prend comme paramètre une `address` et un `uint256`.  
le paramètre `sender` est `indexed`, on pourra donc depuis notre frontend écouter l'`event` `Deposited` pour une adresse particulière. Une recherche de tous les `event` `Deposited` passés pourra également être effectuée en filtrant sur l'adresse qui aura déposé des fonds.

Ensuite depuis l'une de nos fonctions il faudra émettre cet `event` avec `emit` en passant les arguments à l'`event`:

```solidity
function _deposit(address sender, uint256 amount) private {
    _balances[sender] += amount;
    emit Deposited(sender, amount);
}
```

Ci dessus un `event` est émis dès qu'un deposit est effectué.  
Notre front React pourra réagir à cet `event` s'il l'écoute, par exemple pour créer une notification d'un deposit réussi par notre utilisateur.

La convention sur les `event`:

- Le nom de `event` doit commencer par une majuscule
- Le nom de `event` est un verbe qui doit être au passé
- L'`event` doit immédiatement être émit après un changement d'état qu'il représente

## modifier

Les `function modifiers` servent à modifier le comportement d'une fonction.  
En pratique les `modifier` servent à exécuter ou non le corps d'une fonction selon l'évaluation d'un `require`.  
Déclaration d'un `modifier` qui exécutera le corps d'une fonction si et seulement si le `msg.sender` est égal à l'adresse qui serait stockée dans la variable d'état `_owner`:

```solidity
modifier onlyOwner() {
    require(msg.sender == _owner, "Ownable: Only owner can call this function");
    _;
}
```

Déclaration d'un `modifier` qui exécutera le corps d'une fonction si et seulement le `msg.value` est supérieur à 10 ether lors de l'appel de la fonction qui utilisera ce modifier:

```solidity
modifier moreThan10Ether() {
    require(msg.value > 10 ether, "Contract: Not enough ether");
    _;
}
```

le `_;` permet de continuer le flot d'exécution et de passer à l'exécution du `modifier` suivant ou d'exécuter le corps de la fonction si il n'y a plus de `modifier`s

Avant l'exécution du corps d'une fonction les `modifier`s sont exécutés dans l'ordre comme ils apparaissent dans la déclaration de la fonction.

```solidity
modifier firstModifier() {
    // some require here
    // _;
}

modifier secondModifier() {
    // some require here
    // _;
}

modifier thirdModifier() {
    // some require here
    // _;
}

function doSomething() public payable firstModifier secondModifier thirdModifier {
    // do something
}
```

Les `modifier`s ont accès aux variables globales comme `msg.sender` et `msg.value`, mais ne peuvent avoir accès implicitement aux arguments passés à la fonction qu'ils "modifient". Si un `modifier` à besoin d'avoir accès aux paramètres de la fonction qu'il modifie il faudra dans ce cas passer les arguments de la fonctions en paramètres du modifier.

```solidity
modifier onlyOwner() {
    require(msg.sender == _owner, "Ownable: Only owner can call this function");
    _;
}

modifier onlyGoodPercentage(uint256 percentage) {
    require( percentage >= 0 && percentage <= 100, "Contract: Not a valid percentage");
    _;
}

function setPercentage(uint256 percentage) public onlyOwner onlyGoodPercentage(percentage) {
    // do something
}
```

Les modifiers déclarés dans des contract parents peuvent être utilisés par des smart contract enfants qui héritent de ce smart contract parent.
C'est pour cela que dans notre smart contract `SmartWallet` nous pouvons utiliser le modifier `onlyOwner` déclaré dans le contract `Ownable`.

## function

Dans un smart contract tout le code exécutable est écrit dans des fonctions.
Les interactions que nous effectuerons avec notre smart contract ne se fera que via ces fonctions.

### function declarations order

Comme précisé dans le [style guide](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-functions) l'ordre des déclarations des fonctions dans un smart contract suit une convention.

1. constructor
2. receive function (if exists)
3. fallback function (if exists)
4. external
5. public
6. internal
7. private

Au sein des groupes de fonctions qui sont `external`, `public`, `internal` et `private` il faudra placer les fonctions qui ont un modifier `view` et `pure` en dernières.

### constructor

Le constructor est la fonction qui sera exécutée au moment du déploiement de notre smart contract.
Le constructor sert à initialisé les variables d'état de notre smart contract.
Si une fonction ne possède pas de constructor alors un constructor vide est automatiquement créé.
Si un constructor possède des paramètres, il faudra passer les bons arguments à notre constructor
Si notre smart contract hérite d'un parent qui possède un constructor il faudra initialiser le contract parent en appelant son constructor lors du déploiement de notre smart contract enfant.

```solidiy
constructor(address owner_, uint256 tax_) Ownable(owner_) {
    require(tax_ >= 0 && tax_ <= 100, "SmartWallet: Invalid percentage");
    _tax = tax_;
}
```

### receive and fallback

#### receive Ether function

Un smart contract peut recevoir directement de l'ether, comme on en enverrait à une autre adresse depuis notre wallet metamask.  
Pour que notre smart contract puisse accepter de l'Ether par un `transfer` il faut implémenter la fonction `receive`.

```solidity
receive() external payable {
    _deposit(msg.sender, msg.value);
}
```

Au moment de la réception de l'ether la fonction `receive` s'exécute.
Dans le pire des scénarios la fonction `receive` ne pourra utiliser que 2300 unités de gas pour s'exécuter.
2300 unités de gas c'est juste assez pour emetre un `event`.
Le pire des scénarios peut se produire lorsqu'un smart contract essaye d'envoyer directement de l'ether à un autre smart contract via la fonction `transfer`.

```solidity
// Call depuis un autre smart contract avec transfer provequera un `Out of gas` error
payable(adressOfSmartWallet).transfer(1 ether)
```

Il ne peut exister qu'une seule fonction `receive` par contract et elle doit obligatoirement être `external`.  
Pour plus d'information: https://docs.soliditylang.org/en/latest/contracts.html?highlight=receive#receive-ether-function

#### fallback function

La `fallback` est une fonction qui sera exécutée si notre contract est appelé avec une fonction qui n'existe pas. Elle servait initialement à la réception d'ether avant que la fonction `receive` fasse son apparation.  
Il ne peut exister qu'une seule fonction `fallback` par contract et elle doit obligatoirement être `external`
Pour plus d'information: https://docs.soliditylang.org/en/latest/contracts.html?highlight=receive#fallback-function

### function declaration

La grammaire d'une déclaration de fonction:  
https://docs.soliditylang.org/en/latest/grammar.html#a4.SolidityParser.functionDefinition

Une fonction est composée:

1. d'un nom son `identifier`
2. d'une liste de paramètres
3. un paramètre de visibilité: `external`, `public`, `internal`, `private`
4. de paramètre de `state-mutability`: `pure`, `view`, `payable` ou aucun
5. des `modifer`s
6. du type de retour de cette fonction, si elle retourne une valeur
7. le corps de la fonction entre accolades

Préférez cette ordre là au moment de la déclaration de votre fonction.

### visibility

Une fonction peut être
`external`: seulement accessible depuis l'extérieur  
`public`: la visibilité la plus permissive, accessible depuis l'extérieur et depuis notre smart contract
`internal`: accessible depuis notre smart contract et les smart contract qui hériteront de notre smart contract
`private`: la visibilité la plus restrictive, accessible seulement depuis notre smart contract, et même pas les smart contract qui hériteront de notre smart contract.

### state-mutability specifiers

Une fonction déclarée comme `pure` ne pourra ni lire ni écrire les variables d'états de notre smart contract et les variables globales comme `msg.sender` et `msg.value`. Elle ne pourra dépendre que de ses paramètres pour s'exécuter.

Une fonction déclarée comme `view` ne pourra pas écrire/modifier les variables d'états de notre smart contract mais pourra les les lire. Une fonction `view` peut également lire les variables globales comme `msg.sender` et `msg.value`.

Si `pure` ou `view` ne sont pas spécifiés alors notre fonction peut lire et écrire dans le `storage` de notre smart contract.  
Si `payable` est spécifié alors on se retrouve avec une fonction qui peut lire et écrire dans le `storage` et qui peut aussi accepter de l'ether.

### function modifiers

Voir chapitre `modifer`

### Global variables

Depuis une fonction nous pouvons avoir accès à des variables et fonctions globales.  
Ces propriétés peuvent nous renseigner sur la transaction en cours.
Ces fonctions et variables peuvent être directement utilisées dans nos fonctions.
Par exemple `msg.sender` est une variable qui contient l'adresse de l'account (ou du smart contract) qui effectue le call de la fonction et `msg.value` est la quantité en wei passé à la fonction lors de son appel (via le champ `value`).  
Pour plus d'information:  
https://docs.soliditylang.org/en/latest/units-and-global-variables.html?highlight=global%20variables#special-variables-and-functions et https://docs.soliditylang.org/en/latest/cheatsheet.html?highlight=global%20variables#global-variables

### function overloading

Des fonctions peuvent avoir le même nom si elles ont des signatures différentes.
La signature d'une fonction est composée par

- ses paramètres: leur nombres, leur types
- la valeur de retour: leur nombre, leur types

Pour en savoir plus: https://docs.soliditylang.org/en/latest/contracts.html?highlight=overloading#function-overloading
