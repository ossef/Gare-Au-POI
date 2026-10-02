Engineering Project : Efrei
<br/>
Auteur : Youssef Ait El Mahjoub
<br/> 
Challenge : Open Data University — *Tourisme en Train*
<br/>
Sources des données : SNCF, DATAtourisme, ADEME et opérateurs de transport locaux
<br/>
<br/>

---
<div style="font-size: 1.5em; font-weight: bold;">🚆 Gare au POI !</div>
<i>Planification d'itinéraires touristiques multimodaux</i>
 
---
<br/>

## 🎯 Objectif du projet

Dans ce projet, vous allez développer progressivement une **application Web interactive** capable de proposer à un touriste un itinéraire complet en transports en commun, depuis une station de départ jusqu'à une destination touristique en France.

Ce projet s'inscrit dans le cadre du défi Open Data University **« Tourisme en Train »** :

https://defis.data.gouv.fr/defis/tourisme-en-train

L'application doit pouvoir être utilisée même lorsque la ville de départ ou la ville d'arrivée ne dispose pas d'un réseau urbain détaillé dans le projet.

L'utilisateur pourra donc saisir, selon le cas :

- une **ville de départ** ;
- une **station précise** de métro, RER, tramway, bus ou train lorsque le réseau urbain correspondant est disponible ;
- une **ville de destination** ;
- une **station précise** dans la ville d'arrivée ;
- un **site touristique** ;
- ou une destination sélectionnée directement sur la carte.

Il pourra également choisir un critère de recherche : **trajet le plus rapide**, **trajet avec le moins de correspondances** ou **trajet le moins émetteur en CO₂e 🌱**.

Le principe général est le suivant :

```text
Départ
(ville ou station précise)
        ↓
[réseau urbain si disponible]
        ↓
Gare ferroviaire
        ↓
Réseau ferroviaire national
        ↓
Gare d'arrivée
        ↓
[réseau urbain si disponible]
        ↓
Destination
(ville, station ou site touristique)
```

Ainsi, deux villes qui ne font pas partie des villes disposant d'un réseau urbain détaillé doivent tout de même pouvoir être reliées grâce au **réseau ferroviaire national**.

Exemple :

```text
Angoulême → La Rochelle
```

doit pouvoir produire un itinéraire ferroviaire même si aucun graphe urbain détaillé n'est intégré pour ces deux villes.

À l'inverse, lorsqu'un réseau urbain est disponible, l'application peut proposer un trajet plus précis, par exemple :

```text
Victoire — Bordeaux
        ↓
Tramway
        ↓
Bordeaux-Saint-Jean
        ↓
Train
        ↓
Paris
        ↓
Métro
        ↓
Musée du Louvre
```

Le projet est construit progressivement en **quatre phases**, la dernière constituant un niveau avancé consacré à la visite de plusieurs points d'intérêt.

---

## 📁 Données fournies

Le dossier `DATA/` contient des données GTFS téléchargées le **30/09/2026** afin de faciliter le démarrage du projet :

- `NATIONAL_SNCF_GTFS/` : données du réseau ferroviaire national ;
- `URBAN_GTFS/` : données des réseaux urbains fournies pour les 9 villes disponibles.

Ces données peuvent être utilisées comme jeu de référence. Pour votre application finale, il faudra utiliser les **données les plus récentes disponibles** depuis les sources officielles indiquées dans ce document.

### 🔎 Ressource complémentaire

Pour un exemple plus détaillé de manipulation des fichiers **GTFS** et de construction d'un graphe de transport, vous pouvez consulter mon précédent projet consacré au réseau d'Île-de-France :

https://github.com/ossef/Solution_Factory_IT

Ce dépôt peut notamment vous aider à comprendre l'extraction et l'exploitation des données GTFS.

---

# 🚆 Phase 1 — Réseau ferroviaire national

Cette première phase constitue le socle du projet. Vous allez construire vous-mêmes le graphe du réseau ferroviaire français à partir des données Open Data SNCF.

## 1.1 Données à utiliser

Utilisez le jeu de données officiel **SNCF — Horaires des lignes TGV, Intercités et TER** :

https://transport.data.gouv.fr/datasets/horaires-sncf

Le fichier GTFS est également accessible directement ici :

https://eu.ftp.opendatasoft.com/sncf/plandata/Export_OpenData_SNCF_GTFS_NewTripId.zip

Les données sont fournies au format **GTFS (General Transit Feed Specification)**.

Les principaux fichiers utiles sont :

| Fichier | Contenu utile pour le projet |
|---|---|
| `stops.txt` | Gares, arrêts et coordonnées géographiques |
| `routes.txt` | Lignes ou services |
| `trips.txt` | Circulations associées aux lignes |
| `stop_times.txt` | Succession des arrêts et horaires de chaque circulation |
| `calendar.txt` | Jours habituels de circulation |
| `calendar_dates.txt` | Exceptions au calendrier |

