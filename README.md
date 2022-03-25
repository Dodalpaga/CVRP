# CVRP

## Introduction aux VRP

Les problèmes de tournées de véhicules ou **Vehicle Routing Problem** sont des problèmes d’optimisation combinatoire complexes et trouvant de nombreuses applications dans le secteur industriel, plus particulièrement en logistique. Les VRP sont une combinaison de contraintes d’allocation de ressources (répartition de la charge dans les véhicules) et de construction de séquence. La difficulté étant que dans l’industrie ces problèmes prennent rapidement une dimension gigantesque d’où l’intérêt d’avoir des heuristiques pour les résoudre.

### Contexte des problèmes CVRP

L'objectif de ce projet est d'implémenter des méthodes approchées permettant de résoudre un problème de planification de tournées de véhicules.

Il s'agit des problèmes de type CVRP : **_Capacited Vehicle Routing Problem_**. Notre entreprise de livraison dispose d'une flotte de camions avec une capacité limitée. Nous avons un ensemble de clients à desservir en partant d'un point de départ : le dépôt de notre entreprise. Nous connaissons la localisation de chacun des clients dans la section NODE_COORD_SECTION du fichier vrp qui sera présenté dans la partie b). Chaque demande d'un client représente une quantité qu'il faut livrer du dépôt à la localisation du client. Le but est donc de planifier les tournées des véhicules afin de minimiser la somme des distances parcourues (temps de transport).<br/>
**Fonction objectif** : “minimiser la somme des trajets réalisés”<br/>
Il est nécessaire de respecter :

- la contrainte de capacité des camions
- la contrainte qui impose de satisfaire tous les clients

Il faut donc trouver les chemins les plus courts pour organiser la tournée en respectant les contraintes de capacités.<br/>

### Comprendre le format des jeux de données

Une instance est un problème défini par un ensemble de clients et leur demande associée à satisfaire. De nombreux types d’instances ont été créés au cours des années et sont disponibles sur le site de "CVRP LIB - All instances” :
Lien : <a href="http://vrp.galgos.inf.puc-rio.br/index.php/en/">CVRPLIB<a/>

Pour chaque instance, nous pouvons avoir accès à une visualisation graphique modélisant le problème “Plotted Instance”, au fichier vrp et à un fichier solution.

<center>

<img src="img/1.png">
</center>
En sélectionnant une <a href="http://vrp.galgos.inf.puc-rio.br/index.php/en/plotted-instances?data=X-n120-k6">Instance<a/>, plusieurs informations sont directement affichées :

<center>

<img src="img/2.png">
<img src="img/3.png">
</center>

Notamment, on peut afficher la meilleure solution trouvée à l'heure actuelle, potentiellement la solution optimale dans le cas où la colonne _Opt_ est inscrite "yes"

<center>

<img src="img/4.png">
</center>

### Première heuristique : Route First

```python
# Initialisation
L1 = [ client1, client2, ..., clientn ]
routes = [ ]

# Définition d’une fonction pour la création primaire des routes
def create_routes(L1, truckcapacity):
    while L1 not empty:
        # Initialisation d’une nouvelle route
        route = closest_client(warehouse,L1)
        while demands(route) < truck_capacity :
            nextclient = closest_client(route[-1] , L1)
            route.append(nextclient)
            L1.remove(nextclient)
        routes.append(route)
```

La fonction “closest_client” prend en compte deux arguments : l’identifiant d’un client C1 (ou du dépôt) et une liste d’identifiants de client Cx. Elle retourne le client Cp dans Cx qui a la distance euclidienne avec C1 la plus faible.

Nous avons ensuite créé des méthodes nous permettant d’évaluer notre heuristique, à savoir :<br/>

- get_route_cost(route) : permet de calculer la distance parcourue par une route (fonction coût)
- get_route_capacity(route) : permet de calculer la somme des demandes des clients sur une route

En prenant pour initialisation les identifiants des clients dans leur ordre chronologique (i.e. [ 0, 1, 2, …, n ]) voici ce que l’on peut obtenir comme résultats sur l'instance X-n120-k6 :<br/>

