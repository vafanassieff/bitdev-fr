L'objectif de ce récapitulatif est de fournir un aperçu général des discussions lors du BitDev. Il s'agit bien entendu d'un résumé sommaire ne rapportant que les grandes lignes des discussions très riches qui se sont tenues. Nous tenons d'ailleurs à remercier l'ensemble des participants pour cette première édition, qui n'a été réussie que grace à leurs apports d'une grande qualité.

Ce récapitulatif est divisé en deux parties :
- un [récapitulatif général](#récapitulatif-général),
- un [récapitulatif par sujet](#récapitulatif-par-sujet) détaillant un peu plus chaque discussion.

## Récapitulatif général

Environ une quinzaine de personnes étaient présentes pour ce premier BitDev parisien. Pour une partie d'entre elles, les dicussions ont pu se poursuivre dans un bar à proximité de la salle.

Sujets abordés :
- explications sur la nature des BitDevs, les quelques règles à avoir à l'esprit (principalement l'absence de capture d'image ou de son), la fréquence mensuelle prévue, ainsi que la date de la prochaine édition, fixée au 5 juillet 2022,
- discussions sur [Lightning Network](#lightning) :
    - évolution de la capacité annoncée, avec le franchissement récent de la barre des 4000 bitcoins déployés dans des canaux annoncés (publics),
    - probing sur le réseau,
    - gossip,
    - les canaux devraient-ils être non-annoncés par défaut au niveau des implémentations ?
    - outils d'automatisation
- [Comment démarrer un groupe pour échanger des bitcoins](#comment-démarrer-un-groupe-pour-échanger-des-bitcoins)
- [CoinPool](#coinpool) (venu naturellement à partir de la discussion précédente sur les échanges de Bitcoin)
- [Silent Payments](#silent-payments)
- [CoinSwap](#coinswap)
- [Ordinal Numbers](#ordinal-numbers) (rapidement)

## Récapitulatif par sujet

### Lightning

La discussion sur Lightning est partie de l'observation de l'évolution de la [capacité annoncée](https://bitcoinvisuals.com/ln-capacity), où il a notamment été noté que cette capacité a récemment (le jour même du BitDev) dépassé les 4000 BTC, ce qui place à titre de comparaison Lightning au dessus du réseau Liquid mais en-dessous du protocole Whirlpool.

Elle s'est ensuite porté sur les problèmes de maintenance des noeuds LND en raison de l'explosion de la taille de la base de données, qui devraient être corrigés dans la prochaine mise-à-jour, toujous à l'état de *release candidate* au moment de la tenue du BitDev. La pratique par certains noeuds du *probing* ("sondage" en français) a également un impact notable sur ce problème, puisqu'elle est responsable d'une partie de l'expansion de la taille des bases de données des noeuds LND. Pour rappel, le *probing* consiste à émettre un faux paiement[^1] afin de mesurer où est-ce qu'il rate. Si le paiement rate en cours de route, il est possible d'estimer la répartition de la liquidité au sein d'un canal donné. En émettant plusieurs paiements sondes avec des montants différents sur une même route, il devient possible d'estimer cette répartition de plus en plus finement. Cette pratique, qui peut par exemple être utilisée à des fins de déanonymisation des paiements, ou alors afin d'obtenir des informations sur une route avant de tenter d'y propager un paiement afin de maximiser le taux de succès des paiements, pose certains problèmes :
- occupe inutilement de la place en base de données, notamment pour les noeuds LND qui conserve l'intégralité de la charge utile du paquet onion du paiement,
- bloque inutilement des HTLCs pendant certaines durées, ce qui peut bloquer complètement un canal quand le nombre maximal de HTLCs simultanés (une dizaine en pratique) est atteint,
- Les HTLCs bloqués peuvent expirer avant d'avoir été annulés (si le pair est hors-ligne par exemple), forçant la fermeture du canal.

La discussion a naturellement évolué sur les difficultés de scaling de l'approche "gossip" du Lightning Network, où chaque changement de l'état public (frais, taille maximale de HTLC, statut ouvert ou fermé, etc.) doit être propagé à l'ensemble du réseau. De fait, cette propagation est loin d'être instantanée, et peut s'avérer couteuse en ressources pour les noeuds. Des approches alternatives comme Ant Routing sont évoquées.

Les difficultés rencontrées par le gossip sur Lightning pourraient être atténuées en restreignant le nombre de canaux annoncés (aussi appelés canaux publics). Une piste évoquée serait de changer le statut par défaut des canaux lors de leur ouverture. En effet, aujourd'hui les 3 implémentations majoritaires (LND, C-LN et Eclair) ouvrent par défaut des canaux annoncés. Il pourraît être pertinent d'ouvrir plutôt des canaux non-annoncés par défaut, et l'utilisateur qui souhaite réellement proposer ses canaux pour le routage de paiement pourrait alors manuellement ouvrir le canal en l'annonçant. De la sorte, les personnes qui rejoignent le réseau pour payer et recevoir des paiements (et pour qui des canaux non annoncés sont donc suffisants) n'annonceraient par leurs canaux par défaut, soulageant ainsi l'ensemble des noeuds.

Les outils d'automatisation de noeuds Lightning ont été rapidement évoqués. L'utilisation de l'autopilot de LND semble déconseillée. L'attrait de solutions qui permettrait à un utilisateur de pouvoir payer et recevoir sans se soucier lui-même de gérer ses canaux a été réaffirmé.

### Comment démarrer un groupe pour échanger des bitcoins

Cet [article](https://juraj.bednar.io/en/blog-en/2022/03/14/how-to-create-your-own-crypto-trading-group/) évoque la formation de groupes de discussion sur des application de messagerie instantanées afin de tenter de se faire rencontrer acheteurs et vendeurs. Ce type de groupes pose bien évidemment des problèmes de confiance, et il s'agit donc généralement de groupes relativement restreints, avec cooptation.

Cette discussion sur l'achat/vente de bitcoins en pair-à-pair a naturellement évolué vers une discussion sur CoinPool, puisque ce protocole s'y prête assez bien.

### CoinPool

En effet, CoinPool est une proposition qui vise à permettre la propriété partagée entre plusieurs personnes d'une seule et unique UTXO. Du point de vue des participants à un CoinPool donné, l'UTXO est en réalité découpée en parts, et l'attribution de ces parts est traitée de manière off-chain entre les participants du CoinPool. Il est également possible de réaliser des transferts au sein d'un même CoinPool. C'est ainsi que ce protocole pourrait jouer un rôle dans les dispositifs d'échanges de bitcoins en pair-à-pair : CoinPool permettrait par exemple à Alice d'échanger du bitcoin contre des euros avec Bob avec une confidentialité élevée s'ils font tous deux partie du même CoinPool. Alice pourrait envoyer les bitcoins à Bob de manière off-chain au sein du CoinPool, tandis que Bob pourrait utiliser des espèces. De la sorte, seuls les participants du CoinPool seraient au courant du mouvement de fonds, au lieu de l'ensemble du réseau Bitcoin.

L'un des inconvénients des CoinPool est que les transactions off-chain ne peuvent avoir lieu qu'au sein d'un même CoinPool. Ainsi, pour envoyer des fonds à quelqu'un d'autre qu'un des participants du CoinPool, il faut effectuer un retrait on-chain. Il pourrait cependant être possible de se passer de retrait on-chain en "routant" la transaction au sein de plusieurs CoinPools qui se superposent. Par exemple, si Alice, Bob, Carol et Daniel se trouvent dans le CoinPool α, tandis que Daniel, Eve et Fanny se trouvent dans le CoinPool β, et qu'Alice souhaite payer Fanny, il est théoriquement possible pour Alice de faire une transaction off-chain à Daniel au sein du CoinPool α, et Daniel peut ensuite répercuter le paiement jusqu'à Fanny au moyen d'une transaction off-chain au sein du CoinPool β.

### Silent Payments

Le groupe a ensuite abordé le sujet des Silent Payments. Il s'agit d'un protocole qui vise à permettre l'utilisation d'une unique adresse de réception statique permettant *in fine* d'effectuer des transactions vers une multitude d'adresses Bitcoin. Le fonctionnement est analogue à celui de [BIP0047](https://bips.xyz/47), la différence principale étant que, là où BIP0047 utilise l'OP_RETURN d'une transaction Bitcoin pour communiquer le "secret" entre l'émetteur et le destinataire du paiement, Silent Payments utilise directement des informations naturellement contenues dans la transaction de paiement en elle-même.

En effet, l'idée centrale dans les deux protocoles (et d'ailleurs aussi dans les Whisper Addresses) est que le payeur récupère le code statique (par exemple sur le site internet du destinataire), et qu'il *tweak* ce code avec un nombre pour obtenir l'adresse Bitcoin de destination. Il faut ensuite qu'il communique ce nombre au destinataire, pour que celui-ci puisse calculer la clé privée correspondant à l'adresse et récupérer les fonds. Dans BIP0047, cette communication est réalisée une fois pour toute entre deux peers donnés au moyen du OP_RETURN d'une transaction dite "de notification". Dans Silent Payments, le nombre utilisé pour le *tweak* est en fait l'identifiant de transaction de l'un des inputs de la transaction de paiement en elle-même. Il suffit donc au destinataire, à partir du moment où il publie son code statique et s'attend donc à recevoir des paiements, de scanner l'ensemble des nouvelles transactions et de vérifier si, pour chaque input de chaque transaction, en effectuant le tweak à l'aide du code et de la *txid*  de l'input, il tombe sur l'une des adresses en output. Si tel est le cas, alors il s'agit bien d'un paiement qui lui est destiné, et il peut de même calculer la clé privée à l'aide d'un tweak similaire (sur la clé privée cette fois-ci). Si le destinataire fait déjà tourner un *full node*, même de manière discontinue (par exemple une heure tous les soirs), il est peu coûteux d'effectuer cette vérification supplémentaire sur chaque input en plus des vérifications habituelles sur la validité des transactions. Ce coût en temps de calcul pourrait encore être abaissé au besoin en effectuant le tweak avec les *txid* de tous les inputs de la transaction de paiement, afin de n'avoir qu'une seule vérification à faire par transaction, au lieu d'une vérification par input.

### CoinSwap

Le groupe a ensuite brièvement discuté de CoinSwap, un protocole d'amélioration de la confidentialité sur Bitcoin, conçu et en cours d'implémentation par Chris Belcher. L'idée générale est que deux utilisateurs peuvent "échanger" leurs UTXOs, sans qu'il soit possible pour un observateur extérieur de déterminer s'il s'agit d'un CoinSwap ou de transactions classiques. Un utilisateur souhaitant effectuer un CoinSwap afin d'améliorer sa confidentialité on-chain (*taker*) peut trouver des contre-parties (*makers*) avec qui effectuer le swap sur divers serveurs (le fonctionnement du matching taker - maker rappelle celui de JoinMarket, sur lequel Chris Belcher a également travaillé). Pour accroitre sa confidentialité et se protéger de l'indiscrétion du *maker*, le *taker* peut "router" son CoinSwap au travers de plusieurs *makers*, avec de multiples swaps successifs et/ou parallèle où chaque *maker* ne sait pas s'il traite avec le *taker* originel ou un autre *maker*, et où les risques de corrélation de montant ou temporelle peuvent être grandement réduits.

Le protocole est intéressant, mais l'avis général de l'audience semble être qu'il ne substitue pas en totalité aux outils existants comme les transactions jointes (*CoinJoins*), et doit donc plutôt être apprécié comme un outil supplémentaire et complémentaire (par exemple entre deux rounds de *CoinJoin*).

Il faut également noter que l'implémentation actuelle effectue chaque swap au travers d'une adresse multisignature 2-pour-2, ce qui ne correspond pas à l'objectif final où un swap est indissociable d'une simple transaction classique. Il s'agit donc toujours d'un travail en cours.

### Ordinal Numbers

En dessert, nous avons rapidement discuté d'une proposition intitulée "Ordinal Numbers" et visant à attribuer à chaque satoshi un numéro de série lors de sa création par un mineur. L'idée est que, puisque Bitcoin est, par certains aspects, un protocole de traçabilité (des transactions et des coins), il peut faire sens de vouloir renforcer cette-dernière. La proposition ressemble fortement à une proposition similaire formulée en 2012 sur [Bitcointalk](https://bitcointalk.org/index.php?topic=117224.0), et qui avait déjà à l'époque suscité un accueil pour le moins réservé.

[^1]: C'est à dire un paiement dont le hash ne correspond à aucune *preimage* prégénérée, et qui ne pourra donc pas être *settle* par le destinataire et "échouera" systématiquement.