Vous devez construire le graphe à partir de ces données. Il est interdit d'utiliser un service externe fournissant directement l'itinéraire optimal.

> **Remarque :** dans le jeu de données SNCF fourni, `calendar.txt` n'est pas présent. Les dates de circulation des services sont décrites à l'aide de `calendar_dates.txt`. D'autres jeux de données GTFS, notamment urbains, peuvent toutefois contenir `calendar.txt`.

## 1.2 Graphe ferroviaire statique

Commencez par représenter le réseau ferroviaire par un graphe pondéré :

$$
G=(V,E, W)
$$

où :

- $V$ représente l'ensemble des gares ;
- $E$ représente les liaisons ferroviaires directes entre les gares ;
- $w(u,v)$ représente le coût associé à une liaison entre deux gares.

Les liaisons doivent être reconstruites à partir de `trips.txt` et `stop_times.txt`.

Deux arrêts consécutifs d'un même `trip_id` correspondent à une liaison directe.

Par exemple :

```text
Bordeaux-Saint-Jean
        ↓
Angoulême
        ↓
Poitiers
        ↓
Paris-Montparnasse
```

permet de créer les arêtes :

```text
Bordeaux-Saint-Jean → Angoulême
Angoulême → Poitiers
Poitiers → Paris-Montparnasse
```

Dans cette première version, le poids d'une arête pourra représenter une durée de parcours représentative entre les deux gares.

Si plusieurs trains parcourent la même liaison avec des durées différentes, vous devez définir et justifier la stratégie utilisée : minimum, moyenne ou autre choix pertinent.

### Travail demandé

Votre application doit au minimum permettre de :

1. construire automatiquement le graphe à partir du GTFS ;
2. afficher le nombre de sommets et d'arêtes ;
3. afficher les voisins et le degré d'une gare ;
4. étudier la connexité du graphe à l'aide de BFS ou DFS ;
5. déterminer si une destination est accessible depuis une gare donnée ;
6. implémenter l'algorithme de **Dijkstra** ;
7. calculer et afficher un meilleur chemin entre deux gares.

Exemple :

```text
Départ      : Bordeaux-Saint-Jean
Destination : Strasbourg

Bordeaux-Saint-Jean
        ↓
...
        ↓
Strasbourg

Durée estimée : ...
```

Les algorithmes demandés doivent être implémentés par votre groupe. Une bibliothèque de graphes ne doit pas calculer le plus court chemin à votre place.

## 1.3 Réseau ferroviaire temporel

Dans la réalité, le meilleur itinéraire dépend de l'heure à laquelle vous voyagez.

Un chemin composé de liaisons rapides peut être mauvais si une correspondance impose une attente importante.

L'utilisateur fournit maintenant :

```text
Gare de départ
Gare de destination
Date
Heure minimale de départ
```

Exemple :

```text
Départ      : Bordeaux-Saint-Jean
Destination : Strasbourg
Date        : 15/11/2026
Heure       : 08:00
```

Vous devez exploiter :

```text
trips.txt
stop_times.txt
calendar_dates.txt
```

afin de déterminer :

- les trains circulant réellement à la date choisie ;
- les heures de départ ;
- les heures d'arrivée ;
- les correspondances réalisables ;
- les temps d'attente.

L'objectif devient de rechercher **l'itinéraire permettant d'arriver le plus tôt possible**, compte tenu de l'heure de départ.

Vous passez donc d'un problème de plus court chemin statique à un problème de type **Earliest Arrival**.

Exemple de résultat :

```text
08:12  Bordeaux-Saint-Jean
          |
          | Train
          ↓
10:20  Paris

        Correspondance : 35 min

10:55  Paris
          |
          | Train
          ↓
12:45  Strasbourg

Durée totale    : 4 h 33
Correspondances : 1
```

Vous devez expliquer la représentation choisie pour intégrer la dimension temporelle et la différence avec le graphe statique précédent.

### ⏰ Extension — Heure d'arrivée imposée

En complément d'une heure minimale de départ, l'utilisateur peut choisir de spécifier une **heure maximale d'arrivée**.

Exemple :

```text
Bordeaux-Saint-Jean → Strasbourg
Date : 15/11/2026
Arrivée souhaitée : avant 18:00
```

L'application doit alors rechercher un itinéraire permettant de respecter cette contrainte horaire et déterminer un départ adapté.

Cette fonctionnalité constitue une **extension** de la recherche d'itinéraire temporelle. La méthode algorithmique utilisée est laissée à votre choix et devra être expliquée et justifiée.

---

# 🏛️ Phase 2 — Tourisme et critères de voyage

Dans cette deuxième phase, l'utilisateur n'est plus obligé de connaître la gare correspondant à sa destination.

Il peut rechercher directement un lieu touristique :

```text
Musée du Louvre
Château de Versailles
Cité de Carcassonne
Mont-Saint-Michel
...
```

