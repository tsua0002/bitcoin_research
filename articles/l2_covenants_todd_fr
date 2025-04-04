# Analyse des solutions Layer 2 dépendantes des covenants sur Bitcoin

*Résumé en français de l'article de Peter Todd : [https://petertodd.org/2024/covenant-dependent-layer-2-review](https://petertodd.org/2024/covenant-dependent-layer-2-review)*

## Introduction

Les portefeuilles on-chain atteignent une correspondance approximative de 1:1 entre transactions économiques et transactions blockchain. Le réseau Lightning a révolutionné cette approche en créant une correspondance "plusieurs-à-un", où un nombre potentiellement infini de transactions économiques peut s'effectuer via un seul canal Lightning. Cependant, même Lightning impose une limite d'une UTXO par utilisateur. Cette analyse explore les propositions visant à dépasser cette contrainte en permettant à plusieurs utilisateurs de partager une seule UTXO de manière autonome.

## Qu'est-ce qu'un Layer 2?

Pour cette analyse, un Layer 2 (L2) est défini comme un système libellé en Bitcoin, dont le but est de permettre d'effectuer des transactions en BTC plus fréquemment que le nombre de transactions on-chain avec d'autres parties. De telle sorte que *soit*:

1. Personne ne peut voler rentablement des fonds dans le système, en tenant compte des punitions et coûts internes au système.[^1]
2. *(Préféré)* Les véritables propriétaires des fonds peuvent retirer unilatéralement leurs fonds, moins les frais de transaction, sans la coopération de tiers.

Cette définition précise exclut des systèmes comme Liquid, Cashu, Fedimint ou les Statechains, où un tiers contrôle les fonds ou peut les voler.

## Les covenants: mécanismes et importance

Les covenants sont des mécanismes qui restreignent la façon dont une UTXO peut être dépensée, au-delà des simples signatures. Ils permettent d'implémenter des règles complexes essentielles aux protocoles L2 qui partagent des UTXO entre plusieurs participants. Les covenants récursifs, particulièrement controversés, peuvent appliquer ces restrictions indéfiniment aux UTXO enfants, soulevant des questions sur leur impact à long terme.[^2]

## Capacité actuelle et limites de Lightning

Malgré ses limitations, Lightning peut théoriquement supporter environ 1,1 milliard de nouveaux canaux par an, ce qui permettrait d'intégrer une part significative de la population mondiale.[^3] Les contraintes principales de Lightning sont:

- Le besoin d'avoir au minimum une UTXO par utilisateur final[^4]
- L'immobilisation de fonds dans des canaux
- L'exigence que les destinataires soient en ligne pour recevoir des paiements

Les transactions de splicing peuvent modifier la capacité des canaux, avec une capacité théorique d'environ 470 millions d'opérations par an. Cette capacité, bien qu'impressionnante, reste insuffisante pour une adoption véritablement globale sans partage d'UTXO.

## Architectures fondamentales des Layer 2

Deux modèles principaux émergent parmi les systèmes L2:

1. **Les canaux** (comme Lightning) fonctionnent en échangeant des transactions pré-signées qui *pourraient* être minées mais ne le sont pas dans le scénario idéal. Ces systèmes nécessitent des mécanismes d'incitation pour garantir que la transaction correcte soit effectivement minée.

2. **Les UTXO virtuelles (V-UTXO)** créent des UTXO virtuelles via des covenants ou des accords multipartites, permettant des retraits unilatéraux on-chain. Contrairement aux canaux, les transactions dans les systèmes V-UTXO impliquent de dépenser les V-UTXO elles-mêmes dans une seule transaction pré-signée.[^5]

Ces deux approches utilisent généralement un chemin de script "tous les participants sont d'accord" (comme un multisig N-sur-N), taproot étant spécifiquement conçu pour optimiser cette approche.

## Solutions Layer 2 notables

### Lightning et ses évolutions

Lightning constitue la référence actuelle, avec une architecture basée sur des canaux de paiement et un réseau P2P pour le routage. Plusieurs améliorations sont proposées:

- **Channel Factories**: Permettent à plusieurs parties de négocier une adresse multisig n-sur-n et de créer plusieurs canaux Lightning à partir d'une seule transaction on-chain, économisant ainsi de l'espace blockchain.

- **LN-Symmetry/Eltoo**: Supprime le mécanisme de pénalité des canaux Lightning en permettant la mise à jour des états incorrects, simplifiant considérablement le protocole. Cette approche nécessite l'implémentation de SIGHASH_ANYPREVOUT via un soft fork.

### Ark: un système de V-UTXO prometteur

