Rapport de Projet : Incendie et propagation du feu

## Présentation
### Contexte
Ce projet s’inscrit dans le cadre de la Programmation Agent, nous allons étudier la dynamique d’un incendie de forêt et les interactions entre la propagation du feu et l’environnement avec comme agents : les pompiers, les gardes et les pyromanes.
Cette simulation se déroule dans le logiciel NetLogo : il est idéal pour modéliser des systèmes complexes impliquant un grand nombre d’agents et leurs interactions locales, avec une facilité de variation de certains paramètres pour tester les scénarios possibles.

### Objectifs
* **Modéliser les comportements :** Implémentation de machines à états finis pour chaque type d’agent.
* **Simuler la dynamique :** Reproduire les effets environnementaux.
* **Analyse :** Tester l’impact des paramètres et en déduire des solutions.

---

## Architecture des agents
### Description des agents
| Agent | Rôle | Attributs |
| --- | --- | --- |
| patch | Gérer l’état du terrain et les dynamiques | `est-foret?`, `est-terre?`, `est-base?`, `en-feu?`, `est-crame?`, `est-eteint?`, `en-pousse?`, `taille`, `cpt-repousse`, `cpt-reprise-pousse`, `odeur-pyro`, `durabilite-base`, `etat-base` |
| pompier | Eteindre les feux | `eau-restante`, `etat`, `vitesse`, `a-eteindre` |
| garde | Traquer et capturer les pyromanes | `etat`, `vitesse` |
| pyromane | Déclencher des feux | `etat`, `vitesse`, `mon-garde` |

### Dynamiques et interactions des agents
* **Pompier :** La dynamique du pompier est hiérarchique, la condition `eau-restante` $\le 0$ force l’état `en-ravitaillement`, et la condition `urgence-base?` force l’état `en-urgence?` avant même de chercher des patches en feu.
  * **Règle de ravitaillement :** Si ma réserve est vide alors je me ravitaille.
  * **Règle de patrouille :** Si ma réserve n’est pas vide et que la base ne brûle pas et que je ne vois pas de feu alors je patrouille.
  * **Règle d’extinction :** Si ma réserve n’est pas vide et que la base ne brûle pas et que je vois un feu alors j’y vais et je l’éteint.
  * **Règle d’urgence :** Si ma réserve n’est pas vide et que la base brûle alors je passe en état d’urgence.

* **Garde :** La dynamique du garde repose sur la perception, il se met en poursuite d’une cible (`en-poursuite`) dès que celle-ci rentre dans son rayon de détection après l’avoir traqué grâce à l’odeur qu’elle dépose (`en-suivi-odeur`). Une fois la cible trouvée, l’arrestation devient prioritaire et il amène l’agent capturé jusqu’à la base (`en-arrestation`).
  * **Règle d’arrestation :** Si j’attrape un pyromane (en étant assez proche de lui) alors je l’amène à la base pour le capturer.
  * **Règle de patrouille :** Si je n’arrête personne et que je ne poursuis personne et que je ne traque personne alors je patrouille.
  * **Règle de suivie odeur :** Si je n’arrête personne et que je ne poursuis personne et que je sens une odeur alors je la suis.
  * **Règle de poursuite :** Si je n’arrête personne et que je détecte un pyromane dans mon rayon de détection alors je poursuis.

* **Pyromane :** La dynamique des pyromanes est axée sur la survie et la pyromanie, fuir (`en-fuite`) est priorisé face à l’action par défaut (`en-pyromanie`). Une fois capturé (`en-capture`) il passe orange et suit le garde qui l’a attrapé (`mon-garde`).
  * **Règle capturé :** Si je suis capturé alors je suis mon garde et si je suis dans la base alors je meurs.
  * **Règle de pyromanie :** Si je ne suis pas capturé et que je ne vois pas de menace alors je met le feu.
  * **Règle de fuite :** Si je ne suis pas capturé et que je détecte une menace alors je fuis.

* **Patches :** Les patches sont au centre de la simulation et la rendent vivante. Tout d’abord nous décidons quels patches font partie de la base (`est-base?`) pour pouvoir y placer les gardes et les pompiers au setup. Ensuite le reste des patches est soit terre (`est-terre?`) soit forêt (`est-forêt?`), pour donner un effet réaliste et permettre de temporiser l’arrivée du feu, car seuls les patches forêt de taille maximum peuvent être mis en feu par les pyromanes. Après propagation le patch de terre devient forêt en pousse (`en-pousse?`) puis termine sa quand la taille maximale est atteinte (la taille augmente à chaque tick), réglé par un slider (`taille-max`). Cela est visible par un dégradé du vert clair au foncé, en allant de la taille 0 à la taille max, tandis que la terre est marron. Le feu agit de manière progressive sur les patches de forêt et de la base, tous les deux touchés de manières différentes. En effet, l’effet d’inflammation des patches de forêt passe du rouge au rouge foncé lorsque le patch perd de la taille, jusqu’à devenir cramé (`est-crame?`) lorsqu’il atteint la taille 0. Ensuite se passe un petit temps avant que le patch redevienne forêt de taille 0. Pour les patches de base nous sommes partis sur un système de durabilité (`durabilite-max-base`), l’effet de brûle est montré comme ceci : détruit, endommagé, indemne, critique. Un patch de base enflammé perd de la durabilité continuellement, en passant d’un état de durabilité au suivant, puis devient détruit à 0% de durabilité.