Il pourra également sélectionner une destination directement sur une carte.

## 2.1 Données touristiques

Utilisez en priorité **DATAtourisme**, la plateforme nationale Open Data des données touristiques :

https://www.data.gouv.fr/datasets/datatourisme-la-plateforme-nationale-des-donnees-touristiques

DATAtourisme fournit des points d'intérêt (POI) touristiques géolocalisés, notamment :

- patrimoine culturel ;
- patrimoine naturel ;
- musées ;
- monuments ;
- sites touristiques ;
- activités ;
- événements.

Selon l'export choisi, vous pourrez notamment exploiter :

```text
nom
type
latitude
longitude
adresse
commune
description
```

Des exports peuvent être récupérés par région et par catégorie.

## 2.2 Associer un site touristique au réseau

Un point d'intérêt touristique possède des coordonnées géographiques.

Les gares et stations GTFS disposent également de coordonnées : `stop_lat` et `stop_lon`.

Vous devez utiliser ces informations pour identifier les gares ou stations susceptibles de desservir la destination.

### Distance de Haversine

Pour rechercher les gares ou stations **géographiquement proches** d'un point d'intérêt, vous pouvez utiliser la distance de Haversine.

Elle mesure la distance orthodromique entre deux coordonnées géographiques en assimilant la Terre à une sphère :

$$
d =
2R
\arcsin
\left(
\sqrt{
\sin^2\left(\frac{\Delta\varphi}{2}\right)
+
\cos(\varphi_1)\cos(\varphi_2)
\sin^2\left(\frac{\Delta\lambda}{2}\right)
}
\right)
$$

avec :

- $d$ : distance géographique ;
- $R \approx 6\,371$ km : rayon terrestre ;
- $\varphi_1,\varphi_2$ : latitudes en radians ;
- $\lambda_1,\lambda_2$ : longitudes en radians.

**Avec $R = 6\,371$ km, la distance $d$ obtenue par la formule est directement exprimée en kilomètres.**

**Référence :** R. W. Sinnott, *Virtues of the Haversine*, *Sky & Telescope*, vol. 68, no. 2, 1984.

Présentation de la méthode : https://search.r-project.org/CRAN/refmans/geosphere/html/distHaversine.html

### Exemple

Supposons que vous obteniez les distances suivantes entre un POI et trois stations :

| Station | Distance géographique |
|---|---:|
| Station A | 0,35 km |
| Station B | 0,80 km |
| Station C | 1,40 km |

Si vous décidez de retenir toutes les stations situées à moins de 1 km, les stations A et B deviennent des **stations candidates**.

**Attention :** une distance Haversine de 350 m ne signifie pas nécessairement qu'il faut marcher 350 m. Les rues, bâtiments, cours d'eau et autres obstacles ne sont pas pris en compte.

Haversine est utilisée ici pour répondre à la question :

> « Quelles stations sont géographiquement proches de ma destination ? »

Vous devez justifier votre méthode de sélection : station la plus proche, stations dans un rayon donné, les $k$ stations les plus proches, etc.


## 2.3 Estimation des distances parcourues

Les fichiers GTFS ne fournissent pas toujours directement la distance parcourue entre deux arrêts. Cette distance doit donc être **estimée à partir des données géographiques disponibles**.

Deux situations peuvent se présenter.

#### Cas 1 — Le fichier `shapes.txt` est disponible

Certains jeux de données GTFS, notamment urbains, contiennent un fichier `shapes.txt` décrivant la géométrie des lignes de transport.

Un `trip` peut être associé à un `shape_id` dans `trips.txt`. Le fichier `shapes.txt` fournit alors une succession ordonnée de points géographiques :

```text
shape_id
shape_pt_lat
shape_pt_lon
shape_pt_sequence
```

Lorsque ces informations sont disponibles, elles doivent être utilisées afin d'obtenir une estimation plus précise de la distance parcourue.

Pour deux points successifs $p_i$ et $p_{i+1}$ du tracé, la distance peut être estimée avec la formule de Haversine :

$$
d_i=d_H(p_i,p_{i+1})
$$

La longueur estimée du tracé est alors :

$$
d(P)=\sum_{i=1}^{n-1} d_H(p_i,p_{i+1})
$$

Cette méthode permet de prendre en compte la géométrie du tracé de manière plus précise qu'une simple distance entre les arrêts.

#### Cas 2 — Le fichier `shapes.txt` n'est pas disponible

Lorsque `shapes.txt` est absent, comme dans le jeu de données SNCF fourni, la distance doit être estimée à partir des coordonnées des **arrêts successifs du `trip`**.

Pour un trajet :

$$
s_1 \rightarrow s_2 \rightarrow \dots \rightarrow s_n
$$

on calcule :

$$
d(P)=\sum_{i=1}^{n-1} d_H(s_i,s_{i+1})
$$

où les coordonnées des arrêts sont obtenues à partir de `stops.txt`.

