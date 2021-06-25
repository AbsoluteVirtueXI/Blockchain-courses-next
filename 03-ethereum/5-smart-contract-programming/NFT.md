## ERC721: NFT contract

### NFT vs FT

NFT pour "Non-Fongible Token" est un type de token qui se caractérise par son unicité.  
Dans un smart contract de tokens ERC20, tous les tokens émis sont indifférenciables des autres. Nous pourrions les échanger avec d'autres tokens (émis depuis le même smart contract), leur valeur reste la même.  
Cela s'exprime dans la structure même du code d'un ERC20:

```solidity
mapping(address => uint256) private _balances;
```

Dans un ERC20 ce que l'on veut journaliser sont les quantités associés à une adresse Ethereum, et nous l'avons vu, la notion de "token" est juste utilisé pour leur répresentation "humaine", en réalité les tokens ERC20 ne sont qu'une certaine quantité d'une unité plus petite (comme le wei pour l'ether).  
Dans un ERC721 c'est différent, nous souhaitons rendre tous les tokens différenciables et non divisibles en plus petites quantités.  
l'unicité est obtenu en ajoutant un id à chacun des tokens.
Le NFT lie un id unique à une information rendant la division de ce NFT incohérente, on ne peut pas parler de moitié d'information.
On a besoin de 2 mappings pour assurer le suivi d'un NFT et d'un 3eme mapping optionnel pour lier ce token à son contenu (sa valeur):

```solidity
// Mapping from token ID to owner address
mapping(uint256 => address) private _owners;

// Mapping owner address to token count
mapping(address => uint256) private _balances

// Mapping token ID to Information
// Information is a user defined structure
mapping(uint256 => Information) private _informations
```

### Usage in the ecosystem

Les NFT sont utilisés pour identifier un bien ou une donnée.  
On peut les retrouver dans les jeux videos pour associer la propriété d'un asset du jeu à un joueur, dans l'art, surtout numérique, les diplômes, et de plus en plus des tentatives pour associer des NFT à des biens matériels. Evidement pour ce dernier cas une autorité de confiance extérieure à la blockchain est obligatoire.  
Concrétiser la propriété d'une ressource par un NFT permet donc très facilement l'échange et la vente de ce bien. Suite à l'échange ou la vente d'un NFT, la propriété de la ressource est également transféré au nouvel owner.

### EIP-721

Le standard ERC-721 est défini dans l'EIP-721: https://eips.ethereum.org/EIPS/eip-721.  
Et on peut retrouver l'implémentation de ce standard par Openzepplin sur leur repository Github: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol  
Une documentation de leur smart contract est disponible sur: https://docs.openzeppelin.com/contracts/4.x/erc721  
L'api de l'ERC-721 d'OpenZepplin est documentée ici: https://docs.openzeppelin.com/contracts/4.x/api/token/erc721

### ERC721 from OpenZepplin

Live coding:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";

contract GameLoot is ERC721, AccessControl {
    using Counters for Counters.Counter;

    enum LootType {
        Weapon,
        Armor
    }

    struct Loot {
        LootType lootType;
        uint256 attk;
        uint256 def;
        string name;
    }

    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    Counters.Counter private _lootIds;
    mapping(uint256 => Loot) private _loots;

    constructor() ERC721("GameLoot", "LOOT") {
        _setupRole(MINTER_ROLE, msg.sender);
    }

    function loot(
        address player,
        LootType lootType,
        uint256 attk,
        uint256 def,
        string memory name_
    ) public onlyRole(MINTER_ROLE) returns (uint256) {
        _lootIds.increment();
        uint256 currentId = _lootIds.current();
        _mint(player, currentId);
        _loots[currentId] = Loot(lootType, attk, def, name_);
        return currentId;
    }

    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC721, AccessControl) returns (bool) {
        return super.supportsInterface(interfaceId);
    }

    function getLootById(uint256 id) public view returns (Loot memory) {
        return _loots[id];
    }
}
```

### A better NFT contract with ERC721Enumerable and ERC721URIStorage

L'exemple précédent fonctionne correctement, mais il nous manque un fonctionnalité assez essentielle: récupérer tous les NFT que possède une adresse. Afin de pouvoir avoir accès à cette fonctionnalité il faut donc hériter d'un ERC721 spécialisé qui possèdent de nouvelles fonctionnalités: `ERC721Enumerable`.  
`ERC721Enumerable` est un ERC721 avec des fonctionnalités supplémentaires.  
Nous pouvons en profiter pour également hériter de `ERC721URIStorage` qui nous ajoute la fonctionnalité de lier une url à un id de NFT qui contiendra des meta-données, souvent un fichier JSON. Ainsi des wallets ou des applications peuvent avoir accès aux informations et meta-données d'un NFT grâce à une url.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/utils/Strings.sol";


contract GameLoot is ERC721Enumerable, ERC721URIStorage, AccessControl {
    using Counters for Counters.Counter;
    using Strings for uint256; // pour convertir un uint256 en string facilement

    enum LootType {
        Weapon,
        Armor
    }

    struct Loot {
        LootType lootType;
        uint256 itemId; // id de notre item dans le jeu. id connu des développeurs.
        uint256 attk;
        uint256 def;
        string name;
    }

    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    Counters.Counter private _lootIds;
    mapping(uint256 => Loot) private _loots;

    constructor() ERC721("GameLoot", "LOOT") {
        _setupRole(MINTER_ROLE, msg.sender);
    }

    function loot(
        address player,
        LootType lootType,
        uint256 attk,
        uint256 def,
        uint256 itemId,
        string memory name_
    ) public onlyRole(MINTER_ROLE) returns (uint256) {
        _lootIds.increment();
        uint256 currentId = _lootIds.current();
        _mint(player, currentId);
        _setTokenURI(currentId, itemId.toString()); // .toString() sur un uint256 grâce à la lib Strings
        _loots[currentId] = Loot(lootType, attk, def, itemId, name_);
        return currentId;
    }


    function getLootById(uint256 id) public view returns (Loot memory) {
        return _loots[id];
    }

    function tokenURI(uint256 tokenId) public view virtual override(ERC721URIStorage, ERC721) returns (string memory) {
        return super.tokenURI(tokenId);
    }

    // Modifions _baseURI afin de retourner l'url de base
    // Cette fonction est utilisée par tokenURI pour retourner une url complète.
    // En fonction de l'item id (et pas du NFT id), nous aurons une url pour chacun de nos loots
    function _baseURI() internal view virtual override(ERC721) returns (string memory) {
        return "https://www.magnetgame.com/nft/";
    }


    // Il existe 2 définitions de supportsInterface, il faut aider le compilateur à gérer ce conflit.
    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC721Enumerable, ERC721, AccessControl) returns (bool) {
        return super.supportsInterface(interfaceId);
    }

    // Il existe 2 définitions de _beforeTokenTransfer, il faut aider le compilateur à gérer ce conflit.
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 tokenId
    )  internal virtual override(ERC721Enumerable, ERC721) {
        super._beforeTokenTransfer(from, to, tokenId);
    }

    // Il existe 2 définitions de _burn il faut aider le compilateur à gérer ce conflit.
    function _burn(uint256 tokenId) internal virtual override(ERC721URIStorage, ERC721) {
        super._burn(tokenId);
    }
}
```