Ark propose une approche novatrice de scaling via des UTXO virtuelles entièrement transférables. Un coordinateur central (Ark Service Provider, ASP) fournit des V-UTXO avec une durée de vie limitée, généralement organisée en "rounds" de plusieurs semaines. Ces V-UTXO sont créées via des txout de pool, utilisant des mécanismes comme CTV pour permettre à une seule txout on-chain de s'engager dans un arbre de V-UTXO.

L'économie d'Ark repose sur un coût de liquidité (environ 0,23% pour un round de 4 semaines) que l'ASP doit supporter.[^6] Ce coût est lié au capital immobilisé par l'ASP entre le moment où une V-UTXO est dépensée et l'expiration du round.

Ark présente plusieurs défis:
- Un amorçage difficile: un nouvel ASP avec peu d'utilisateurs doit trouver un équilibre entre fréquence des rounds et coûts par utilisateur
- Une interactivité élevée: les utilisateurs doivent interagir avec l'ASP avant l'expiration de leurs V-UTXO[^7]
- Des risques de centralisation: il devient économiquement avantageux pour les utilisateurs de faire confiance à l'ASP plutôt que d'assurer leur propre sécurité

### Autres approches significatives

- **Validity Rollups**: Utilisant des preuves à connaissance nulle pour garantir les règles de la chaîne, cette approche nécessiterait probablement des opcodes génériques comme OP_CAT pour valider ces preuves.[^8]

- **CoinPool**: Permet à plusieurs utilisateurs de partager une seule UTXO avec possibilité de transfert de fonds et retrait unilatéral, mais nécessite plusieurs nouvelles fonctionnalités de soft fork.

- **BitVM**: Offre une exécution conditionnelle entre deux parties, mais ne résout pas directement la limitation d'une UTXO par utilisateur.

## Défis liés à la mempool

Les systèmes L2 sont confrontés à trois problématiques majeures liées à la mempool Bitcoin:

1. **Transaction pinning**: Des exploitations économiques rendant difficile la confirmation des transactions désirables. Le pinning via BIP-125 Rule #3 reste le principal problème non résolu, potentiellement solvable par le replace-by-fee-rate.

2. **Paiement des frais**: RBF reste l'approche la plus efficace pour les paiements de frais, mais nécessite la pré-signature de variantes de transactions. Les sorties d'ancrage (anchor outputs) avec CPFP sont moins efficientes mais plus simples à implémenter.

3. **Replacement cycling**: Des attaques exploitant le remplacement de transactions pour empêcher l'exécution de HTLCs jusqu'à ce qu'une autre expiration se produise, permettant potentiellement le vol de fonds.[^9]

## Soft forks proposés et leurs applications

### OP_Expire
Rendrait une output non dépensable après un certain temps, résolvant élégamment les problèmes de replacement cycling. Toutefois, cette approche soulève des préoccupations en cas de réorganisation de la blockchain.

### SIGHASH_ANYPREVOUT
Ne s'engage pas sur l'UTXO spécifique dépensée, ce qui est essentiel pour LN-Symmetry. Cette fonctionnalité évite de devoir signer chaque état antérieur du canal, mais présente des risques de réutilisation de signatures si mal utilisée.

### OP_CheckTemplateVerify (CTV)
Restreint précisément la façon dont une UTXO peut être dépensée sans permettre de covenants récursifs. CTV offre un excellent équilibre entre puissance et simplicité pour la plupart des utilisations L2, particulièrement les arbres de txout.

### OP_CAT et Script Revival
Permet d'implémenter diverses formes de covenants, y compris récursifs, mais soulève des inquiétudes quant aux conséquences imprévues comme le Miner Extractable Value (MEVil), l'implémentation de Drivechains, ou la prolifération de jetons on-chain consommant de l'espace blockchain.[^10]

## Architectures de pools de fonds et scaling

Toutes les solutions L2 partageant une UTXO entre plusieurs utilisateurs peuvent être conceptualisées comme des pools de fonds avec différentes structures:

1. **Transactions pré-signées individuelles**: Le cas le plus simple, comme dans Lightning, où la pool change d'état une seule fois.

2. **Arbres de txout**: Une structure où la pool peut être divisée en dépensant l'UTXO racine dans un arbre de transactions prédéfinies. Cette approche offre des options de retrait flexibles mais à un coût logarithmique par rapport au nombre de participants.

3. **Schémas basés sur les soldes**: Traitent la pool comme un solde évolutif, nécessitant des covenants avancés et des structures de données merkelisées pour maintenir l'état et garantir la disponibilité des données.

Ces structures partagent toutes un compromis fondamental entre efficacité de scaling et sécurité.