### Exemple

Considérons un trajet passant successivement par quatre arrêts :

Arrêt A → 42 km → Arrêt B → 71 km → Arrêt C → 36 km → Arrêt D

La distance estimée du trajet est :

$$
d(P)=42+71+36=149\ \mathrm{km}
$$

**Attention :** les méthodes précédentes fournissent une **estimation de la distance réellement parcourue sur le réseau**. La précision obtenue dépend des informations géographiques disponibles dans le jeu GTFS.

> **Remarque :** en l'absence de `shapes.txt`, la précision de l'estimation dépend notamment du nombre d'arrêts présents dans le `trip` considéré. Un trajet comportant peu d'arrêts intermédiaires peut conduire à une estimation moins proche de la distance réellement parcourue.

Il faut également distinguer les deux utilisations de Haversine dans le projet :

- dans la section 2.2, elle permet d'identifier les stations géographiquement proches d'un point d'intérêt ;
- dans cette section, elle permet d'estimer les distances parcourues à partir des données géographiques disponibles dans le GTFS.

Les distances ainsi estimées pourront notamment être utilisées pour calculer les émissions de $\mathrm{CO_2e}$ associées à un itinéraire.

---

## 🌱 2.4 Estimation des émissions de CO₂e

Pour chaque itinéraire, vous devez fournir une estimation de son impact carbone.

Cette estimation dépend :

- de la **distance estimée** de chaque segment, calculée selon la méthode décrite en section 2.3 ;
- du **mode de transport utilisé** sur chaque segment : TGV, TER, métro, tramway, bus, etc. ;
- du **facteur d'émission** associé à ce mode de transport.

Les facteurs d'émission des transports peuvent notamment être exprimés en :

$$
\mathrm{gCO_2e/(voyageur\cdot km)}
$$

Un **voyageur-kilomètre** représente le transport d'un voyageur sur un kilomètre.

Par exemple :

- 1 voyageur parcourant 100 km représente 100 voyageur-kilomètres ;
- 10 voyageurs parcourant chacun 100 km représentent 1 000 voyageur-kilomètres.

Dans votre application, l'empreinte affichée correspond à celle d'**un voyageur** effectuant l'itinéraire proposé.

Pour un segment $e$ :

$$
E(e)=d(e)\times FE(m_e)
$$

avec :

- $d(e)$ : distance estimée du segment en **km**, calculée à partir des données GTFS selon la méthode décrite en section 2.3 ;
- $m_e$ : mode de transport utilisé sur le segment ;
- $FE(m_e)$ : facteur d'émission du mode de transport en $\mathrm{gCO_2e/(voyageur\cdot km)}$ ;
- $E(e)$ : émission estimée du segment en $\mathrm{gCO_2e/voyageur}$.
- 
L'analyse des unités donne :

$$\mathrm{km} \times \frac{\mathrm{gCO_2e}}{\mathrm{voyageur}\cdot\mathrm{km}} = \frac{\mathrm{gCO_2e}}{\mathrm{voyageur}}$$

Pour un itinéraire multimodal $P$ composé de plusieurs segments :

$$E(P) = \sum_{e\in P} d(e)\times FE(m_e)$$

Le calcul doit donc être effectué **segment par segment**, afin de prendre en compte les différents modes de transport pouvant composer un même itinéraire. Le facteur d'émission peut ainsi être différent d'un segment à l'autre.

### Exemple numérique

Considérons, uniquement à titre d'exemple, l'itinéraire fictif suivant :

| Segment | Distance estimée | Facteur utilisé |
|---|---:|---:|
| Tramway | 5 km | 4 gCO₂e/(voyageur·km) |
| Train | 500 km | 3 gCO₂e/(voyageur·km) |
| Métro | 8 km | 5 gCO₂e/(voyageur·km) |

Pour le tramway : $E_{\mathrm{tram}} = 5 \times 4 = 20\ \mathrm{gCO_2e/voyageur}$

Pour le train : $E_{\mathrm{train}} = 500 \times 3 = 1500\ \mathrm{gCO_2e/voyageur}$

Pour le métro : $E_{\mathrm{metro}} = 8 \times 5 = 40\ \mathrm{gCO_2e/voyageur}$

L'émission totale estimée est donc :

$$E_{\mathrm{total}} = 20+1500+40 = 1560\ \mathrm{gCO_2e/voyageur} =  1{,}56\ \mathrm{kgCO_2e/voyageur}$$

**Les distances et les facteurs d'émission utilisés dans cet exemple sont uniquement illustratifs.** Ils ne doivent pas être utilisés comme valeurs de référence.

### Facteurs d'émission

Pour votre application, vous devez utiliser des facteurs correspondant aux **modes de transport réellement utilisés**.

Les facteurs peuvent notamment être obtenus à partir de **Impact CO₂**, service qui s'appuie sur les données environnementales de l'ADEME :

https://impactco2.fr/outils/transport

