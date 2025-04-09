# Gestion d'UTXOs pour Wallets d'Entreprise

*Adaptation de l'[article](https://blog.bitgo.com/utxo-management-for-enterprise-wallets-5357dad08dd1) de Mark "Murch" Erhardt sur le forum BitBlog (2020) avec mise en contexte actuelle*

## La crise de congestion comme catalyseur

La congestion du réseau Bitcoin de fin 2017 a représenté un moment charnière pour les opérateurs de portefeuilles d'entreprise, révélant des fragilités structurelles jusqu'alors masquées. Avec des frais dépassant fréquemment 100 satoshis/octet et culminant à plus de 1 000 satoshis/octet, de nombreux services se sont retrouvés confrontés à des portefeuilles techniquement paralysés.

L'ensemble des UTXO a connu une croissance exponentielle, augmentant de 28% en seulement trois mois (passant de 50M à 64M), rendant de nombreuses petites UTXOs économiquement inutilisables. Cette crise a brutalement mis en lumière l'importance cruciale d'une gestion proactive et stratégique des unités de transaction non dépensées pour assurer la continuité opérationnelle.
<img width="1301" alt="https://jochen-hoenicke.de/queue" src="https://github.com/user-attachments/assets/925123bf-5170-43be-8273-ee17b664c909" />
*Taille de la mempool entre décembre 2016 et juillet 2018 en vMB*

### Évolution récente des défis de congestion

Loin d'être un phénomène ponctuel, les problématiques de congestion ont continué d'évoluer. Lors du Halving d'avril 2024, le réseau a connu une explosion des frais sans précédent, largement attribuable au lancement des Runes. Cette dynamique fait suite à l'introduction du protocole Ordinals en 2023, remettant fondamentalement en question la perspective historique de frais durablement bas sur le réseau Bitcoin.

L'étude "[Bitcoin Ordinals: Determinants and impact on total transaction fees](https://www.sciencedirect.com/science/article/abs/pii/S0275531924001314)" apporte un éclairage subtil sur l'impact des Ordinals. Si ces protocoles ont indubitablement créé une demande supplémentaire pour l'espace de bloc, les acteurs impliqués démontrent des capacités d'optimisation significatives. En effet, les utilisateurs d'Ordinals sélectionnent stratégiquement leurs moments de transaction et optimisent leurs processus, limitant ainsi l'impact financier relatif à la taille de leurs transactions.

Ces développements soulignent l'importance stratégique des mécanismes de gestion des UTXOs, désormais critiques non seulement pour les transactions on-chain traditionnelles, mais aussi pour la gestion de liquidités des canaux Lightning Network. Des discussions récentes sur Reddit ([Comment le réseau Lightning peut-il survivre dans un environnement de frais élevés ?](https://www.reddit.com/r/lightningnetwork/comments/18tlhc0/how_does_the_lightning_network_survive_in_a_high/)) ont mis en évidence la fragilité potentielle des nœuds Lightning dans un environnement de frais élevés persistants, notamment lors des opérations sensibles d'ouverture et de fermeture de canaux.

Dans ce contexte mouvant, les entreprises du secteur sont contraintes de s'adapter à un marché des frais désormais plus volatile et structurellement plus élevé. Les stratégies présentées dans cet article deviennent plus que jamais un référentiel opérationnel essentiel.

<img width="1307" alt="https://jochen-hoenicke.de/queue" src="https://github.com/user-attachments/assets/dec7b005-5a02-4b2a-bf6d-84fd07f265f4" />
*Taille de la mempool entre décembre 2016 et avril 2025 en vMB*

## Configuration : Portefeuille chaud dédié à l'envoi

<img width="690" alt="send-only hot wallet" src="https://github.com/user-attachments/assets/6d069582-ae2e-4b8c-afcd-f57ac5d90626" />

*Schéma fait par Mark Erhardt pour illustrer Send-only hot wallet setup*

*Portefeuille actif en **envoi uniquement** : les retraits vers des destinataires externes sont effectués depuis le portefeuille actif (hot wallet). Les dépôts provenant de parties externes sont reçus sur un portefeuille tiède (warm wallet), lequel est restreint à n’envoyer des fonds que vers le portefeuille actif (ou éventuellement vers un stockage à froid, non représenté ici).*

