# État de l'art sur les Bitcoin Vaults

Cette étude synthétise les recherches sur les Bitcoin Vaults, s'appuyant principalement sur les références fournies par Bitcoin Optech [^1] et l'article fondamental de Swambo et al. (2020) [^3] concernant les protocoles de garde utilisant cette technologie.

## Principes fondamentaux des Bitcoin Vaults

Un **Bitcoin Vault** constitue une forme spécialisée de covenant (contrat restrictif, discuté précédemment dans [L2 & Covenants](../fr_l2_convenants-peter-todd.md)) qui impose une séquence temporelle et causale de deux transactions distinctes, confirmées dans des blocs différents, pour finaliser le transfert de fonds. La première transaction signale l'intention de dépense, créant ainsi une fenêtre d'opportunité critique pendant laquelle l'utilisateur légitime peut intercepter et annuler toute tentative non autorisée avant l'exécution de la seconde transaction[^1].

Le mécanisme des vaults introduit trois caractéristiques sécuritaires essentielles: un délai obligatoire entre l'initiation et la finalisation d'une dépense, une voie de récupération pour contrecarrer les transactions malveillantes, et la possibilité de rediriger les fonds vers une adresse hautement sécurisée ou, dans les cas extrêmes, d'opter pour la destruction du capital plutôt que sa compromission[^2].

## Mécanismes d'implémentation des Vaults

L'analyse des différentes approches d'implémentation révèle deux paradigmes distincts, chacun présentant ses propres avantages et complexités.

### Implémentations nécessitant des modifications du consensus

La littérature examine plusieurs propositions d'extension du protocole Bitcoin par de nouveaux opcodes. `OP_VAULT` et `OP_UNVAULT` ont été proposés en 2023, conçus spécifiquement pour faciliter les mécanismes de vault[^4]. D'autres approches incluent l'utilisation de `OP_CHECKTEMPLATEVERIFY` (CTV), qui permettrait la vérification de templates de transactions, ou `OP_TAPLEAF_UPDATE_VERIFY`, qui s'appuierait sur l'infrastructure Taproot. Les combinaisons d'opcodes comme `OP_CHECKSIGFROMSTACK` avec `OP_CAT` (proposé après 2023) fourniraient également une base technique pour l'implémentation de vaults, tout comme les mécanismes MATT (Merklize All The Things) qui permettraient des constructions contractuelles plus sophistiquées.

### Implémentations utilisant l'infrastructure existante

La recherche de Swambo et al. présente une méthodologie innovante basée sur des transactions pré-signées avec suppression sécurisée des clés, évitant ainsi la nécessité de modifications du consensus[^3]. Cette approche pragmatique exploite les fonctionnalités existantes du protocole Bitcoin. D'autres méthodes explorent l'utilisation des signatures Schnorr combinées avec des mécanismes comme `OP_CAT`, ou encore des structures multisignatures intégrant des verrouillages temporels.

## Architecture de sécurité multi-niveaux selon Swambo et al.

Le protocole de garde développé par Swambo et al. propose une architecture modulaire et robuste organisée autour de trois types de portefeuilles complémentaires.

Le **portefeuille actif**, destiné aux opérations quotidiennes, reste connecté à l'interface utilisateur principale. Il représente le composant le plus accessible mais aussi potentiellement le plus vulnérable du système. Le **portefeuille vault** constitue le cœur sécuritaire du dispositif, employant des clés éphémères pour authentifier les transactions de covenant. Ces clés, générées pour un usage unique puis détruites, limitent significativement la surface d'attaque. Finalement, le **portefeuille de récupération**, distribué géographiquement avec des mesures de sécurité physique renforcées, sert de mécanisme ultime de sauvegarde[^3].

Cette architecture s'articule autour de deux types de transactions covenant complémentaires. La **transaction vault** offre une bifurcation d'exécution: soit une dépense standard après expiration d'un timelock prédéfini, soit une récupération immédiate en cas de détection d'activité suspecte. La **transaction push-to-recovery-wallet (P2RW)** redirige, quant à elle, les fonds vers le portefeuille de récupération hautement sécurisé.

L'ensemble est surveillé par un **watchower** (tour de garde) qui scrute en permanence la blockchain et le réseau P2P pour détecter toute tentative non autorisée d'extraction des fonds sécurisés. Ce composant peut, selon sa configuration, soit notifier le propriétaire légitime, soit déclencher automatiquement les mécanismes de récupération préétablis.