Une API est également disponible pour récupérer les **facteurs d'émission associés aux différents modes de transport** :

https://impactco2.fr/outils/api

Des informations complémentaires concernant les facteurs d'émission des transports ferroviaires sont disponibles auprès de **SNCF Voyageurs** :

https://www.sncf-voyageurs.com/fr/decouvrez-notre-entreprise/rse-et-transitions/le-calcul-de-lempreinte-carbone-des-transports/

> **Important :** l'utilisation d'une API ou d'un service externe pour calculer directement la distance ou les émissions de CO₂e d'un itinéraire n'est pas autorisée. Les distances doivent être estimées à partir des données GTFS selon la méthode décrite en section 2.3, puis les émissions doivent être calculées par votre application à partir de ces distances et des facteurs d'émission correspondants.

## 2.5 Critères de recherche

L'utilisateur doit pouvoir choisir au minimum l'un des trois critères suivants.

### Trajet le plus rapide

$$
\min_{P} T(P)
$$

### Trajet avec le moins de correspondances

$$
\min_{P} N_{\mathrm{corr}}(P)
$$

### 🌱 Trajet le moins émetteur en CO₂e

$$
\min_{P} E(P)
$$

Pour une même origine et une même destination, les solutions peuvent être différentes.

Votre application doit permettre de les comparer :

| Proposition | Durée | Correspondances | CO₂e |
|---|---:|---:|---:|
| Plus rapide | ... | ... | ... |
| Moins de correspondances | ... | ... | ... |
| 🌱 Moins émetteur | ... | ... | ... |

---

# 🚇 Phase 3 — Itinéraire Multimodal de bout en bout

Dans cette troisième phase, vous devez intégrer les réseaux de transport urbain.

Le point de départ n'est donc plus obligatoirement une gare SNCF.

L'utilisateur peut sélectionner une station de métro, tramway, bus, RER ou train dans sa ville.

Par exemple :

```text
Départ      : Victoire — Bordeaux
Destination : Musée du Louvre — Paris
```

Votre application doit être capable de construire un itinéraire complet :

```text
Victoire
    ↓
Tramway
    ↓
Bordeaux-Saint-Jean
    ↓
Train
    ↓
Paris
    ↓
Métro / RER / Bus
    ↓
Station proche du Louvre
    ↓
Musée du Louvre
```

## 3.1 Périmètre des réseaux urbains détaillés

Le réseau ferroviaire développé en Phase 1 reste **national**. L'application ne doit donc pas être limitée aux villes ci-dessous.


Les villes suivantes correspondent au **périmètre cible des réseaux urbains détaillés** du projet. Les données urbaines (métro, RER, tramway, bus ou autres) de 9 de ces villes sont fournies dans le dossier `DATA/URBAN_GTFS/`. Le cas particulier de Lyon est précisé dans la section 3.2.

1. Paris / Île-de-France ;
2. Bordeaux ;
3. Lyon ;
4. Marseille ;
5. Nice ;
6. Toulouse ;
7. Strasbourg ;
8. Nantes ;
9. Montpellier ;
10. Avignon.

Cette liste constitue le périmètre multimodal détaillé du projet et ne représente pas un classement statistique officiel de la fréquentation touristique.

Deux niveaux de service doivent donc être distingués :

- **partout en France** : recherche d'un itinéraire ferroviaire entre villes ou gares à partir du réseau SNCF ;
- **dans les villes couvertes par les données urbaines** : possibilité d'étendre l'itinéraire jusqu'à une station précise ou un point d'intérêt (POI) en utilisant le réseau urbain.

Par exemple :

```text
Ville A → Train → Ville B
```

reste possible même si A et B ne font pas partie des dix villes ci-dessus.

En revanche :

```text
Station urbaine → Gare → Train → Gare → Station urbaine → POI
```

nécessite que les données urbaines correspondantes soient disponibles.

## 3.2 Données des réseaux urbains

Les liens ci-dessous pointent vers les jeux de données identifiés pour le projet.