La stratégie de gestion de portefeuille Bitcoin repose sur un écosystème de deux à trois wallets complémentaires, chacun avec une fonction spécifique. Ici on se concentre sur un portefeuille chaud qui envoie uniquement. 

### Rôles des Portefeuilles

- **Portefeuille Chaud** : Dédié exclusivement à l'envoi de transactions
- **Portefeuille Tiède** : Réceptionne l'ensemble des fonds entrants
- **Portefeuille Froid** : Réserve de stockage optionnelle pour la majorité des fonds de l'organisation

### Dynamique de Circulation des Fonds

Le mécanisme de réapprovisionnement suit une logique en cascade :

1. Lorsque le portefeuille chaud s'épuise, il est rechargé par le portefeuille tiède
2. Les fonds excédentaires du portefeuille tiède sont transférés vers le stockage froid
3. En cas d'insuffisance simultanée des portefeuilles tiède et chaud, le portefeuille froid peut remplir le portefeuille chaud

Cette architecture permet une ségrégation claire des fonctions et une gestion dynamique des liquidités.

**Utilisateur type**: Courtier, Service de Prêt

## Avantages Stratégiques

### Sécurisation Renforcée des Dépôts

Le modèle multi-portefeuille offre une architecture de sécurité robuste. Les dépôts sont initialement réceptionnés dans le portefeuille tiède, qui peut limiter strictement les envois uniquement vers le portefeuille chaud et froid. Un mécanisme d'approbation explicite peut être mis en place pour toute transaction sortante, garantissant une isolation des flux et un contrôle granulaire des mouvements de fonds.

### Optimisation de la Gestion des UTXO

Le portefeuille chaud est uniquement utilisé pour l'envoi de transactions. La gestion des UTXO se concentre sur le maintien de fonds suffisants et d'un nombre adéquat d'UTXO. Pendant ce temps, les fonds du portefeuille tiède peuvent être consolidés avec une faible préférence temporelle.

## Limitations Potentielles

### Homogénéité de la Pool d'UTXO

Le portefeuille d'envoi présente des contraintes structurelles. Tous les fonds sont soit des sorties de change d'envois précédents, soit des fonds envoyés via des recharges. Le portefeuille n'a pas une grande variance de valeurs d'UTXO. Cette configuration rend l'évitement du change restant plus difficile, avec presque chaque paiement entraînant la création d'une sortie de change. Cet inconvénient peut être largement atténué en regroupant les paiements et en réduisant ainsi le nombre de sorties de change par paiement.

### Overhead Transactionnel

Comme tous les fonds sont d'abord reçus dans les portefeuilles tièdes, ils doivent être redirigés vers le portefeuille chaud avant de devenir disponibles pour la dépense. Même lorsque les fonds sont consolidés pour le transfert, des sorties de transaction supplémentaires sont créées par rapport à un modèle dans lequel les fonds sont reçus directement dans le portefeuille chaud.

### Défis Opérationnels

Le portefeuille chaud, conçu uniquement pour l'envoi, devient rapidement vulnérable aux fluctuations du réseau Bitcoin. Un nombre limité d'UTXO peut conduire à des chaînes de transactions non confirmées, compromettant potentiellement la capacité à effectuer de nouveaux paiements.

Les blocs lents ou les pics soudains de transactions peuvent épuiser instantanément les fonds confirmés, créant des goulots d'étranglement opérationnels critiques.

### Stratégies de Résilience

Pour contrer ces défis, plusieurs mécanismes sont développés. La première ligne de défense réside dans le maintien de réserves suffisantes, calculées pour absorber trois heures de trafic potentiel. Cette approche prend en compte la variabilité intrinsèque du réseau Bitcoin, notamment les périodes occasionnelles de congestion.

Le regroupement des paiements (batching) émerge comme une solution technique puissante. En consolidant plusieurs transactions dans un unique envoi, les organisations peuvent réduire significativement les coûts transactionnels - jusqu'à 80% dans certains cas. L'efficacité maximale est atteinte en visant 5 à 10 paiements par transaction, avec des intervalles de regroupement adaptés entre une et quinze minutes.

### Optimisation Technique des UTXO

