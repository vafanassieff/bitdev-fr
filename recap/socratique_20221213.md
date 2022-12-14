Ce récapitulatif se veut un résumé, sommaire mais exhaustif, des discussions qui se sont tenues lors de la deuxième édition du BitDevFR. Un grand merci aux personnes présentes pour leur participation !

Ce récapitulatif est divisé en deux parties :
- un [récapitulatif général](#récapitulatif-général),
- un [récapitulatif par sujet](#récapitulatif-par-sujet) détaillant un peu plus chaque discussion.

## Récapitulatif général

Environ une douzaine de personnes étaient présentes pour ce second BitDev parisien. Pour une partie d'entre elles, les dicussions ont pu se poursuivre dans un bar situé à quelques mètres de la salle.

Sujets abordés :
- rappels succints sur la nature des BitDevs,
- [DLCs dans des canaux Lightning](#dlcs-dans-des-canaux-lightning),
- [des boucles For dans les scripts Bitcoin](#des-boucles-for-dans-les-scripts-bitcoin),
- [Merkelize All The Things (Matt)](#merkelize-all-the-things),
- [Ephemeral Anchors](#ephemeral-anchors)

A noter que des liens vers les ressources permettant de mieux comprendre les différents sujets figurent déjà dans le [document](https://github.com/vafanassieff/bitdev-fr/blob/master/events/socratique_20221213.md) utilisé le jour J, et ne sont donc pas reproduits ici.

## Récapitulatif par sujet

### DLCs dans des canaux Lightning

Crypto Garage a annoncé avoir ouvert et fermé avec succès pour la première fois un canal Lightning avec un DLC à l'intérieur.

Les difficultés d'ordre technique sont multiples, mais on peut notamment citer la nécessité de séparer les fonds du canal qui sont alloués au DLC et ceux qui restent disponible pour les transactions Lightning classiques. En effet, sinon, il faudrait recalculer toutes les signatures des transactions du DLC (potentiellement très nombreuses) à chaque fois qu'un paiement passe.

Les applications de DLCs à l'intérieur de canaux Lightning sont nombreuses. Cela permet par exemple de créer des contrats futures ou des options, et aussi de faire "rouler" ce type de contrats à l'intérieur du canal sans repasser on-chain, ce qui permet de créer des produits commes des futures perpétuels. Un autre exemple est la possibilité de simuler une balance stable en USD, en shortant le prix du Bitcoin dans le DLC.

### Des boucles For dans les scripts Bitcoin

Bitcoinbrillance a montré qu'il était possible de créer des boucles for ayant un nombre fini d'itérations dans Bitcoin Script, avec un exemple à 7 itérations. Les véritables applications ne sont pas évidentes pour l'instant, mais on pourrait par exemple imaginer jouer au morpion on-chain.

### Merkelize All The Things

Ressource : https://merkle.fun

L'idée générale de cette proposition est qu'il est possible de représenter n'importe quel "smart contract" sur Bitcoin avec des covenants "basiques" et beaucoup d'arbres de Merkle.

Il est alors possible de représenter de manière succinte les divers changements d'état. De plus, dans la majorité des cas, tout resterait off-chain, et le passage on-chain ne se ferait que pour trancher un différent entre deux parties. Cette étape est optimisée, là encore grâce à une structure en arbre de Merkle et l'utilisation de la méthode de la bissectrice : si Alice publie un état et que Bob le conteste, il suffit de descendre l'arbre jusqu'à trouver la feuille où leurs avis divergent, et il est alors facile (computationellement parlant) de voir qui a raison, ce qui signifie que ce passage on-chain n'exige pas beaucoup de calculs de la part de l'ensemble des noeuds. La fréquence de ces passages on-chain peut être réduite en requiérant d'Alice et de Bob qu'ils déposent une garantie s'ils décident de passer on-chain : celui qui avait tort voit son dépôt confisqué et/ou brûlé.

Matt requiert quelques nouveaux opcodes, et beaucoup de choses en sont encore à une phase préliminaire de réflexion. Cependant, l'une des conclusions intéressantes et qu'il est "facile" de créer des contrats arbitrairement complexes à partir de covenants somme toute très sommaires. Cela soulève donc encore la question des covenants dans Bitcoin : puisqu'il semble de toute façon possible d'accéder à des contrats avancés même avec des covenants simples (bien que cela dépende des covenants, voir par exemple OP_CTV/BIP119), autant designer les covenants pour pouvoir efficacement représenter ces contrats complexes.

### Ephemeral Anchors

L'idée générale est d'ajouter sur les transactions d'engagements Lightning un output de 0 satoshi qui puisse être dépensé par tout le monde, et tirer partie des règles de relai des transactions v3 en cours d'écriture pour utiliser cet output pour augmenter les frais de cette transaction *a posteriori*.

Un tel dispositif pourrait par exemple permettre à un Lightning Service Provider (LSP) de fee bump les transactions de fermeture de canaux ses clients sans intervention de leur part, et sans que cela requiert de tradeoff particulier.

Des hésitations ont été formulées quant aux potentielles vecteurs d'attaques (notamment par fee pinning) introduits par cette proposition, mais qui peuvent être atténué en mettant en place les règles appropriées (comme c'est le cas pour [CPFP carveout](https://fanismichalakis.fr/posts/anchor-outputs/#carve-out-and-anchor-outputs) par exemple).