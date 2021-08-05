# CEX and DEX

## Principle

### Exchange

Un exchange est une place de marché ou un acheteur échange un asset contre un autre asset d'un vendeur, publiquement.  
Dans sa manière la plus simple:

1. un acheteur souhaite acheter une ressource pour un certain prix, un `buying order` ou le `bid`.
2. un vendeur souhaite vendre une ressource pour un certain prix, un `selling order` ou le `ask`/`offer`.
3. si le prix de l'acheteur et du vendeur correspondent alors une transaction, un échange, est effectué car il y a un `matching orders`.

Tous les ordres d'achats et de ventes sont stockés dans un `order book`, consultable publiquement, renseignant ainsi sur l'état du marché, c'est à dire sur l'offre et la demande.

Dans un exchange les prix ne sont régulés que par l'offre et la demande, respectant ainsi une loi immuable de la finance et de l'économie: Plus une ressource est demandés plus elle devient rare et plus son prix augmente, l'offre est inférieure à la demande.
Ce qui implique que plus une ressource inonde le marché, moins elle est rare et plus son prix baisse, l'offre est supérieure à la demande.

Peuvent également intervenir des leviers psychologiques rendant certains comportements et prix irrationnels, le plus connu dans le milieu des startups et des crypto est le [FOMO](https://en.wikipedia.org/wiki/Fear_of_missing_out)

### Market makers

L'histoire des bourses montrent qu'un équilibre naturel entre l'offre et la demande est difficile à obtenir.  
En effet les bonnes ressources au bon prix tendent à s'épuiser, et donc à faire monter le prix de ces ressources, rendant ainsi ces ressources inintéressantes pour des acheteurs car trop chères.  
Ou il y a trop d'offres pour des ressources pour pas assez acheteurs rendant ainsi la vente de ces ressources inintéressantes pour les vendeurs car le prix tend à baisser.

Des organismes ou individus sont donc mandatés par les propriétaires d'une ressource (une action, une crypto, un exchange?) pour fluidifier le marché artificiellement. Ils apportent des `liquidités` en agissant sur les `bid` ou sur les `ask` en fonction de l'état du marché disponible via l'`order book`.  
Ils achètent ou vendent pour combler les trous de l'`order book` afin que les acheteurs ou les vendeurs puissent trouver des `matching orders` selon qu'ils soient vendeurs ou acheteurs.  
Les market makers arrivent en renfort d'entreprises qui souhaitent redresser leurs actions ou pour contribuer au bon déroulement d'une [IPO](https://en.wikipedia.org/wiki/Initial_public_offering).  
Evidement cette activité, tolérée et légale, est mal connue et souffre d'une mauvaise réputation.  
Les market makers sont actuellement au coeur de l'activité de tous les exchanges, centralisés ou décentralisés, mais un type d'exchange décentralisés, les Automated Market Makers (AMM) rendra cette activité désuète, n'utilisant plus d'`order book` et régulant les prix par une formule mathématique en fonctions de proportions des reserves dans un `liquidity pool`.  
L'interaction avec ces `AMM` est simple et totalement `on-chain` ce qui a contribué à leur popularité dans les protocoles DeFi.  
Les AMM trouvent leur source dans un article de Vitalik Buterin de juin 2017: [On Path Independence](https://vitalik.ca/general/2017/06/22/marketmakers.html)

## CEX: Centralized Exchange

Ce sont des places de marchés que l'on dit "centralized".  
Elles fonctionnent exactement comme des bourses traditionnelles, seulement elles sont spécialisés dans les échanges de cryptomonaies et FIAT.  
Elles sont les intermédiaires entre les acheteurs et les vendeurs et s'assurent que les acteurs d'un échange possèdent bien les ressources qu'ils souhaitent échanger.  
L'échange se fait via leur intermédiaire et transite via leurs comptes en banque si des FIAT est l'une des paires de l'échange, ou via des account de blockchains dont ils sont les propriétaires (c'est à dire qu'ils possèdent les clefs privées associées à ces adresses).  
En effet sur ces exchanges on peut vous attribuer des adresses Ethereum ou Bitcoin afin que vous puissiez les échanger, mais eux seuls possèdent les clefs privés de ces accounts: ils font l'échange à votre place.

Qu'on aime ou qu'on n'aime pas, les CEX sont actuellement la seule alternative si dans un échange des FIAT sont impliquées, car il n'existe aujourd'hui pas de moyens de vérifier `on-chain` que vous possédez bien la quantité de FIAT à utiliser dans un échange ou qu'un transfert de FIAT à correctement été effectué.  
Egalement si on souhaite échanger des cryptomonaies qui sont dans des blockchains différentes les CEX sont actuellement nécessaires. Ceci est vrai jusqu'a ce que des protocoles d'interopérabilités `cross-chain` soient opérationnels, par exemple le projet Moonbeam qui permettra d'utiliser des assets Ethereum sur des parachains du réseau Polkadot/Kusama (ou des blockchains externes connectées au réseau Polkadot/Kusama via des [bridges](https://wiki.polkadot.network/docs/learn-bridges)).
Le plus gros avantage des CEX, contrairement au DEX classiques, sont leurs liquidités, car ils travaillent avec des FIAT.  
Les FIAT sont les actifs les plus liquides, on peut se les procurer facilement, ils sont assez stables car régulées et contrôlées et ouvrent la porte à de nombreux acteurs (acheteurs/vendeurs) qui peuvent ainsi échanger du FIAT contre n'importe quelles crypto ou vice versa.

Les CEX les plus connus sont: Binance, Kraken et Coinbase.  
Sur un CEX si vous souhaitez dans vos échanges ou conversions travailler avec des FIAT il vous faudra obligatoirement justifier de votre identité: [KYC policy](https://en.wikipedia.org/wiki/Know_your_customer).

## DEX: Decentralized Exchange

Ces exchanges ne fonctionnent qu'avec des cryptomonnaies/tokens d'une même Blockchain (des DEX `cross-chain` sont en cours de développement.).  
Les échanges se font donc via des smart contracts de manière décentralisée et sans autorités de contrôle intermédiaires pour valider les transactions, ce qui n'est possible qu'avec des cryptomonnaies.

L'`order book` étant la pièce essentielle d'un exchange (jusqu'a récemment), les premières versions d'exchanges décentralisés se sont construites sur le même modèle que les exchanges centralisés.  
Des smart contracts effectuaient l'echange des ressources, mais de part la nature de l'EVM et de Solidity entrainant des limitations, des protocoles complexes mêlant des composants `off-chain` à des smart contracts `on-chain` ont été développés pour se rapprocher des fonctionnalités d'un exchange décentralisé.  
Par exemple [0x Protocol](https://0x.org/) a développé un framework pour permettre de créer ses propres `DEX`, mêlant noeuds `off-chain` pour passer des ordres et des smart contracts qui effectuent les transactions correspondantes à ces ordres.  
De plus leur désavantage majeur est leur manque de liquidités. Les DEX classiques ne travaillant qu'avec des crypto, certaines inintéressantes ou en trop petites ou trop grandes quantités, les traders doivent attendre plus longtemps pour trouver un match avec leurs ordres, s'exposant ainsi plus longtemps dans la durée à une volatilité imprévue du marché.  
L'utilisation d'un `order book` est pertinente dans un exchange ou la liquidité est élevée, il est possible que les DEX avec `order book` `cross-chain` résolvent ce problème, mais pour l'instant la solution qui a été suggéré dans [On Path Independence](https://vitalik.ca/general/2017/06/22/marketmakers.html) a été **massivement** adoptée faisant des AMM la solution idéale pour de véritable DEX.  
Parmi les `off-chain order book DEX` on peut retrouver: dYdX, IDEX, 0x.