- Cost | Capacity for route 0 : 1705 | 21
- Cost | Capacity for route 1 : 2212 | 21
- Cost | Capacity for route 2 : 2985 | 21
- Cost | Capacity for route 3 : 2811 | 21
- Cost | Capacity for route 4 : 3430 | 21
- Cost | Capacity for route 5 : 2705 | 14<br/>

Total cost = 15848<br/>

Routes : [[19, 115, 106, 60, 118, 93, 9, 87, 51, 49, 37, 30, 72, 24, 28, 79, 15, 83, 92, 54, 69], [95, 70, 53, 111, 44, 2, 77, 38, 41, 48, 1, 42, 75, 112, 63, 108, 81, 97, 12, 98, 8], [31, 90, 20, 11, 78, 117, 29, 66, 32, 22, 13, 57, 82, 14, 43, 17, 16, 102, 40, 4, 25], [86, 71, 61, 39, 110, 101, 33, 26, 10, 55, 0, 23, 36, 56, 99, 76, 58, 21, 3, 94, 59], [7, 113, 89, 91, 5, 34, 52, 80, 68, 67, 84, 88, 50, 62, 103, 104, 100, 46, 116, 107, 65], [35, 114, 47, 64, 85, 73, 109, 105, 6, 96, 45, 74, 18, 27]]

<center>

<img src="img/5.png">
</center>

### Seconde heuristique : Cluster First

```python
from sklearn.cluster import KMeans
km = KMeans(
     init="k-means++",
     n_clusters=nb_clusters,
     n_init=100,
     max_iter=600,
     random_state=42
)
km.fit(df)
clusters = km.labels_
centroids = km.cluster_centers_
```

On commence par créer des routes sans forcément respecter la condition de capacité :

<center>

<img src="img/6.png">
</center>

```
Voici les regroupements de clients par clusters :
[[0, 3, 21, 23, 36, 52, 56, 58, 59, 67, 68, 76, 80, 94, 99],
[6, 8, 13, 14, 17, 18, 22, 27, 32, 35, 43, 47, 57, 64, 73, 82, 85, 96, 105,
    109, 114],
[9, 15, 19, 20, 24, 28, 29, 30, 31, 37, 49, 51, 53, 54, 60, 66, 70, 72, 79,
    83, 87, 90, 92, 93, 95, 106, 115, 117],
[1, 2, 7, 11, 12, 38, 41, 42, 44, 48, 63, 69, 75, 77, 78, 81, 97, 98, 108,
    111, 112, 113],
[5, 10, 26, 33, 34, 39, 55, 61, 71, 86, 89, 91, 101, 110],
[4, 16, 25, 40, 45, 46, 50, 62, 65, 74, 84, 88, 100, 102, 103, 104, 107, 116]]

Voici la demande de chaque cluster (rappel : capacité max = 21) :
[15, 21, 28, 22, 14, 18]

Voici la liste des clusters capacitaires :
[0, 1, 4, 5]

Voici la liste des clusters excédentaires :
[2, 3]
```

Ensuite on applique l'heuristique suivante :<br/>
_“Les clusters excédentaires donnent par proximité aux clusters capacitaires”_

<center>

<img src="img/7.png">
</center>

Désormais nous avons un groupe de routes constituées par proximité entre les voisins, et respectant la condition de capacité des camions. Il est possible que nous ne trouvions pas d’équilibre et qu’il reste toujours un ou plusieurs clusters excédentaires. Dans ce cas, il faut augmenter **le nombre de clusters de 1** et refaire toute la méthode.

Ensuite, localement sur chaque route on choisit le parcours le plus court :

<center>

<img src="img/8.png">
</center>

### Résultats

| Instance    | Opt | Upper Bound | Best Route First | Best Cluster First |
| ----------- | --- | ----------- | ---------------- | ------------------ |
| X-n106-k14  | yes | 26362       | 27982            | 32787              |
| X-n110-k13  | yes | 14971       | 16908            | 22368              |
| X-n115-k10  | yes | 12747       | 15029            | 19931              |
| X-n313-k71  | no  | 94043       | 104643           | 139757             |
| X-n1001-k43 | no  | 72355       | 83764            | 123013             |
| X-n190-k8   | yes | 16980       | 19060            | 21758              |