La gestion dynamique des unités de transaction (UTXO) devient un levier stratégique. La division stratégique des grandes UTXO, combinée à des consolidations périodiques du portefeuille tiède, permet de maintenir une flexibilité opérationnelle.

L'adoption d'adresses segwit natives offre un bénéfice supplémentaire, réduisant les frais de transaction et améliorant l'efficacité globale du système.

### Perspective Systémique

Ce modèle représente plus qu'une simple approche technique. Il incarne une philosophie de gestion des risques adaptative, capable de naviguer dans l'écosystème volatile des cryptomonnaies.

Les organisations qui maîtrisent ces subtilités transforment une potentielle fragilité infrastructurelle en un avantage opérationnel robuste.


## Configuration : Portefeuille Chaud d'Envoi et de Réception
<img width="681" alt="Hot send and receive" src="https://github.com/user-attachments/assets/d9c5c327-99da-41ac-b1d1-b8eba427e755" />

*Schéma fait par Mark Erhardt pour illustrer Send and receive hot wallet setup*

*Portefeuille actif en envoi et réception : le portefeuille actif (hot wallet) reçoit les dépôts et effectue également les retraits. Il est réapprovisionné par un portefeuille tiède (warm wallet) lorsqu’il est presque vide, et inversement, les fonds excédentaires du portefeuille actif sont transférés vers le portefeuille tiède.*

Cette configuration fait reposer l'architecture sur le hot wallet qui reçoit ET envoie les fonds.

### Rôles des Portefeuilles

Le portefeuille chaud constitue l'interface principale de réception et d'envoi des transactions. Il centralise directement tous les dépôts entrants et gère l'intégralité des opérations de sortie. Le portefeuille tiède intervient comme un mécanisme de régulation, recevant les fonds excédentaires du portefeuille chaud et préparant leur éventuel transfert vers le stockage à froid. Le portefeuille froid représente la réserve de sécurité ultime, stockant la majorité des actifs de l'organisation.

### Dynamique de Circulation des Fonds

Le mécanisme de réapprovisionnement suit une logique en cascade. Lorsque le portefeuille chaud atteint un niveau de liquidité insuffisant, il est immédiatement rechargé par le portefeuille tiède. Les fonds excédentaires du portefeuille tiède sont systématiquement transférés vers le stockage froid, maintenant un équilibre optimal. En cas de tarissement simultané des portefeuilles chaud et tiède, le portefeuille froid peut directement réapprovisionner le portefeuille chaud, assurant une continuité opérationnelle parfaite.

Cette architecture permet une ségrégation claire des fonctions et une gestion dynamique des liquidités.

**Utilisateur type**: Plateforme de Trading, Service de Garde de Cryptoactifs

## Avantages

L'auto-réapprovisionnement représente une caractéristique structurelle fondamentale de cette configuration de portefeuille. Les dépôts affluent directement vers le portefeuille chaud, créant un mécanisme naturel de compensation qui minimise significativement les interventions manuelles de réapprovisionnement. Cette dynamique réduit non seulement la complexité opérationnelle mais diminue également le nombre de transactions internes de gestion des fonds, optimisant par la même occasion les coûts de transaction.

La constitution d'une pool d'UTXO hétérogène émerge comme un avantage techniquement sophistiqué. En recevant directement les dépôts, le portefeuille chaud accumule naturellement des unités de transaction (UTXO) de valeurs diversifiées. Cette hétérogénéité augmente considérablement l'espace combinatoire pour la sélection des ensembles d'entrées, améliorant mécaniquement la probabilité de générer des transactions sans sortie de change.

## Inconvénients

La gestion des UTXO dans un portefeuille d'envoi et de réception présente une complexité opérationnelle accrue. Le flux constant de transactions entrantes, généralement supérieur aux transactions sortantes, impose une vigilance technique permanente. Les équipes doivent développer des stratégies algorithmiques sophistiquées pour maintenir une efficacité transactionnelle optimale, en anticipant et en contrôlant la fragmentation potentielle du portefeuille.

Un défi technique majeur réside dans la constitution d'ensembles d'entrées. Les montants moyens reçus étant structurellement plus petits que les montants nécessaires aux retraits groupés, ces portefeuilles génèrent mécaniquement des ensembles d'entrées plus volumineux, particulièrement durant les périodes de frais élevés. Cette caractéristique implique des coûts de transaction potentiellement plus importants et nécessite une stratégie de gestion UTXO particulièrement raffinée.