| Ville | Réseau | Principaux modes | Jeu de données |
|---|---|---|---|
| Paris / Île-de-France | Île-de-France Mobilités | Métro, RER, train, tramway, bus | https://transport.data.gouv.fr/datasets/reseau-urbain-et-interurbain-dile-de-france-mobilites |
| Bordeaux | TBM | Tramway, bus, ferry | https://transport.data.gouv.fr/datasets/offres-de-services-bus-tram-et-scolaire-au-format-gtfs-netex-gtfs-rt-siri-lite |
| Lyon | TCL / SYTRAL Mobilités | Métro, tramway, bus, funiculaire | https://transport.data.gouv.fr/datasets/horaires-theoriques-du-reseau-transports-en-commun-lyonnais |
| Marseille | RTM / Métropole Aix-Marseille-Provence | Métro, tramway, bus | https://transport.data.gouv.fr/datasets/reseaux-de-transports-en-commun-de-la-metropole-daix-marseille-provence-et-des-bouches-du-rhone |
| Nice | Lignes d'Azur | Tramway, bus | https://transport.data.gouv.fr/datasets/donnees-statiques-et-dynamiques-du-reseau-de-transport-lignes-dazur |
| Toulouse | Tisséo | Métro, tramway, bus, téléphérique | https://transport.data.gouv.fr/datasets/tisseo-reseau-transport-urbain-toulousain |
| Strasbourg | CTS | Tramway, bus | https://transport.data.gouv.fr/datasets/donnees-theoriques-gtfs-et-temps-reel-siri-lite-du-reseau-cts |
| Nantes | Naolib | Tramway, bus et autres transports du réseau | https://transport.data.gouv.fr/datasets/reseau-de-transports-collectifs-naolib |
| Montpellier | TaM | Tramway, bus | https://transport.data.gouv.fr/datasets/offre-de-transport-tam-en-temps-reel-gtfs-rt-urbain-et-suburbain |
| Avignon | Orizo | Tramway, bus | https://transport.data.gouv.fr/datasets/reseau-orizo-grand-avignon |

### Cas particulier de Lyon

Le jeu TCL est référencé sur le Point d'Accès National, mais l'accès aux données courantes du producteur peut nécessiter une authentification sur la plateforme Grand Lyon :

https://data.grandlyon.com/portail/fr/connexion

Vérifiez toujours la date et la période de validité du GTFS utilisé.

## 3.3 Construire les graphes urbains

Les réseaux urbains utilisent la même structure GTFS générale que celle étudiée en Phase 1 :

```text
stops.txt
routes.txt
trips.txt
stop_times.txt
calendar.txt
calendar_dates.txt
```

Selon les réseaux, vous pourrez également trouver :

```text
transfers.txt
shapes.txt
```

Vous devez construire les graphes urbains à partir de ces fichiers.

Vous ne devez pas utiliser le calculateur d'itinéraires de l'opérateur pour résoudre le problème à votre place.

## 3.4 Fusionner les réseaux

Vous disposez maintenant de plusieurs graphes :

- le réseau urbain de départ ;
- le réseau ferroviaire national ;
- le réseau urbain d'arrivée.

Le réseau multimodal peut être représenté par :

$$
G =
G_{\mathrm{urbain,dep}}
\cup
G_{\mathrm{SNCF}}
\cup
G_{\mathrm{urbain,arr}}
\cup
E_{\mathrm{transferts}}
$$

où $E_{\mathrm{transferts}}$ contient les arêtes permettant de passer d'un réseau à un autre.

Vous devez notamment résoudre le problème suivant :

> Comment reconnaître qu'une gare SNCF et une station d'un réseau urbain permettent une correspondance ?

Les identifiants utilisés dans les différents GTFS ne sont généralement pas identiques.

Vous pourrez notamment exploiter :

- le nom de la gare ou de la station ;
- les coordonnées géographiques ;
- la distance géographique entre les arrêts ;
- `parent_station` ;
- `transfers.txt` lorsqu'il est disponible.

Votre méthode d'appariement doit être expliquée et justifiée.

## 3.5 Relier le réseau à la destination finale

Le POI touristique n'est généralement pas un sommet du réseau de transport.

Vous devez donc construire la dernière étape :

```text
Station d'arrivée
        ↓
Déplacement terminal
        ↓
POI touristique
```

La méthode de proximité développée en Phase 2 peut être réutilisée pour sélectionner plusieurs stations candidates autour du POI.

Ne choisissez pas nécessairement la station géographiquement la plus proche : une station légèrement plus éloignée peut conduire à un meilleur itinéraire global.

## 3.6 Optimisation du trajet complet

Les critères introduits précédemment doivent maintenant être appliqués à l'ensemble du trajet.

La durée totale peut par exemple être décomposée en :

$$T(P) = T_{\mathrm{urbain,dep}} + T_{\mathrm{train}} + T_{\mathrm{urbain,arr}} + T_{\mathrm{attente}} + T_{\mathrm{marche}}$$

Les émissions totales sont obtenues en additionnant les émissions des différents segments :

$$ E(P) = \sum_{e\in P} d(e)\times FE_{\mathrm{mode}(e)} $$

Le nombre de correspondances doit également être calculé sur l'ensemble du parcours.

Votre application doit pouvoir proposer :

- le trajet **le plus rapide** ;
- le trajet avec **le moins de correspondances** ;
- le trajet **le moins émetteur en CO₂e 🌱**.

---

# 🧭 Phase 4 — Visite de plusieurs points d'intérêt

Dans cette phase avancée, l'utilisateur ne souhaite plus nécessairement rejoindre une seule destination. Il peut vouloir **visiter plusieurs monuments ou points d'intérêt au cours d'un même parcours**.

L'utilisateur fournit :