## Risques systémiques et considérations pratiques

Les L2 partageant des UTXO entre plusieurs utilisateurs augmentent potentiellement la demande de blockspace en cas de défaillance. Cette caractéristique crée un risque "d'attaque par sortie massive" où un grand nombre d'utilisateurs tenteraient simultanément de retirer leurs fonds on-chain, ce qui pourrait saturer la capacité de la blockchain.[^11]

Avant d'adopter largement de nouveaux schémas de partage d'UTXO, il serait prudent d'acquérir davantage d'expérience opérationnelle avec Lightning et ses limites actuelles.

## Tableau comparatif des besoins en soft forks

|                  | OP_Expire | SIGHASH_ANYPREVOUT | CTV | OP_CAT | Script Revival |
|------------------|:---------:|:------------------:|:---:|:------:|:--------------:|
| Lightning        |     U     |         U          |  U  |        |                |
| Channel Factories|     U     |                    |  U  |        |                |
| LN-Symmetry      |     U     |         R          |     |        |                |
| Ark              |     U     |                    |  R  |        |                |
| Advanced Ark     |     U     |                    |  U  |        |        R       |
| Validity Rollups |           |                    |     |    R   |        U       |

*U = Utile, R = Requis*

## Conclusion et recommandations

L'analyse comparative montre que CTV est la solution la plus largement applicable, suivie par SIGHASH_ANYPREVOUT. CTV présente un risque limité d'effets secondaires imprévus comparé à OP_CAT ou au Script Revival complet, tout en offrant suffisamment de puissance pour de nombreuses applications L2.[^12]

La feuille de route recommandée par Peter Todd consiste à:
1. Implémenter un "consensus cleanup" pour corriger des bugs historiques
2. Déployer CTV, qui répond aux besoins majoritaires des L2
3. Évaluer prudemment l'implémentation d'opcodes plus puissants comme OP_CAT, après analyse approfondie des risques potentiels

Cette approche progressive permet d'améliorer significativement les capacités L2 de Bitcoin tout en préservant ses propriétés fondamentales de sécurité et de décentralisation. L'adoption de ces améliorations pourrait transformer profondément les possibilités de scaling de Bitcoin, tout en maintenant le modèle de sécurité qui a fait son succès.

[^1]: Il est probable que certaines implémentations Lightning existantes ne limitent pas correctement la valeur totale des HTLCs "dust" en circulation, ce qui pourrait les rendre vulnérables à certaines attaques.

[^2]: Les covenants récursifs sont considérés comme indésirables par certains depuis longtemps car ils peuvent restreindre les pièces indéfiniment, potentiellement sans la permission d'un tiers comme un gouvernement.

[^3]: Alex Bosworth a indépendamment réalisé une analyse en 2022 arrivant aux mêmes chiffres concernant la capacité de Lightning.

[^4]: Lightning nécessite une UTXO par utilisateur car c'est un réseau: avec seulement une UTXO pour deux utilisateurs, on ne pourrait avoir que des paires isolées d'utilisateurs, sans réseau plus large.

[^5]: Pour les besoins du RBF, un système V-UTXO pourrait avoir plusieurs transactions pré-signées, mais leur objectif serait de réaliser une seule transaction économique.

[^6]: Ce calcul n'est pas considéré comme un taux d'intérêt composé car le montant principal ne change pas, similaire au fonctionnement des paiements d'obligations.

[^7]: Si l'ASP refuse de permettre la dépense d'une V-UTXO dans un nouveau round, la seule alternative pour l'utilisateur est un retrait unilatéral on-chain, qui peut être coûteux et risqué si effectué trop près de l'expiration.

[^8]: StarkWare, notamment, fait campagne pour l'adoption d'OP_CAT afin de permettre la vérification de preuves STARK en script Bitcoin.

[^9]: Pour une explication plus accessible des attaques de replacement cycling, Peter Todd recommande le fil de tweets de mononautical, en complément du papier original d'Antoine Riard.

[^10]: La complexité des instruments financiers potentiels qui pourraient être créés avec OP_CAT rend très difficile d'exclure le risque de Miner Extractable Value that is evIL (MEVil), comme l'a nommé Matt Corallo.

[^11]: "Mass Exit Attacks on the Lightning Network", une étude de Cosimo Sguanci et Anastasios Sidiropoulos de février 2024, analyse en détail ce type d'attaques.

[^12]: CTV s'impose car de nombreuses solutions s'intègrent dans le modèle de conception "s'assurer que la transaction de dépense correspond à ce modèle"; même les constructions basées sur OP_CAT peuvent efficacement utiliser CTV.
