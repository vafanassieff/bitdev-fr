Ce récapitulatif se veut un résumé, sommaire mais exhaustif, des discussions qui se sont tenues lors de la deuxième édition du BitDevFR. Un grand merci aux personnes présentes pour leur participation !

Ce récapitulatif est divisé en deux parties :
- un [récapitulatif général](#récapitulatif-général),
- un [récapitulatif par sujet](#récapitulatif-par-sujet) détaillant un peu plus chaque discussion.

## Récapitulatif général

Environ une quinzaine de personnes étaient présentes pour ce second BitDev parisien. Pour une partie d'entre elles, les dicussions ont pu se poursuivre dans un bar à une dizaine de minutes de la salle.

Sujets abordés :
- rappels succints sur la nature des BitDevs,
- annonce de la date et du lieu du troisième BitDevFR, qui se tiendra à Biarritz le 25 août 2022, à l'occasion de [Surfin'Bitcoin](https://surfinbitcoin.com/)
- [discussion](#unspent-capacity-whirlpool) sur la "Unspent Capacity" de whirlpool (nouvel ATH), qui a dérivé sur une discussion plus générale,
- discussion sur un nouveau [mécanisme anti-sybil](#riddle),
- [discussion](#taproot-et-musig2) sur Taproot (adoption, parité) et Musig2,
- [Hertzbleed](#hertzbleed),
- [RGB et MyCitadel](#rgb-et-mycitadel),
- [Package Relay](#package-relay),
- [Simple Multiparty Schorr signing](#simple-multiparty-schorr-signing).

A noter que des liens vers les ressources permettant de mieux comprendre les différents sujets figurent déjà dans le [document](https://github.com/vafanassieff/bitdev-fr/blob/master/events/socratique_20220705.md) utilisé le jour J, et ne sont donc pas reproduits ici.


## Récapitulatif par sujet

### Unspent Capacity Whirlpool

La "Unspent Capacity" du protocole de coinjoin *whirlpool* (sur le coordinateur principal) avait atteint le 30 juin un nouveau plus haut, avec une valeur totale de presque 4818 bitcoins. L'occasion de rappeler la signification de cette métrique, qui représente le volume total d'UTXOs présents dans le pool de liquidité global à un instant donné, et donc une certaine mesure de l'anonomity set de whirlpool.

La discussion a ensuite dévié sur le sujet des coinjoins de manière plus générale, avec notamment le sujet de JoinMarket. Il s'agit d'un autre protocole de coinjoins, au fonctionnement relativement différent de celui de Whirlpool. L'une des disctinctions principales est la notion de maker / taker présente dans JoinMarket mais totalement absente de Whirlpool. Les makers mettent à disposition leurs UTXOs pour les coinjoins, créés par les takers, qui en échange paient une commission aux makers.

Le sujet de la résistance de JoinMarket aux attaques Sybil, où une même entité pourrait se faire passer pour une multitude de makers et éventuellement désanonymiser certains coinjoins, a conduit à l'implémentation d'un mécanisme anti-sybil par le biais de "Fidelity Bonds". Un Fidelity Bond est une représentation d'un certain nombre de bitcoins bloqués au moyen d'un timelock. Le problème de ce mécanisme, en l'état, est qu'il donne l'avantage aux makers ayant beaucoup de bitcoins placés dans des Fidelity Bonds, car la probabilité d'être choisi en tant que maker est proportionnelle au carré du nombre de bitcoins placés dans un bond. Ainsi, plusieurs des participants au BitDev ont remarqué que, depuis l'introduction de ce mécanisme, ils sont beaucoup plus rarement sélectionnés pour participer à des coinjoins.

Cela a naturellement mené au sujet suivant, qui concernait une nouvelle proposition de mécanisme anti-sybil.

### RIDDLE

RIDDLE est une proposition de mécanisme anti-sybil "légère", en cela qu'elle ne requiert pas l'immobilisation d'UTXOs, à la différence des Fidelity Bonds.

L'idée générale est qu'un utilisateur souhaitant utiliser un service doit au préalable prouver qu'il contrôle une UTXO, au moyen de la signature correspondante. Un semblant de confidentialité est conservé en utilisant des *ring signatures* pour dissimuler l'UTXO en question au sein d'un groupe d'UTXO semblables : le vérficateur peut alors s'assurer que l'utilisateur contrôle bien au moins une des UTXOs du groupe, mais il ne peut pas savoir exactement laquelle. En outre, puisqu'il s'agit seulement de transmettre des signatures, l'empreinte on-chain de ce mécanisme est nulle.

Pour qu'un mécanisme anti-Sybil/anti-spam soit efficace, il faut qu'intervienne quelque part un coût, qui reste minime pour un utilisateur standard mais grimpe rapidement pour un utilisateur qui effectuerait beaucoup de requêtes de manière anormale. Cela est atteint en jouant sur les conditions que doit vérifier l'UTXO, qui varie en fonction des services et des cas d'usage. Par exemple, un service souhaitant limiter le nombre de multi-comptes pourrait requérir une signature correspondant à une UTXO d'une certaine taille (par exemple : au moins 0.01 BTC) et chaque UTXO pourrait n'être utilisable qu'une seule fois. Un utilisateur pourrait bien sûr créer plusieurs comptes s'il dispose de plusieurs UTXOs de plus de 0.01 BTC, mais un nombre limité. Il pourrait éventuellement créer à dessein un grand nombre d'UTXOs ayant juste la taille requise pour maximiser le nombre de comptes, mais cela nécessiterait des transactions on-chain, avec le coût associé.
A l'inverse, un cas d'usage typiquement plus souple, comme du rate-limiting sur une API publique, pourrait ne requérir qu'une nouvelle signature toutes les x requêtes, sans imposer de conditions particulières sur l'UTXO.

### Taproot et Musig2

La discussion a ensuité porté sur Taproot, dont un récent rapport de Bitmex Research montrait que l'adoption ne décollait pas encore tout à fait. L'occasion d'évoquer également Musig2 et son utilisation de Taproot, rendue plus difficile par une décision prise lors du développement de Taproot de toujours considérer la clé privée (et donc la clé publique) comme étant du même signe, alors que traditionnellement les deux signes sont possibles. Cela occasionne des difficultés pour les développeurs qui construisent avec Taproot, car il faut en permanence s'assurer d'adapter le signe du résultats des différentes opérations cryptographiques lors de la génération de clés. En contrepartie, ce choix permet de supprimer l'octet de signe de la clé publique et donc d'alléger légèrement les transactions. Pour plus de détails, voir cet [article](https://medium.com/p/f86476af05d7) de Blockstream, qui date de 2019 mais est toujours d'actualité.

### Hertzbleed

Hertzbleed est un nouveau type d'attaque par canal auxiliaire qui avait été à la source d'une certaine agitation dans l'écosystème Bitcoin, car elle permettait potentiellement de déduire une clé privée en analysant la fréquence de certains processeurs lors de la signature. On sait maintenant qu'on peut se rassurer  dans la majorité des cas en ce qui concerne Bitcoin, car :
- l'attaque exploite la technologie "Turbo Boost" d'Intel (ou équivalent chez d'autres) qui permet d'augmenter dynamiquement la fréquence du processeur pour en améliorer les performances. Désactiver cette technologie dans le BIOS permet donc de se prémunir de l'attaque, si besoin,
- l'attaque a besoin d'observer le processeur un grand nombre de fois pendant qu'il signe avec une clé privée donnée (il s'agit d'une attaque statistique). Ainsi, seuls les cas d'usage où le nombre de signatures avec une même clé est élevé sont vulnérables. On peut par exemple penser à certains gros neouds Lightning, qui effectuent potentiellement plusieurs opérations de signature avec la même clé privée par seconde. Dans les autres cas, ce vecteur d'attaque est a priori sans conséquence.

### RGB et MyCitadel

Nous avons ensuite brièvement discuté de RGB, dont la sortie de nouveaux éléments venait d'être retardée (ils ont fini par être livrés à l'heure d'écriture de ces lignes). L'occasion d'évoquer également MyCitadel, le *wallet* Bitcoin, Lightning et RGB développé par Pandora Corp. Au moment du BitDev, seule la partie Bitcoin était disponible, mais était déjà très intéressante avec la possibilité de créer simplement des schémas complexes, alliant multisignature et timelocks, le tout en tirant partie de Taproot et des signatures de Schnorr.

### Package Relay

Une modification des règles de gestion du mempool au niveau des noeuds appelée "Package Relay" a été rapidement discutée, essentiellement en ce qui concerne sa motivation, qui est de permettre d'ajuster les frais de transactions avec CPFP de manière plus claire et prévisible (utile notamment pour Lightning). L'idée générale de Package Relay est de traiter les transactions qui dépendent les unes des autres en paquets, et de regarder les frais par byte du paquet global plutôt que transaction par transaction.

### Simple Multiparty Schorr signing

Un papier de recherche en cryptographie concernant un protocole de signature de Schnorr impliquant plusieurs parties a été rapidement évoqué en fin de session.