- un point de départ ;
- un point d'arrivée ;
- une liste de points d'intérêt à visiter ;
- un critère d'optimisation.

Le point de départ et le point d'arrivée peuvent être identiques.

Par exemple :

```text
Départ  : Gare de Lyon
Arrivée : Gare de Lyon

À visiter :
- Musée du Louvre
- Tour Eiffel
- Arc de Triomphe
- Sacré-Cœur
```

Un autre utilisateur pourrait au contraire demander :

```text
Départ  : Gare du Nord
Arrivée : Gare Montparnasse

À visiter :
- Musée du Louvre
- Panthéon
- Tour Eiffel
```

## 4.1 Problème à résoudre

L'ordre dans lequel les points d'intérêt sont saisis par l'utilisateur **ne doit pas nécessairement être l'ordre de visite**.

Votre application doit proposer un ordre pertinent et construire l'itinéraire correspondant.

Si l'ensemble des lieux à visiter est :

$$
M=\{m_1,m_2,\ldots,m_k\}
$$

vous devez rechercher un parcours :

```text
Départ
   ↓
POI ?
   ↓
POI ?
   ↓
...
   ↓
POI ?
   ↓
Arrivée
```

passant par **tous les points d'intérêt demandés**.

Le problème comporte ainsi deux dimensions :

1. déterminer un ordre de visite intéressant ;
2. calculer les trajets permettant de relier les différentes étapes.

Le coût d'un parcours peut être défini selon le critère choisi par l'utilisateur : durée totale, nombre de correspondances, émissions de CO₂e ou autre critère justifié.

## 4.2 Cas où le départ et l'arrivée sont identiques

Vous devez accepter le cas :

$$
s=t
$$

Un utilisateur peut par exemple partir d'une gare, d'une station ou d'un autre point du réseau, visiter plusieurs monuments puis revenir à son point de départ.

Exemple :

```text
Point de départ
      ↓
Monument A
      ↓
Monument B
      ↓
Monument C
      ↓
Point de départ
```

## 4.3 Quelques pistes de réflexion

Cette partie est volontairement plus ouverte.

Avant de choisir une méthode, posez-vous notamment les questions suivantes :

- combien d'ordres de visite sont possibles pour $k$ monuments ?
- est-il raisonnable de tous les tester lorsque $k$ augmente ?
- choisir à chaque étape le monument actuellement le plus proche donne-t-il toujours la meilleure solution globale ?
- peut-on réutiliser les algorithmes développés dans les phases précédentes pour calculer le coût entre deux étapes ?
- faut-il utiliser une méthode exacte dans tous les cas ?
- une heuristique peut-elle être pertinente lorsque le nombre de lieux à visiter devient important ?
- comment comparer la qualité et le temps de calcul de plusieurs approches ?

Plusieurs familles d'approches sont envisageables : exploration exhaustive, programmation dynamique, méthodes gloutonnes, heuristiques ou autres stratégies que vous jugerez pertinentes.

**Aucune méthode n'est imposée dans l'énoncé.** Vous devez choisir, implémenter, expliquer et évaluer votre approche.

Votre solution devra notamment discuter :

- de la qualité des itinéraires obtenus ;
- de la complexité de la méthode ;
- de son temps de calcul lorsque le nombre de monuments augmente ;
- des compromis éventuels entre optimalité et rapidité.


---

# ⚙️ Contraintes algorithmiques

Le cœur algorithmique du projet doit être développé par votre groupe.

Il est interdit d'utiliser une bibliothèque, une API ou un service externe calculant directement :

- le plus court chemin ;
- l'itinéraire ferroviaire ;
- les correspondances ;
- l'itinéraire multimodal complet.

Les bibliothèques restent autorisées pour :

- lire et traiter les fichiers CSV/GTFS ;
- manipuler les dates et heures ;
- effectuer des calculs géographiques élémentaires ;
- développer l'interface graphique ;
- afficher les données et les itinéraires sur une carte.

Une bibliothèque de graphes peut éventuellement être utilisée pour **stocker ou visualiser** le graphe, mais les algorithmes demandés dans le cadre du projet doivent être vos propres implémentations.

---

# 💻 Contraintes techniques

Le projet doit prendre la forme d'une **application Web interactive**.

L'application doit comporter au minimum :

- une interface Web permettant de saisir ou sélectionner une **ville**, une **gare/station** ou un **point d'intérêt**, selon les données disponibles ;
- une **carte interactive** permettant de visualiser les gares, stations, points d'intérêt touristiques et itinéraires proposés ;
- la possibilité, dans la phase avancée, de sélectionner **plusieurs points d'intérêt à visiter** ;
- la possibilité de sélectionner le **critère d'optimisation** ;
- l'affichage détaillé de l'itinéraire : modes de transport, gares/stations, correspondances, horaires, durée et émissions estimées.

## Architecture de l'application

Le projet doit être organisé autour de deux composants distincts :