## Défis Opérationnels

Les portefeuilles chauds d'envoi et de réception font face à des défis significatifs liés au volume transactionnel. Dans les environnements de vente au détail à haut trafic, ces portefeuilles peuvent enregistrer des ratios de transactions entrantes vers sortantes atteignant 20 contre 1. Chaque transaction entrante générant au minimum une UTXO, ce modèle peut rapidement conduire à une prolifération exponentielle, avec des portefeuilles gonfant jusqu'à contenir des centaines de milliers d'UTXO.

La problématique des frais de transaction constitue un enjeu économique crucial. La multiplication des UTXO contraint ces portefeuilles à utiliser plusieurs entrées pour chaque paiement. Dans un contexte où la rapidité d'exécution est primordiale, les opérateurs visent une inclusion rapide dans les blocs, ce qui implique des taux de frais potentiellement élevés, grevant significativement la rentabilité des transactions.

## Recommandations Techniques

### Consolidation Stratégique des UTXO

La consolidation représente une réponse méthodologique au problème de prolifération des UTXO. Mark, recommande dans son billet de blog, la mise en place de processus automatisés de consolidation, idéalement sous forme de tâches planifiées. Un mécanisme efficace pourrait consister à générer des transactions de consolidation lorsque plus de 200 UTXO sont inférieures à 1 mBTC.

La stratégie de consolidation doit néanmoins observer des garde-fous stricts. Il est crucial de limiter le nombre de transactions de consolidation en cours pour maintenir la flexibilité opérationnelle du portefeuille. Les paramètres optimaux varient selon les modèles de dépense, pouvant inclure des seuils différenciés par tranche de valeur et des considérations d'âge des UTXO.

### Gestion Dynamique de la Pool d'UTXO

Le maintien d'une variance adaptée dans les montants d'UTXO constitue une stratégie clé. Les meilleures pratiques suggèrent de maintenir une pool entre 200 et 5 000 UTXO, garantissant suffisamment de diversité pour optimiser la sélection tout en évitant la fragmentation excessive.

### Stratégie de Regroupement

Le regroupement des paiements représente une technique d'optimisation majeure. En combinant plusieurs paiements au sein d'une transaction unique, les organisations peuvent réduire significativement les coûts transactionnels, potentiellement jusqu'à 80% des frais globaux. Cette approche nécessite un équilibrage constant avec les processus de consolidation pour prévenir l'inflation incontrôlée de la pool d'UTXO.

Les recommandations techniques présentées visent à offrir une approche holistique de gestion des portefeuilles chauds, conciliant efficacité opérationnelle, optimisation des coûts et flexibilité transactionnelle.

## Exemples 

L'article original de Mark Erardht étant fait pour BitGo les exemples donnés ici se basent sur BitGo. Le but n'est pas d'en faire la promotion mais de comprendre les services et la conception de l'architecture détaillée. On reprend ici une étude de cas de SatSource avec portefeuille chaud dédié à l'envoie et TradeCorp avec portefeuille chaud envoie et récéption. 

### Étude de Cas : SatSource configuration 1

**Transformation Technique**

SatSource, avec un portefeuille d'envoi uniquement, a initialement envoyé des transactions séparées pour chaque destinataire. Avec l'augmentation du volume de paiements, l'entreprise a constaté des limitations dans la disponibilité des fonds confirmés, particulièrement durant les périodes de blocs lents ou de demande de retrait élevée.

**Stratégie de Regroupement**

L'implémentation du batching a révélé des gains significatifs. Avant l'optimisation, chaque transaction générait des frais généraux de 215 octets virtuels (vB) par paiement, incluant une entrée de transaction, une sortie de change et l'en-tête.

En regroupant trois paiements en une transaction unique, ce coût est descendu à environ 93 vB par paiement, représentant une réduction de 57% des frais. Avec la croissance de l'entreprise, la taille du regroupement a progressé jusqu'à neuf paiements par transaction, faisant chuter le poids à 52 vB par paiement - soit une économie de 76% par rapport au modèle initial.

