# Assets

## Native blockchain assets

- Ether
- BTC
- DOT

## ERC20

## ERC721

## Wrapped Ether

L'Ether est utilisé dans beaucoup de protocole DeFi, mais dans sa forme `wrapped`.  
En effet l'Ether ne possède pas toutes les fonctionnalités d'un ERC20, et notamment la plus intéressante pour un protocole qui devra réagir en notre nom, la possibilité d'effectuer un `transferFrom`.  
Un smart contract qui devra gérer les Ethers d'utilisateurs devra aussi utiliser ses propres mappings (ou d'autres structures de données) afin de garder un tracking des balances de ses utilisateurs, ainsi que d'autres fonctionnalités utiles comme les autorisations de transferts et les transferts.  
Afin de permettre à un protocole DeFi de travailler avec nos Ethers avec les même fonctionnalités qu'un ERC20 on peut utiliser un `wETH` token.
le [wETH](https://weth.io/) est un ERC20, il possède donc toutes les fonctionnalités d'un ERC20.  
le `wETH` a cette particularité: `1 wETH == 1 ETH`.  
Le smart contract du `wETH` sur le mainnet est déployé à l'adresse: [0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2](https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2).  
Pour obtenir du `wETH` nous devons envoyer de l'Ether via `function deposit() public payable`.  
Il est également possible d'envoyer directement de l'Ether à l'adresse du smart contract, car une fonction de fallback (que l'on nomme désormais la fonction `receive`) gère la réception d'Ether.  
Pour convertir nos `wETH` en `ETH` nous devons appeler `function withdraw(uint wad) public`. Cette fonction brulera la quantité de `wETH` passée en paramètre et nous enverra la quantité d'Ether correspondante.  
Nous avons la certitude que pour tout `wETH` créé il existe exactement la même quantité d'Ethers dans le smart contract, et lorsqu'un `wETH` est brûlé alors alors la quantité d'Ethers correspondante est envoyé aux burners.

**Exercice 1**:  
Créer un smart contract `WETH` qui possède le nom `Wrapped Ether` et le symbole `WETH ` que vous déploierez sur un testnet.  
Ce smart contract devra fournir toutes les fonctionnalités attendues pour fonctionner comme le `WETH` officiel du mainnet.

**Exercice 2**:  
Sachant que l'adresse sur laquelle sera déployée un smart contract est déterminée en fonction de l'adresse qui effectue la transaction et son `nonce` (le `nonce` est le nombre de transactions qu'a effectué un account sur le réseau), déployer le smart contract `WETH` à la même adresse sur les réseaux de tests: ropsten, kovan, rinkeby, goerli.

## Collateral-backed Tokens

Dans le même principe que le `wETH` qui est "backed" par de l'ether, nous pouvons généraliser ce concept à n'importe quel type de liquidité, et donc pas que de l'ether.  
Un smart contract peut ainsi émettre des tokens à la réception d'autres tokens.  
Les tokens envoyés sont appelés le `collateral`, les tokens émis/créés sont appelés les `Collateral-backed tokens`.  
Pour toute création d'un `Collateral-backed token` il faut que d'autres assets/tokens soient déposés dans le smart contract.  
Pour récupérer son `collateral`, un utilisateur devra renvoyer les `Collateral-backed tokens` au smart contract qui seront brûlés et un transfert de son `collateral` sera effectué vers son adresse.

**Exercice 3**:  
Créer un smart contract `CollateralToken` qui sera un ERC20 et qui aura pour symbole `CT`.  
Créer un smart contract `CollateralBackedToken` qui sera un ERC20 et qui aura pour symbole `CBT`.  
Via une fonction `deposit`, un utilisateur pourra déposer du `CT` dans le smart contract `CollateralBackedToken` et recevra ainsi 1/2 fois la quantité de `CT` déposée, en `CBT`.
Via une fonction `withdraw`, un utilisateur pourra récupérer ses `CT` en brulant ses `CBT`, il recevra donc la deux fois de la quantité de `CBT` brûlée, en `CT`.  
`1 CBT == 2 CT`.