```text
projet/
│
├── frontend/
│   └── Interface Web et carte interactive
│
└── backend/
    └── Données, construction des graphes,
        algorithmes et calcul des itinéraires
```

Le **frontend** est responsable de l'interaction avec l'utilisateur et de la visualisation des résultats.

Le **backend** est notamment responsable :

- du chargement et du traitement des données Open Data ;
- de la construction des graphes ;
- de la gestion des horaires et des correspondances ;
- de l'association entre réseaux et points d'intérêt ;
- de l'exécution des algorithmes de recherche d'itinéraires ;
- du calcul des différents critères.

Le frontend communique avec le backend à travers une **API Web**.

Les technologies utilisées pour le frontend, le backend et la cartographie sont laissées au choix du groupe.

Les bibliothèques nécessaires à l'affichage d'une carte interactive sont autorisées. La carte constitue cependant un **outil d'interaction et de visualisation** : elle ne doit pas calculer l'itinéraire à la place du backend.

## 🐳 Conteneurisation — recommandée

La conteneurisation de l'application avec **Docker est souhaitable mais non obligatoire**.

Dans ce cas, le frontend et le backend pourront être exécutés dans des conteneurs distincts :

~~~text
projet/
│
├── frontend/
│   ├── Dockerfile
│   └── ...
│
├── backend/
│   ├── Dockerfile
│   └── ...
│
└── docker-compose.yml
~~~

L'utilisation de **Docker Compose** est recommandée afin de permettre le lancement de l'ensemble de l'application avec une commande unique :

~~~bash
docker compose up
~~~

Des services supplémentaires, par exemple une base de données, peuvent être ajoutés si l'architecture retenue le nécessite.

Une architecture comportant plusieurs services est également possible, mais **aucun découpage en microservices n'est imposé**. Tout choix d'architecture supplémentaire devra être cohérent avec les besoins du projet et justifié.

---

# 📋 Récapitulatif des phases

| Phase | Objectif principal | Données utilisées | Attendus |
|---|---|---|---|
| **1A — Ferroviaire statique** | Construire le graphe ferroviaire français | GTFS SNCF : `stops.txt`, `trips.txt`, `stop_times.txt` | Construction du graphe, voisins, degrés, BFS/DFS, connexité, Dijkstra et itinéraire entre deux gares |
| **1B — Ferroviaire temporel** | Intégrer les horaires et les correspondances | `trips.txt`, `stop_times.txt`, `calendar_dates.txt` | Services disponibles, horaires, attentes, correspondances et recherche de l'arrivée au plus tôt |
| **2A — Tourisme** | Permettre de choisir un POI plutôt qu'une gare | DATAtourisme + coordonnées GTFS | Recherche et géolocalisation d'un POI, sélection des gares/stations candidates et association au réseau |
| **2B — Critères 🌱** | Comparer plusieurs itinéraires | Horaires, distances et facteurs d'émission | Recherche du trajet le plus rapide, avec le moins de correspondances et le moins émetteur en CO₂e |
| **3A — Réseaux urbains** | Enrichir le réseau national dans les 10 villes couvertes | GTFS des réseaux urbains | Construction des graphes métro, tramway, bus, RER, etc. et création des correspondances avec le réseau SNCF |
| **3B — Multimodal** | Calculer un trajet de bout en bout lorsque les données locales sont disponibles | SNCF + réseaux urbains + DATAtourisme + facteurs d'émission | Itinéraire ville/station → train → ville/station/POI et comparaison selon temps, correspondances et CO₂e 🌱 |
| **4 — Multi-POI 🧭** | Organiser la visite de plusieurs lieux | Graphe multimodal + liste de POI | Choix de l'ordre de visite, itinéraire passant par tous les POI, comparaison de méthodes et étude de la complexité |

---

# 🔗 Sources principales

## 🚆 Réseau ferroviaire

➔ SNCF Horaires TGV, Intercités et TER  : https://transport.data.gouv.fr/datasets/horaires-sncf

## 🏛️ Données touristiques

➔ DATAtourisme  : https://www.data.gouv.fr/datasets/datatourisme-la-plateforme-nationale-des-donnees-touristiques


## 🌱 Facteurs d'émission

➔ Impact CO₂ — Comparateur des modes de transport : https://impactco2.fr/outils/transport

➔ Base Empreinte — ADEME  : https://base-empreinte.ademe.fr/

➔  SNCF Voyageurs — Méthode de calcul de l'empreinte carbone  :https://www.sncf-voyageurs.com/fr/decouvrez-notre-entreprise/rse-et-transitions/le-calcul-de-lempreinte-carbone-des-transports/

## 🚇 Réseaux urbains

➔ Les liens vers les jeux de données des dix réseaux retenus sont regroupés dans la section **3.2 — Données des réseaux urbains**.

## 🕸️ Graphes et algorithmes

➔ Cours Moodle "Algorithmes et théorie des graphes" ou équivalent.