---

## Analyses
### Scénarios
* **Scénario 1 : Victoire des pyromanes, la base est détruite et tous les gardes sont morts.**
  Nous assistons à une victoire des pyromanes si il n’y a pas assez de pompiers et que tous les gardes meurent du feu. Pour ça nous avons implémenté un bouton toggle (`mort-feu`) qui tue les gardes s’ils sont dans le feu, ceci donne plus de chance aux pyromanes de l’emporter. Aussi, nous voyons facilement une victoire de leur part :
  * En baissant le nombre de pompiers (`nombre-pompiers`), leur réserve d’eau max (`reserve-max-pompier`) et en baissant la durabilité max de la base (`durabilite-base-max`).
  * En augmentant les chance de mettre le feu à un patch (`chance-declencher-feu`) et en augmentant la vitesse à laquelle le feu se propage (`vitesse-propagation-feu`).

* **Scénario 2 : Victoire des pompiers/gardes, tous les pyromanes sont capturés/morts et il n’y a plus de feu.**
  Cette victoire est causée par l’arrestation de tous les pyromanes, de ce fait les chances augmentent avec le nombre de gardes et l’augmentation de leur rayon de détection (`rayon-detection-garde`) ainsi qu’en les rendant immunisés au feu, et en rendant les pyromanes non-immunisés. La victoire est principalement causée par :
  * Le nombre de pompiers et une réserve d’eau élevées, une taille de la base plus grande avec une durabilité haute.
  * Un nombre de pyromanes bas avec une chance de mettre le feu basse et une vitesse de propagation du feu basse, ainsi qu’en rendant les pyromanes sensibles au feu, de ce fait pas besoin de tous les arrêter s’ils sont encerclés par les flammes.

* **Scénario 3 : Equilibre, les pompiers éteignent le feu progressivement pendant que les pyromanes continuent de mettre le feu, aussi la base est protégée.**
  Nous arrivons à ce résultat simplement en mettant tous les sliders au maximum et en mettant le nombre de gardes à 0. Ainsi les pyromanes restent indéfiniment et les pompiers continuent d’éteindre le feu. A force d’expérimentation, nous avons trouvé que le ratio 1:10 de pyromanes-pompiers permet d’arriver à ce résultat. C’est pourquoi le maximum de pompiers est de 200 et le maximum de pyromanes est de 20.

Pour finir nous avons rajouté des graphes et des monitor pour mieux voir comment avance la simulation et l’impact qu’ont les agents sur celle-ci. Le graphe « Efficacité des pompiers » montre le nombre de patches éteint à chaque tick. Le graphe « Gardes vs Pyromanes » montre l’incidence qu’ont les gardes sur les pyromanes, on peut suivre l’activité des pyromanes en fonction de la pression que les gardes mettent sur eux, quand ils fuient ils ne mettent pas le feu.

---

## Gestion projet
### Répartition des tâches
Au départ, nous avons décidé ensemble de la direction de base du projet via une sorte de brainstorming avec un bloc note. Puis nous avons commencé chacun de notre côté en faisant un squelette du projet. Ensuite nous avons choisi la version qui nous plaisait le mieux, et l’avons améliorée au fur et à mesure en rajoutant chacun notre tour des fonctionnalités, et en corrigeant celles des autres après communication. Nous avons utilisé le gestionnaire des sources de gitlab pour rester à jour, et permettre d’expliquer ce qui a changé entre chaque version.

### Difficultés
Le feu est de loin la plus grand difficulté que nous avons pu rencontrer, il est très instable de base et nous avons passé du temps à discuter de comment faire pour le gérer au mieux. Le changement avec le plus d’effet fut de changer l’impact des pompiers, au départ éteindre un patch, puis un cône en face et nous avons finalement opter pour un effet en rayon autour d’eux, car les deux premières méthodes n’étaient pas concluantes. Les interactions des agents étaient aussi un défi, nous avons eu du mal à gérer le sens des priorités des états, faire en sorte que chaque agent ayant les mêmes états ne se gênent pas, en particulier le système de capture. Il arrivait que certains agents se bloquaient complètement entre deux états, ou n’arrivaient plus à repasser dans l’état principal car une condition les en empêchait.

---

## Processus d'utilisation

Pour exécuter et utiliser cette simulation NetLogo :

1. **Prérequis :** Téléchargez et installez l’application [NetLogo](https://ccl.northwestern.edu/netlogo/) (version 6.x recommandée).
2. **Récupération du projet :** Clonez ou téléchargez ce dépôt contenant le fichier source du modèle NetLogo (`.nlogo`).
3. **Lancement :**
   - Ouvrez le logiciel NetLogo.
   - Chargez le fichier de modèle du projet depuis l'application via le menu *File > Open*.
4. **Configuration de la simulation :**
   - Ajustez les curseurs (*sliders*) et boutons (*toggles*) de l'interface selon le scénario que vous souhaitez tester (par exemple : `nombre-pompiers`, `nombre-gardes`, `nombre-pyromanes`, `chance-declencher-feu`, etc.).
5. **Exécution :**
   - Cliquez sur le bouton **setup** pour initialiser l'environnement et placer les agents.
   - Cliquez sur le bouton **go** pour lancer le déroulement de la simulation et observez les graphiques, compteurs et comportements sur la grille.