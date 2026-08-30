# MAOA — Pick-and-Delivery Traveling Salesman Problem (PD-TSP)

Projet M2 AI2D 2025-2026 — UE Modèles et Algorithmes d'Ordonnancement et Applications (MAOA), Sorbonne Université. Auteurs : Mohamed NAJEM, Jules MAZLUM.

## Description / Sujet

Ce projet étudie une variante du problème du voyageur de commerce (TSP) où le véhicule ne se contente pas de visiter des villes, mais doit aussi gérer un inventaire dynamique :

- il transporte initialement un ensemble d'objets à **livrer** (ce qui allège la charge) ;
- il **ramasse** de nouveaux objets à profit dans les villes visitées (ce qui alourdit la charge) ;
- le coût de déplacement entre deux villes dépend du **poids transporté** au moment du trajet.

Deux variantes du coût de transport sont étudiées :

| Variante | Formule du coût |
|---|---|
| Linéaire | `C(Wi) = a·Wi` |
| Quadratique | `C(Wi) = a·Wi + b·Wi²` (pénalise fortement la surcharge) |

L'objectif est de trouver une tournée hamiltonienne et un plan de chargement `{zik, yik}` maximisant :

```
Z(Π, P) = Σ profits des objets ramassés − Σ coûts de déplacement (dépendants du poids)
```

**Objectifs du projet :**
- **Modélisation** : représenter les villes, les objets à livrer/ramasser, l'évolution du poids `Wi` et la fonction objectif `Z = profit − coût`.
- **Heuristiques gloutonnes** : construction rapide de solutions admissibles (nearest-neighbor pondéré, « profit d'abord », « livrer au plus tôt »).
- **Méthodes itératives** : amélioration locale via des opérateurs de voisinage — **Swap**, **2-Opt**, et **VND** (Variable Neighborhood Descent, combinant les deux précédents).
- **Résolution exacte (PLNE)** : formulation compacte de la variante linéaire, résolue avec **Gurobi**, servant de référence pour évaluer la qualité des heuristiques.
- **Comparaison expérimentale** : qualité de solution, temps de calcul, taille d'instance résolue, et sensibilité au paramètre `b` du modèle quadratique.

### Approches implémentées

**Heuristiques gloutonnes**
1. Gloutonne (nearest-neighbor pondéré) — sélectionne à chaque étape la ville qui maximise directement `Z = profit − coût`.
2. Profit d'abord — privilégie les villes au plus fort profit potentiel, sans tenir compte du coût de transport.
3. Livrer au plus tôt — priorise les villes où le poids de livraison est le plus important, pour alléger rapidement le véhicule.

**Méthodes itératives (recherche locale)**
- Swap : échange de deux villes dans la tournée.
- 2-Opt : inversion d'un segment de la tournée (élimine les croisements et raccourcit les trajets).
- VND : combine Swap et 2-Opt, en alternant les voisinages jusqu'à convergence.

**Modélisation PLNE (Gurobi)**
Formulation compacte de la variante linéaire avec variables de routage `xij`, variables de collecte `yik`, variables de flot continues `fij` (pour éviter la non-linéarité du produit poids × arc), contraintes d'élimination de sous-tours (MTZ) et contraintes de conservation du flot représentant l'évolution du poids `Wi`.

## Structure du dépôt

```text
.
├── MAZLUM_NAJEM.ipynb        Notebook contenant l'ensemble du code : modélisation, heuristiques, recherche locale
│                             (Swap/2-Opt/VND), formulation PLNE Gurobi, visualisations et expérimentations
├── MAZLUM_NAJEM.html         Export HTML du notebook (lecture rapide sans exécuter le code)
├── Rapport-MAZLUM_NAJEM.pdf  Rapport détaillé présentant la démarche, les résultats et leur interprétation
├── Projet2-PD-TSP.pdf        Énoncé original du projet (module MAOA)
└── TS2004t2/                 Instances de test utilisées pour les expérimentations
```

## Installation / Prérequis

```bash
pip install gurobipy
```

Une licence Gurobi est requise pour la partie PLNE.

## Utilisation

1. Ouvrir `MAZLUM_NAJEM.ipynb` dans Jupyter (ou consulter directement `MAZLUM_NAJEM.html` pour une lecture rapide).
2. Installer les dépendances nécessaires (notamment `gurobipy`).
3. Exécuter les cellules dans l'ordre pour reproduire les instances, les heuristiques, la recherche locale et les comparaisons expérimentales.

## Résultats principaux

- Les heuristiques simples se comportent très différemment selon la fonction de coût : « Profit d'abord » est souvent contre-productive dans la variante quadratique (elle ramasse des objets lourds sans se soucier du poids, ce qui fait exploser le coût), tandis que la stratégie Gloutonne offre le meilleur compromis distance/poids dès la construction.
- La recherche locale améliore significativement les tournées initiales. L'opérateur Swap s'avère plus déterminant que le 2-Opt dans la variante quadratique, car il agit directement sur l'ordre de collecte/livraison (donc sur l'évolution du poids `Wi`), tandis que le 2-Opt optimise surtout la géométrie du trajet.
- Gloutonne + VND est la méthode hybride la plus robuste : elle combine gestion de la charge (Swap) et optimisation géométrique (2-Opt), et surpasse en moyenne Gurobi sur des instances de taille moyenne (20 villes) lorsque celui-ci est contraint en temps de calcul.
- Sur les petites instances, la résolution exacte (Gurobi) révèle des arbitrages économiques fins (par exemple, renoncer à tout profit pour minimiser le coût de transport) que les heuristiques ne capturent pas naturellement.
- Le paramètre `b` (pénalisation quadratique) a un impact très différent selon la stratégie : la heuristique Gloutonne, qui intègre déjà `Z` dans sa construction, résiste beaucoup mieux à l'augmentation de `b` que « Profit d'abord » ou « Livrer tôt ».

Le détail complet des expérimentations, tableaux de résultats et visualisations est disponible dans le [rapport complet](./Rapport-MAZLUM_NAJEM.pdf).

## Références

- Veenstra et al., *Pickup-and-Delivery TSP with Handling Costs*, EJOR, 2017.
- Carrabs et al., *Variable Neighborhood Search for the PD-TSP with LIFO Loading*, INFORMS, 2007.
- Malli et al., *A computational study for the inventory routing problem*, [arXiv:2007.14740](https://arxiv.org/pdf/2007.14740), 2020.
- [Pickup-and-Delivery Site (H. Hernández-Pérez)](https://hhperez.webs.ull.es/PDsite/index.html)

## Auteurs

- Mohamed NAJEM
- Jules MAZLUM
