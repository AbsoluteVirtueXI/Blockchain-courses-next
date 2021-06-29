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

## Stablecoins

### Une monnaie de référence stable pour des échanges stables

Nous pouvons vendre l'accès ou les services de notre Dapp/smart contract assez facilement en Ether ou en tokens. Malheureusement les cryptomonnaies n'étant pas régulées par une autorité de contrôle et le marché est volatile. Il est donc impossible de prendre une quantité de cryptomonnaie comme de l'Ether/BTC ou même un ERC20 standard comme prix de référence pour notre service.  
En effet vendre l'accès à notre application 0.001 Ether aujourd'hui le 29 juin 2021 correspondrait à environ 1.74 Euro, mais cette même quantité d'Ether peut avoir un prix variant fortement dans les semaines qui suivent, pour le meilleur comme pour le pire.  
Cet exemple s'applique aussi à une quantité de cryptomonnaies que l'on laisserait dormir sur notre wallet, ou à l'achat de bien `offchain` avec des cryptomonnaies.

### La monnaie Fiat une monnaie stable ?

Oui.  
Les monnaies souveraines sont d'excellentes monnaies de références car elles possèdent une stabilité contrôlée par des autorités monétaires, de nos jours les banques centrales, qui se battent chaque jour contre une inflation ou une déflation trop importante.  
Elles sont utilisés sur les marchés pour leur stabilité.
De plus elles sont naturellement la référence des cryptomonnaies sur les marchés, actuellement on compare toujours le prix du BTC, de l'ETH ou d'un token à son prix en USD ou en EURO et pas l'inverse.  
Les fiat sont donc d'excellentes monnaies de références, mais totalement inutilisables dans une application Blockchain, nous ne pouvons pas accepter du Dollar ou de l'Euro dans notre smart contract. Le problème est que l'émission de ces monnaies et les transactions avec ces monnaies se font `offchain`, il est donc impossible avec un pur processus `onchain` de récupérer des fiat ou des informations sur les cours des monnaies souveraines.

### Tokens indexés sur des Fiat

Ce sont les stablecoins.  
Les stablescoins sont des tokens qui auront toujours la même valeur comparée à une monnaie de référence.  
Actuellement la monnaie de référence généralement utilisée est le Dollar, ainsi nous aurons toujours `1 stable coin == 1 USD`.
Les stables coins sont des ERC20, nous pouvons donc utiliser un équivalent du Dollar dans nos applications avec les fonctionnalités d'un ERC20, c'est très pratique pour nos applications décentralisées.
Il existe actuellement 3 technologies pour créer des stables coins.

#### Fiat-collateralized stablecoins

Les `fiat-collateralized stablecoins` sont des `Collateral-backed Tokens` dont le collatéral est envoyé `offchain` pour créer `onchain` un stable coin.  
Des fiats sont envoyés à une entreprise, sur leur compte en banque, qui gère ensuite l'émission des stablecoins qui seront ensuite envoyés à l'utilisateur.  
Ce processus est totalement centralisé, l'entreprise à un total contrôle et il n'y a aucune transparence sur la quantité de fiat sur leur compte en banque. Il est impossible de savoir si la quantité de fiat sur leur compte en banque correspond à la quantité de stablecoins émis.  
[Tether Limited](https://tether.to/) est une organisation qui fonctionne de cette manière, elle est la créatrice du stablecoin `Tether` aussi connu sous le symbole `USDT`.
Leur whitepaper: https://tether.to/wp-content/uploads/2016/06/TetherWhitePaper.pdf
Etant totalement centralisé, avec aucune transparence sur l'état de leur compte en banque qui reçoit le collatéral des utilisateurs c'est un modèle qui tend à ne plus exister.  
Néanmoins l'`USDT` est actuellement le stablecoin le plus utilisé et qui possède la market cap la plus importante de tous les stablecoins.
Tether Limited à fait l'objet de poursuites par la justice et est au centre de multiples allégations:

- Tether USDT disparus
- Manipulation des prix de marché par émission massive d'USDT
- Une réserve de Fiat qui n'est pas synchronisé à la total supply de tous les USDT émis.

Le smart contract de `TETHER USD` est déployé sur le mainnet à l'adresse [0xdac17f958d2ee523a2206206994597c13d831ec7](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7)

**Exercice**:
Trouver l'adresse de l'owner du smart contract déployé sur le mainnet.  
Quelle est la particularité de cette adresse?  
Quels sont les pouvoirs de cette adresse sur ce smart contract?

#### Crypto-collateralized stablecoins

Ces stables coins sont aussi des `Collateral-backed Tokens` mais cette fois le collatéral est de la cryptomonnaie.  
L'émission de ces stablecoins peut donc totalement être gérée `onchain` par des smart contracts en fonction du collatéral reçu en cryptomonnaie et des règles définies.  
Il faut néanmoins récupérer une information `offchain`: Le prix en fiat de la quantité de collatéral envoyé, pour cela un `oracle` est utilisé.  
Un `oracle` est un smart contract qui est modifié/updaté par un programme `offchain` qui renseignera l'`oracle` avec les données récupérées.  
Dans le cas d'un stablecoin, un oracle récupérera des informations sur des exchanges ou sites de référecement via leur api pour obtenir le prix en fiat du collatéral reçu et émettra la quantité de stablecoin correspondante.  
L'architecture est décentralisé et donc plus complexe.  
le `Crypto-collateralized stablecoin` le plus connu et le plus utilisé est le `DAI` crée par [MakerDAO](https://makerdao.com/en/).  
Le whitepaper: https://makerdao.com/en/whitepaper
Le protocole Maker est un `MakerDAO's Multi-Collateral Dai (MCD) System`, qui fonctionne de manière totalement décentralisée.  
Ce protocole est reconnu comme l'un des plus avancé de l'écosystème nécessitant de grandes compétences techniques et financières pour le comprendre totalement.  
Dans la [documentation](https://docs.makerdao.com/) du protocole nous pouvons retrouver une overview de l'architecture logicielle.  
Tous les codes sont publics et accessibles sur leur [repository](https://github.com/makerdao).  
Les composants essentiels sont:

- Un système de gouvernance décentralisé permettant aux owners d'un token de gouvernance, le `MKR`, d'influer ou de paramétrer le protocole.
- Un réseau d'oracle
- un système d'`incentives` pour motiver l'utilisation du protocole
- Une ingénierie financière dans la gestion des dettes et de surplus afin de garantir la stabilité du DAI.

#### Algorithmic Stablecoins

Les `Algorithmic Stablecoins` régulent leur prix en influant avec des algorithmes sur la total supply du stablecoin.  
Ainsi l'enjeu est de faire baisser la total supply lorsque le stable coin est à moins de 1 Dollar pour créer de la rareté, ou l'inverse, d'augmenter la total supply si le stable coin est à plus de 1 Dollar.
Dans le cas de [Ampleforth](https://www.ampleforth.org/) le prix du token est récupéré sur les marchés par des oracles et les balances de tous les utilisateurs sont augmentées ou diminuées en fonction du prix du stablecoin.  
Le [Frax protocol](`https://frax.finance/`) est un autre exemple mais avec une autre méthode de régulation de son prix.
