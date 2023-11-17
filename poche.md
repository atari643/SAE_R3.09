# Une poche de chocolatines

Le NIST (National Institute of Standards and Technology, organisme de standardisation américain) organise depuis 2016 un concours international en vue de la standardisation d’algorithmes cryptographiques post-quantiques. Dans le cadre de ce concours, le NIST a [publié](https://www.ssi.gouv.fr/actualite/selection-par-le-nist-de-futurs-standards-en-cryptographie-post-quantique/) le 5 juillet dernier une liste de quatre premiers algorithmes sélectionnés.

Pourtant, un crypto-système qui semblait très prometteur n'a pas été retenu dans cette liste. Il s'agit d'un système de chiffrement original, qui utilise des poches et des pochons. Il a été proposé, il y a quelques années, par une jeune informaticienne bordelaise, Augustine Kifli, et nous allons voir comment fonctionne ce système dans le cadre de ce projet.

## Principe

Si on pose la question à `Copilot`, il nous raconte :

> Le principe de ce système est de chiffrer un message en le cachant dans un pochon. Le pochon est ensuite caché dans une poche. Le pochon est un objet qui peut contenir un message, mais qui ne peut pas être ouvert. La poche est un objet qui peut contenir un pochon, mais qui ne peut pas être ouvert. Le message est donc caché dans le pochon, qui est caché dans la poche.

Mais comme d'habitude, `Copilot` raconte n'importe quoi ! Il faudrait lire le papier original pour comprendre le fonctionnement de ce système. En attendant, en voici les grandes lignes.

### Un problème de poche

Comme nous l'avons vu en cours, afin de réaliser un crypto-système à clé publique, il faut s'appuyer sur un problème inversible mais fortement asymétrique d'un point de vue calculatoire.

Le problème asymétrique utilisé par Kifli est le suivant : étant donné $n$ entiers positifs $a_1$, $a_2$, ..., $a_n$ ainsi qu'un entier $c$, existe-t-il un choix judicieux parmi les $a_i$ tel que leur somme vaille exactement $c$ ?

Dit autrement, on se demande s'il existe des booléens $e_1$, $e_2$, ..., $e_n$ tels que $\sum_{i=1}^{n}{e_i \times a_i}=c$.

Cet ensemble de nombres $a_i$ est appelé une *poche*.

> On parle alors de "problème de poche" qui a été formalisé au 12ième siècle lors de compétitions organisées à Saint-Girons ayant pour objectif de chercher une sélection optimale de victuailles, de différentes tailles, achetées sur le marché des producteurs occitans locaux, afin de remplir exactement une poche d'une certaine capacité.

#### Exemple

Considérons le problème suivant : $a_1=366$, $a_2=385$, $a_3=392$, $a_4=401$, $a_5=422$, $a_6=437$ et la valeur cible est $c=1214$. Quels sont les $a_i$ (s'il en existe) qui, additionnés, valent $c$ ?

Une recherche rapide montre qu'une solution est $a2+a3+a6=385+392+437=1214=c$, soit $(e1,e2,e3,e4,e5,e6)=(0,1,1,0,0,1)$

#### Complexité

Une version, dite décisionnelle, de ce problème consiste juste à demander s'il existe de tels $e_i$, sans chercher à les calculer. Ce problème est considéré comme très difficile (en théorie de la complexité on parle de problème NP-complet) et on ne connaît pas d'algorithme polynomial en $n$ permettant de le résoudre dans tous les cas; il est de plus très improbable qu'un tel algorithme existe.

### Un problème de pochon

Une variante du problème précédent considère un ensemble de nombres $b_i$ dit **super-croissants** (on appelle alors cet ensemble un *pochon*), c'est-à-dire tels que $b_i>\sum_{j=1}^{i-1}{b_j}$. Dans ce cas, il existe une solution *très rapide* (il suffit de trier les nombres) au *problème de poche*, qui consiste à prendre le plus grand $b_i$ tel que $b_i\leq c$, puis à recommencer avec $c-b_i$.

> Un *pochon* est donc une *poche* avec des (super) croissants, aussi appelés *chocolatines*.

## Crypto-système de Kifli

On utilise le problème décrit précédemment et l'idée est d'utiliser un *pochon* comme clé privée et de construire une *poche* (équivalente mathématiquement) qui sera la clé publique.

> Une démonstration complète de ce crypto-système a été présentée lors de la conférence [Sparse Days](https://www.ladepeche.fr/2022/06/16/saint-girons-devient-la-capitale-mondiale-des-maths-pendant-trois-jours-10370416.php) à Saint-Girons, capitale couserannaise, et en présence de Jack Dongarra, prix Turing 2021, l'équivalent du Nobel pour les mathématiques !

### Génération des clés

On commence par construire la **clé privée** :

- Choisir un entier $n$ fixé suffisamment grand (par exemple $n=100$) qui va correspondre à la taille de la *poche* et du *pochon*. Ce sera également la taille du bloc élémentaire d'un message (binaire) à chiffrer.
- Générer un *pochon* $\mathcal{P}=\{b_1,b_2,...,b_n\}$ de taille $n$ (c'est-à-dire un ensemble de $n$ entiers super-croissants). Ce sera l'élément principal de la **clé privée** du crypto-système.
- Choisir un entier $M$ tel que $\sum_{i=1}^{n}{b_i} \lt M$.
On choisit ensuite au hasard un entier $W$, compris entre $1$ et $M-1$, et premier avec $M$.
- On choisit aussi au hasard une permutation $\sigma$ de l'ensemble $\{1,...,n\}$. Cette permutation sera utilisée pour mélanger les éléments du *pochon*. Le *pochon* $\mathcal{P}$, $M$, $W$, et $\sigma$ constituent la **clé privée**.

On peut maintenant construire la **clé publique** :

- On calcule $a_i = b_{\sigma(i)} \times W \mod M$ pour tout $i$ entre $1$ et $n$. L'ensemble $\mathcal{A}=\{a_1,a_2,...,a_n\}$ n'est clairement pas **super-croissant**, c'est une simple *poche*, mais elle est équivalente au $pochon$ $\mathcal{P}$, c'est la **clé publique**.

### Chiffrement

Si on souhaite envoyer un message constitué de la suite de $n$ bits $\{x_1,...,x_n\}$, on utilise la **clé publique** $\mathcal{A}$ et on calcule :

$$C=\sum_{i=1}^{n}{x_i \times a_i}$$

$C$ est le message chiffré, il est constitué d'un seul nombre entier. Si quelqu'un intercepte $C$ et connait la **clé publique** $\mathcal{A}$, il ne peut remonter aux $x_i$, car la résolution résolution de ce problème est réputée difficile (NP-complet).

### Déchiffrement

Le récepteur du message, qui connait la **clé privée** $\mathcal{P}$, retrouve les $x_i$ de la façon suivante. Il calcule d'abord $D = W^{-1} \times C \mod M$. Il résout ensuite le problème de *poche de (super) croissants* avec le poids total $D$, et les poids $\{b_1,...,b_n\}$ du *pochon* $\mathcal{P}$. Il calcule ainsi des nombres $e_i \in \{0,1\}$ tels que :

$$\sum_{i=1}^{n}{e_i \times b_i}=D$$

Il en déduit enfin les bits $x_i$ du message original par la relation $x_i = e_{\sigma(i)}$.

> Magnifique !

### Exemple

Essayons donc maintenant ce crypto-système sur un exemple concret :

- On choisit $n=5$ et le *pochon* $\mathcal{P}=\{2,5,11,23,55\}$. On choisit $M=113$ et $W=27$ qui est bien premier avec $M$. On choisit la permutation $\sigma$ telle que $\sigma(1)=2$, $\sigma(2)=5$, $\sigma(3)=3$, $\sigma(4)=1$ et $\sigma(5)=4$.

- On construit la poche $\mathcal{A}$ en calculant $a_i = b_{\sigma(i)} \times W \mod M$ pour tout $i$ entre $1$ et $n$. On obtient $\mathcal{A}=\{22, 16, 71, 54, 56\}$.

- On veut envoyer le message binaire $10101$. On transmet donc : $C=a_1+a_3+a_5=149$.

- Pour retrouver le message binaire en clair envoyé, on calcule tout d'abord $W^{-1}$ par l'algorithme d'Euclide, et on trouve $W^{-1}=67$ (modulo $113$).

- On calcule $D=W^{-1} \times C \mod M = 39 \mod 113$. On décompose alors $39$ sur le *pochon* $\mathcal{P}$ et on trouve :

$$
\begin{align*}
39 &= 23+11+5 \\
        &= b_4 + b_3 + b_2 \\
        &= b_{\sigma(5)} + b_{\sigma(3)} + b_{\sigma(1)}
\end{align*}
$$

- Le message initial était donc $10101$. $\square$

## Implémentation

Le module R3.09 de cryptographie sera, pour partie, évalué par le travail réalisé dans le cadre de ce projet (l'autre partie de l'évaluation étant une interrogation portant sur les principaux concepts de la cryptographie et de la sécurité informatique).
**On vous demande de travailler en équipe (2 ou 3 étudiants)**.

### Consignes

A partir des spécifications proposées au cours du module R3.09
(en particulier : *exponentiation modulaire*, *PGCD*, *Bezout/Euclide*, ...),
**on vous demande d'implémenter le crypto-système de Kifli**.
Vous devrez en particulier écrire les fonctions suivantes :

- `gen_pochon(n)` qui génère un pochon de taille $n$.

- `gen_cle_privee(n)` qui génère une clé privée.

- `gen_cle_publique(cle_privee)` qui génère la clé publique à partir de 
la clé privée.

- `solve_pochon(pochon, c)` qui résout un problème de *pochon*.

- `chiffre(message, cle_publique)` qui chiffre un message.

- `dechiffre(message_chiffre, cle_privee)` qui déchiffre un message.

- `test_chiffrement_dechiffrement(message, n)` qui chiffre et déchiffre un message et vérifie que le message déchiffré est bien égal au message initial.

D'après vous, le NIST a-t-il eu raison de ne pas retenir le crypto-système de Kifli dans sa liste d'algorithmes post-quantique ? **Justifiez votre réponse**.

### Rendu

Ce travail est à rendre, au plus tard, le **vendredi 1er décembre 2023**, dans un projet hébergé sur le *Gitlab* de l'IUT. Pensez à inviter (avec un rôle $\ge$ *Reporter*) votre enseignant responsable du module R3.09.