## Évolution chronologique des propositions de Vaults (2019-2025)

### Premières implémentations (2019-2020)

Les fondations conceptuelles ont été posées avec Revault, une architecture de vault multi-parties. Cette période a vu l'émergence de services spécialisés pour la gestion, la transmission et la diffusion de transactions pré-signées. Des prototypes fonctionnels ont été développés en Python, tandis que des recherches exploraient la faisabilité de vaults sans covenants, tout en identifiant les faiblesses inhérentes aux propositions initiales[^2].

### Développements majeurs (2021-2022)

Cette période a été caractérisée par l'intégration croissante avec Taproot et l'émergence de la proposition d'opcode OP_TAPLEAF_UPDATE_VERIFY, qui simplifierait considérablement certaines conceptions de vaults. Les chercheurs ont également exploré l'utilisation des signatures Schnorr avec OP_CAT comme méthode alternative de création de vaults. Les mécanismes de vault ont servi de cas d'étude de référence pour évaluer différentes conceptions de covenants, conduisant à l'élaboration de code fonctionnel pour un vault basé sur CTV.

### Développements récents (2023-2025)

L'année 2023 a vu la proposition des opcodes OP_VAULT et OP_UNVAULT, accompagnée d'une ébauche de BIP. Cette période a également été marquée par diverses explorations alternatives d'implémentation, notamment l'utilisation des annexes taproot pour le stockage des métadonnées liées aux vaults, une conception basée sur MATT, ainsi qu'un design alternatif inspiré par OP_TLUV.

En 2024, la BIP345 a formalisé la proposition des opcodes OP_VAULT, accompagnée d'autres modifications consensuelles[^4]. Cette même année a également vu l'apparition d'un prototype de vault exploitant OP_CAT combiné aux signatures Schnorr.

L'année 2025 a vu émerger des propositions d'amélioration de l'opcode CTV pour développer des vaults offrant une flexibilité accrue, ainsi que des avancées significatives dans l'utilisation d'OP_CAT pour les implémentations de vaults.

## Considérations techniques approfondies

### Conception scripturale et structures contractuelles

Les implémentations techniques des vaults reposent sur des structures conditionnelles `OP_IF/OP_ELSE` créant des chemins d'exécution distincts pour les différents scénarios (dépense standard, récupération d'urgence). Ces structures intègrent `OP_CHECKSEQUENCEVERIFY` pour imposer des contraintes temporelles relatives entre transactions, complétées par des schémas multisignatures établissant différents niveaux d'autorisation. Un avantage notable des designs actuels est la génération de témoins (witnesses) plus compacts comparativement aux propositions de covenants génériques.

### Modèle de menace et stratégies défensives

Le travail de Swambo et al. développe un modèle de menace sophistiqué identifiant divers vecteurs d'attaque et scénarios de défaillance potentiels[^3]. La défense en profondeur constitue le principe architectural fondamental, implémentant plusieurs mécanismes de sécurité indépendants qui fonctionnent en parallèle. Cette approche intègre des redondances délibérées pour éliminer les points uniques de défaillance et promeut la diversité des composants matériels et logiciels comme rempart contre les logiciels malveillants ciblant des plateformes spécifiques.

### Défis d'implémentation

Plusieurs défis pratiques demeurent à surmonter, notamment l'allocation dynamique des frais pour les transactions covenant dans un environnement où la mempool est fluctuante. La compatibilité avec les portefeuilles matériels existants représente également un obstacle significatif, tout comme la complexité inhérente aux mécanismes d'annulation de transactions. Un avantage notable de l'opcode OP_VAULT proposé serait la réduction du nombre d'étapes requises pour effectuer une dépense comparativement aux autres propositions[^4].

### Re-vaulting : sécurité par récursivité

Le concept de re-vaulting introduit un troisième chemin d'exécution dans chaque transaction vault, permettant de rediriger automatiquement les fonds vers une nouvelle structure de vault. Ce mécanisme ajoute une couche supplémentaire de sécurité en créant des barrières successives contre les tentatives d'extraction non autorisées, bien qu'il augmente proportionnellement la complexité opérationnelle du système.

## Implémentations de référence

### Revault

