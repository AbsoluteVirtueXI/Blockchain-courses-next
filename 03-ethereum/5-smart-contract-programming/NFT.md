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
Et on peut retrouver l'impélementation de ce standard par Openzepplin sur leur repository Github: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol  
Une documentation de leur smart contract est disponible sur: https://docs.openzeppelin.com/contracts/4.x/erc721  
L'api de l'ERC-721 d'OpenZepplin est documentée ici: https://docs.openzeppelin.com/contracts/4.x/api/token/erc721

### ERC721 from OpenZepplin

See remix livecoding