**Optimisation des Adresses**
Sur recommandation de BitGo, SatSource a migré vers des adresses de change segwit natives. Malgré une augmentation du poids de sortie de 32 à 43 octets, les entrées de transaction sont passées de 140 à 105 octets, générant une réduction supplémentaire de 13% des frais.

**Résultats Quantifiables**

- Réduction totale des frais de transaction : près de 80%
- 3,5% des transactions évitent la création de sorties de change
- Pour 1 000 transactions quotidiennes : économie d'environ 1% sur les frais

Cette étude démontre l'efficacité du regroupement et de l'adaptation des formats d'adresses pour les portefeuilles d'envoi Bitcoin.


### Étude de Cas : TradeCorp configuration 2

**Configuration Initiale**

TradeCorp, une plateforme de trading, gère un portefeuille chaud traitant environ 500 dépôts et 400 transactions sortantes quotidiennes. Au démarrage, le portefeuille contenait 63 000 UTXO, avec un accroissement de 500 UTXO hebdomadaires.

**Stratégie de Sélection des UTXO**

L'activation de la sélection des UTXO par BitGo a généré des résultats significatifs :
- 93% de transactions sans sortie de change durant les quatre premières semaines
- Économie d'environ 0,45 bloc Bitcoin de données blockchain par semaine
- Réduction de 20% de la pool d'UTXO

**Consolidation et Défragmentation**

Après quelques mois, le portefeuille se stabilise sous 500 UTXO. La valeur moyenne des UTXO augmente considérablement, réduisant significativement la taille des ensembles d'entrées. Cette défragmentation permet une réduction estimée des frais de transaction de plus de 50%.

**Performances sur Trois Mois**

- 58,5% de transactions sans sortie de change
- 35 000 transactions traitées
- Économies de frais de transaction d'environ 20%

**Optimisation des Taux de Frais**

La sélection d'UTXO sensible aux frais permet :
- Ensembles d'entrées à faibles taux de frais 1,4 fois plus petits
- 32% des transactions à taux de frais bas
- 45% des données d'entrée à taux de frais réduits
- Réduction globale des dépenses en frais d'au moins 13%

**Transition vers Segwit**

L'adoption d'adresses de dépôt segwit natives génère une économie d'espace de bloc supplémentaire de 5%, avec 19,4% des entrées utilisant ce format.

**Résultats Globaux**

Impact total sur les frais de transaction : réduction de plus de 65%, avec une fluidité accrue pendant les périodes de taux élevés. Une extension potentielle de l'intervalle de regroupement pourrait encore optimiser les coûts par paiement.

## Perspectives 

### Détails de la taille du script dépendant du type d'adresse et des inputs/outputs
<img width="702" alt="Tab for size transaction per inputs/output size" src="https://github.com/user-attachments/assets/831e50b2-74bb-4b57-b33c-4878641d8918" />

*Taproot améliorera considérablement les coûts de transaction pour les portefeuilles multisignatures, tout en rendant les entrées et les sorties indiscernables des transactions à signature unique lorsque le chemin de dépense par clé est utilisé.*


### Conclusion 

Ces études de cas montre l'importance d'avoir et de construire des wallets adaptés avec une stratégie préalable afin de pouvoir opérer efficacement et dans le temps sur Bitcoin. 
D'autres hausses de frais pourraient amener des solutions de gestion avancées à se développer et prendre du terrain. 

L'utilisation de taproot à ouvert des possibilités entièrement nouvelles pour les capacités des portefeuilles en réduisant les tailles d'entrée multisig à la même taille que le single-sig dans de nombreux cas. Même en venant d'entrées segwit enveloppées et segwit natives, les entrées pay-to-taproot dépensées via le chemin clé sont 45-58% plus petites pour le multisig 2-sur-3.

Une étude chiffrée sur l'utilisation de taproot dans l'optimisation réelle réalisées par les entreprises du secteur pourrait être intéressante à réaliser. Sur le modèle de transition vers Segwit de TradeCorp. 

Si vous souhaitez développer ces solutions ou que votre entreprise à besoin de conseil en stratégie de gestion d'UTXOs n'hesitez pas à prendre contact avec moi par LinkedIn [Thomas Suau](https://www.linkedin.com/in/thomas-suau-92932889/) ou par mail [thom.suau@orange.fr](mailto:thom.suau@orange.fr?subject=Prise%20de%20contact). 