Mentionné comme implémentation pratique de référence, Revault propose une architecture de vault multi-parties distribuée. Son approche collaborative pour la gestion des fonds s'appuie sur des mécanismes de sécurité renforcés et une distribution des responsabilités qui minimise les risques de compromission. Le code source est disponible à [https://github.com/revault/revault_net](https://github.com/revault/revault_net).

### Python-vaults

Cette bibliothèque open-source fournit une infrastructure pour la construction d'arbres de transactions pré-signées, permettant aux développeurs d'explorer et d'implémenter différentes configurations de vault sans attendre les modifications du protocole Bitcoin. Le code source est disponible à [https://github.com/kanzure/python-vaults](https://github.com/kanzure/python-vaults).

## Analyse critique des avantages et inconvénients

Les Bitcoin Vaults offrent une protection substantielle contre le vol de clés privées, l'une des vulnérabilités les plus critiques dans l'écosystème Bitcoin. Leur capacité à détecter et neutraliser les attaques en cours d'exécution représente une avancée majeure dans la sécurisation des fonds numériques. Le mécanisme permet également de limiter les pertes même en cas de compromission partielle du système et supporte le traitement par lots lors des opérations de dépense ou de gel de multiples sorties.

Cependant, ces bénéfices s'accompagnent d'une complexité opérationnelle accrue pour l'utilisateur final et nécessitent souvent des configurations matérielles et logicielles spécialisées. Le risque de perte de fonds en raison d'erreurs procédurales ou de défaillances systémiques ne peut être écarté. De plus, tout système de sécurité représente fondamentalement un compromis entre protection contre la perte accidentelle et protection contre le vol délibéré, équilibre que chaque implémentation de vault doit calibrer selon son contexte d'utilisation.

## Conclusion

Les Bitcoin Vaults représentent une avancée architecturale significative dans la sécurisation des avoirs numériques, particulièrement adaptée aux détenteurs institutionnels ou individuels de quantités substantielles de Bitcoin. Leur approche temporelle et multi-transactionnelle introduit un paradigme de sécurité qui transcende les modèles traditionnels de signature unique ou multiple.

Les recherches actuelles démontrent que diverses options d'implémentation existent, certaines exploitant les fonctionnalités natives de Bitcoin tandis que d'autres anticipent des améliorations du protocole via des soft forks ciblés. L'avenir des vaults dépendra probablement de la trajectoire d'évolution du protocole Bitcoin lui-même, notamment concernant l'adoption d'opcodes comme OP_VAULT ou CTV.

L'architecture optimale, comme le souligne la recherche de Swambo et al., combine des principes fondamentaux de sécurité : défense stratifiée, redondance fonctionnelle et compartimentation des risques. Ce modèle permet un accès contrôlé aux fonds, limitant l'exposition aux menaces par une gestion granulaire des montants accessibles. La conception modulaire avec des interdépendances minimales entre composants, associée à une diversification des systèmes, constitue le fondement d'un protocole de garde résilient face à la sophistication croissante des vecteurs d'attaque.

## Références

[^1]: Bitcoin Optech. "Vaults." Bitcoin Optech. [https://bitcoinops.org/en/topics/vaults/](https://bitcoinops.org/en/topics/vaults/)

[^2]: Jorge Timón. "Covenants in Elements Alpha." Blockstream Blog, 2017. [https://blog.blockstream.com/en-covenants-in-elements-alpha/](https://blog.blockstream.com/en-covenants-in-elements-alpha/)

[^3]: Jacob Swambo, Spencer Hommel, Bob McElrath, and Bryan Bishop. "Custody Protocols Using Bitcoin Vaults." arXiv preprint arXiv:2005.11776, 2020. [https://arxiv.org/abs/2005.11776](https://arxiv.org/abs/2005.11776)

[^4]: James O'Beirne. "[bitcoin-dev] BIP Proposal: OP_VAULT and OP_UNVAULT." Bitcoin-dev mailing list, 2023. [https://gnusha.org/pi/bitcoindev/CABaSBawe_oF_zoso2RQBX+7OWDoCwC7T2MeKSX9fYRUQaY_xmg@mail.gmail.com/](https://gnusha.org/pi/bitcoindev/CABaSBawe_oF_zoso2RQBX+7OWDoCwC7T2MeKSX9fYRUQaY_xmg@mail.gmail.com/)
