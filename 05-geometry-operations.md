# Opèrations géométriques {#geometric-operations}

## Prérequis {-}

- Ce chapitre utilise les mêmes paquets que le chapitre \@ref(spatial-operations) mais avec l'ajout de **spDataLarge**, qui a été installé dans le chapitre \@ref(spatial-class):


```r
library(sf)
library(terra)
library(dplyr)
library(spData)
library(spDataLarge)
```

## Introduction

Jusqu'à présent, ce livre a abordé la structure des jeux de données géographiques (chapitre \@ref(spatial-class)), et la manière de les manipuler en fonction de leurs attributs non géographiques (chapitre \@ref(attr)) et de leurs relations spatiales (chapitre \@ref(spatial-operations)).
Ce chapitre se concentre sur la manipulation des éléments géographiques des objets géographiques, par exemple en simplifiant et en convertissant les géométries vectorielles, en recadrant les rasters et en convertissant les objets vectoriels en rasters et les rasters en vecteurs.
Après l'avoir lu --- et avoir fait les exercices à la fin --- vous devriez comprendre et contrôler la colonne géométrique des objets `sf` ainsi que l'étendue et l'emplacement géographique des pixels représentés dans les rasters par rapport à d'autres objets géographiques.

La section \@ref(geo-vec) couvre la transformation des géométries vectorielles avec des opérations "unaires" (ou fonction avec un argument)  et "binaires" (fonction avec plus d'un argument).
Les opérations unaires portent sur une seule géométrie de manière isolée, notamment la simplification (de lignes et de polygones), la création de tampons et de centroïdes, et le déplacement/la mise à l'échelle/la rotation de géométries uniques à l'aide de " transformations affines " (sections \@ref(simplification) à \@ref(transformations affines)).
Les transformations binaires modifient une géométrie en fonction de la forme d'une autre, y compris l'écrêtage et les unions géométriques (\index{vector!union}), traités respectivement dans les sections \@ref(écrêtage) et \@ref(unions géométriques).
Les transformations de type (d'un polygone à une ligne, par exemple) sont présentées dans la section \@ref(type-trans).

La section \@ref(geo-ras) couvre les transformations géométriques sur les objets rasters.
Il s'agit de modifier la taille et le nombre des pixels, et de leur attribuer de nouvelles valeurs.
Elle enseigne comment modifier la résolution (également appelée agrégation et désagrégation), l'étendue et l'origine d'un objet matriciel.
Ces opérations sont particulièrement utiles si l'on souhaite aligner des rasters provenant de sources diverses.
Les objets rasters alignés partagent une correspondance biunivoque entre les pixels, ce qui permet de les traiter à l'aide d'opérations d'algèbre raster, décrites dans la section \@ref(map-algebra). La dernière section \@ref(raster-vector) relie les objets vectoriels et rasters. 
Elle montre comment les valeurs matricielles peuvent être "masquées" et "extraites" par des géométries vectorielles.
Il est important de noter qu'elle montre comment " polygoniser " les données raster et " rastériser " les veceurs, ce qui rend les deux modèles de données plus interchangeables.

## Opérations géométriques sur les données vectorielles {#geo-vec}

Cette section traite des opérations qui, d'une manière ou d'une autre, modifient la géométrie des objets vectoriels (`sf`).
Elle est plus avancée que les opérations sur les données spatiales présentées dans le chapitre précédent (dans la section \@ref(spatial-vec)), parce qu'ici nous allons plus loin dans la géométrie :
les fonctions présentées dans cette section fonctionnent sur les objets de la classe `sfc` en plus des objets de la classe `sf`.

### Simplification

\index{vector!simplification} 
La simplification est un processus de généralisation des objets vectoriels (lignes et polygones) généralement destiné à être utilisé dans des cartes à plus petite échelle.
Une autre raison de simplifier les objets est de réduire la quantité de mémoire, d'espace disque et de bande passante qu'ils consomment :
il peut être judicieux de simplifier des géométries complexes avant de les publier sous forme de cartes interactives. 
Le paquet **sf** fournit `st_simplify()`, qui utilise l'implémentation GEOS de l'algorithme de Douglas-Peucker pour réduire le nombre de sommets.
`st_simplify()` utilise la `dTolerance` pour contrôler le niveau de généralisation des unités de la carte [voir @douglas_algorithms_1973 pour plus de détails].
La figure \@ref(fig:seine-simp) illustre la simplification d'une géométrie `LINESTRING` représentant la Seine et ses affluents.
La géométrie simplifiée a été créée par la commande suivante :


```r
seine_simp = st_simplify(seine, dTolerance = 2000)  # 2000 m
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/seine-simp-1.png" alt="Comparaison de la géométrie originale et simplifiée de la Seine." width="100%" />
<p class="caption">(\#fig:seine-simp)Comparaison de la géométrie originale et simplifiée de la Seine.</p>
</div>

L'objet `seine_simp` résultant est une copie de l'objet original `seine` mais avec moins de vertices.
Le résultat étant visuellement plus simple (Figure \@ref(fig:seine-simp), à droite) et consommant moins de mémoire que l'objet original, comme vérifié ci-dessous :


```r
object.size(seine)
#> 18096 bytes
object.size(seine_simp)
#> 9112 bytes
```

La simplification est également applicable aux polygones.
Ceci est illustré par l'utilisation de `us_states`, représentant les États-Unis contigus.
Comme nous le montrons dans le chapitre \@ref(reproj-geo-data), GEOS suppose que les données sont dans un CRS projeté et cela pourrait conduire à des résultats inattendus lors de l'utilisation d'un CRS géographique.
Par conséquent, la première étape consiste à projeter les données dans un CRS projeté adéquat, tel que le US National Atlas Equal Area (epsg = 2163) (à gauche sur la figure \@ref(fig:us-simp)) :





```r
us_states2163 = st_transform(us_states, "EPSG:2163")
us_states2163 = us_states2163 %>% 
  mutate(AREA = as.numeric(AREA)) 
```

`st_simplify()` works equally well with projected polygons:


```r
us_states_simp1 = st_simplify(us_states2163, dTolerance = 100000)  # 100 km
```

Une limitation de `st_simplify()` est qu'il simplifie les objets sur une base géométrique.
Cela signifie que la "topologie" est perdue, ce qui donne lieu à des polygones se superposant ou séparés par des vides, comme le montre la figure \@ref(fig:us-simp) (panneau du milieu).
`ms_simplify()` de **rmapshaper** fournit une alternative qui surmonte ce problème.
Par défaut, il utilise l'algorithme de Visvalingam, qui surmonte certaines limitations de l'algorithme de Douglas-Peucker [@visvalingam_line_1993].
<!-- https://bost.ocks.org/mike/simplify/ -->
L'extrait de code suivant utilise cette fonction pour simplifier `us_states2163`.
Le résultat n'a que 1% des sommets de l'entrée (fixée à l'aide de l'argument `keep`) mais son nombre d'objets reste intact car nous avons fixé `keep_shapes = TRUE` :^[
La simplification des objets multi-polygones peut supprimer les petits polygones internes, même si l'argument `keep_shapes` est défini à TRUE. Pour éviter cela, vous devez définir `explode = TRUE`. Cette option convertit tous les mutlipolygones en polygones séparés avant leur simplification.
]


```r
# proportion des points à garder (0-1; par defaut 0.05)
us_states_simp2 = rmapshaper::ms_simplify(us_states2163, keep = 0.01,
                                          keep_shapes = TRUE)
```

Enfin, la comparaison visuelle de l'ensemble de données originales et des deux versions simplifiées montre des différences entre les sorties des algorithmes de Douglas-Peucker (`st_simplify`) et de Visvalingam (`ms_simplify`) (Figure \@ref(fig:us-simp)) :

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/us-simp-1.png" alt="Simplification des polygones, comparant la géométrie originale des États-Unis continentaux avec des versions simplifiées, générées avec les fonctions des paquets sf (au centre) et rmapshaper (à droite)." width="100%" />
<p class="caption">(\#fig:us-simp)Simplification des polygones, comparant la géométrie originale des États-Unis continentaux avec des versions simplifiées, générées avec les fonctions des paquets sf (au centre) et rmapshaper (à droite).</p>
</div>

### Centroïdes

\index{vector!centroids} 
Les opérations de centroïdes identifient le centre des objets géographiques.
Comme pour les mesures statistiques de tendance centrale (y compris les définitions de la moyenne et de la médiane), il existe de nombreuses façons de définir le centre géographique d'un objet.
Toutes créent des représentations par un point unique d'objets vectoriels plus complexes.

Le *centroïde géographique* est sans doute l'opération la plus couramment utilisée.
Ce type d'opération (souvent jute appelé "centroïde") représente le centre de masse d'un objet spatial (pensez à une assiette en équilibre sur votre doigt).
Les centroïdes géographiques ont de nombreuses utilisations, par exemple pour créer une représentation ponctuelle simple de géométries complexes, ou pour estimer les distances entre polygones.
Ils peuvent être calculés à l'aide de la fonction **sf** `st_centroid()`, comme le montre le code ci-dessous, qui génère les centroïdes géographiques de régions de Nouvelle-Zélande et d'affluents de la Seine, illustrés par des points noirs sur la figure \@ref(fig:centr).


```r
nz_centroid = st_centroid(nz)
seine_centroid = st_centroid(seine)
```

Parfois, le centroïde géographique se trouve en dehors des limites de l'objet parent (pensez à un beignet).
Dans ce cas, les opérations dites de *point sur la surface* peuvent être utilisées pour garantir que le point se trouvera dans l'objet parent (par exemple, pour étiqueter des objets de type multipolygones irréguliers tels que des îles), comme l'illustrent les points rouges de la figure \@ref(fig:centr).
Remarquez que ces points rouges se trouvent toujours sur leurs objets parents.
Ils ont été créés avec `st_point_on_surface()` comme suit :^[
Une description du fonctionnement de `st_point_on_surface()` est fournie sur https://gis.stackexchange.com/q/76498.
]


```r
nz_pos = st_point_on_surface(nz)
seine_pos = st_point_on_surface(seine)
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/centr-1.png" alt="Centroïdes (points noirs) et points sur la surface (points rouges) des ensembles de données des régions de Nouvelle-Zélande (à gauche) et de la Seine (à droite)." width="100%" />
<p class="caption">(\#fig:centr)Centroïdes (points noirs) et points sur la surface (points rouges) des ensembles de données des régions de Nouvelle-Zélande (à gauche) et de la Seine (à droite).</p>
</div>

Il existe d'autres types de centroïdes, notamment le *centre de Chebyshev* et le *centre visuel*.
Nous ne les explorerons pas ici, mais il est possible de les calculer à l'aide de R, comme nous le verrons dans le chapitre \@ref(algorithms).

### Buffers/tampons

\index{vector!buffers} 
Les buffers ou tampons sont des polygones représentant la zone située à une distance donnée d'une caractéristique géométrique :
Que le type d'origine soit un point, une ligne ou un polygone, la sortie est toujours un polygone.
Contrairement à la simplification (qui est souvent utilisée pour la visualisation et la réduction de la taille des fichiers), la mise en mémoire tampon est généralement utilisée pour l'analyse des données géographiques.
Combien de points se trouvent à une distance donnée de cette ligne ?
Quels groupes démographiques se trouvent à une distance de déplacement de ce nouveau magasin ?
Il est possible de répondre à ce genre de questions et de les visualiser en créant des tampons autour des entités géographiques d'intérêt.

La figure \@ref(fig:buffs) illustre des buffers de différentes tailles (5 et 50 km) entourant la Seine et ses affluents.
Les commandes ci-dessous, utilisées pour créer ces buffers,  montrent que la commande `st_buffer()` nécessite au moins deux arguments : une géométrie d'entrée et une distance, fournie dans les unités du SRC (dans ce cas, les mètres) :


```r
seine_buff_5km = st_buffer(seine, dist = 5000)
seine_buff_50km = st_buffer(seine, dist = 50000)
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/buffs-1.png" alt="Tampons de 5 km autour du jeu de données de la Seine  (à gauche) et de 50 km (à droite). Notez les couleurs, qui reflètent le fait qu'un tampon est créé par élément géométrique." width="75%" />
<p class="caption">(\#fig:buffs)Tampons de 5 km autour du jeu de données de la Seine  (à gauche) et de 50 km (à droite). Notez les couleurs, qui reflètent le fait qu'un tampon est créé par élément géométrique.</p>
</div>

\BeginKnitrBlock{rmdnote}<div class="rmdnote">Le troisième et dernier argument de `st_buffer()` est `nQuadSegs`, qui signifie 'nombre de segments par quadrant' et qui est fixé par défaut à 30 (ce qui signifie que les cercles créés par les buffers sont composés de $4 \times 30 = 120$ lignes).
Cet argument a rarement besoin d´être défini.
Les cas inhabituels où il peut être utile incluent lorsque la mémoire consommée par la sortie d´une opération de tampon est une préoccupation majeure (dans ce cas, il devrait être réduit) ou lorsque la très haute précision est nécessaire (dans ce cas, il devrait être augmenté).</div>\EndKnitrBlock{rmdnote}



### Application affine

\index{vector!affine transformation} 
Une application affine est une transformation qui préserve les lignes et le parallélisme.
Cependant, les angles ou la longueur ne sont pas nécessairement préservés.
Les transformations affines comprennent, entre autres, le déplacement (translation), la mise à l'échelle et la rotation.
En outre, il est possible d'utiliser n'importe quelle combinaison de celles-ci.
Les applications affines sont une partie essentielle de la géocomputation.
Par exemple, le décalage est nécessaire pour le placement d'étiquettes, la mise à l'échelle est utilisée dans les cartogrammes de zones non contiguës (voir la section \@ref(other-mapping-packages)), et de nombreuses transformations affines sont appliquées lors de la reprojection ou de l'amélioration de la géométrie créée à partir d'une carte déformée ou mal projetée.
Le paquet **sf** implémente la transformation affine pour les objets des classes `sfg` et `sfc`.


```r
nz_sfc = st_geometry(nz)
```

Le décalage déplace chaque point de la même distance en unités cartographiques.
Cela peut être fait en ajoutant un vecteur numérique à un objet vectoriel.
Par exemple, le code ci-dessous déplace toutes les coordonnées y de 100 000 mètres vers le nord, mais laisse les coordonnées x intactes (panneau gauche de la figure \@ref(fig:affine-trans)).  


```r
nz_shift = nz_sfc + c(0, 100000)
```

La mise à l'échelle agrandit ou rétrécit les objets par un facteur.
Elle peut être appliquée de manière globale ou locale.
La mise à l'échelle globale augmente ou diminue toutes les valeurs des coordonnées par rapport aux coordonnées d'origine, tout en gardant intactes les relations topologiques de toutes les géométries.
Elle peut être effectuée par soustraction ou multiplication d'un objet `sfg` ou `sfc`.



Le changement à l'échelle locale traite les géométries indépendamment et nécessite des points autour desquels les géométries vont être mises à l'échelle, par exemple des centroïdes.
Dans l'exemple ci-dessous, chaque géométrie est réduite d'un facteur deux autour des centroïdes (panneau central de la figure \@ref(fig:affine-trans)).
Pour cela, chaque objet est d'abord décalé de manière à ce que son centre ait les coordonnées `0, 0` (`(nz_sfc - nz_centroid_sfc)`). 
Ensuite, les tailles des géométries sont réduites de moitié (`* 0.5`).
Enfin, le centroïde de chaque objet est ramené aux coordonnées des données d'entrée (`+ nz_centroid_sfc`). 


```r
nz_centroid_sfc = st_centroid(nz_sfc)
nz_scale = (nz_sfc - nz_centroid_sfc) * 0.5 + nz_centroid_sfc
```

La rotation de coordonnées bidimensionnelles nécessite une matrice de rotation :

$$
R =
\begin{bmatrix}
\cos \theta & -\sin \theta \\  
\sin \theta & \cos \theta \\
\end{bmatrix}
$$

Elle fait tourner les points dans le sens des aiguilles d'une montre.
La matrice de rotation peut être implémentée dans R comme suit :


```r
rotation = function(a){
  r = a * pi / 180 #degrées en radians
  matrix(c(cos(r), sin(r), -sin(r), cos(r)), nrow = 2, ncol = 2)
} 
```

La fonction `rotation` accepte un argument `a` - un angle de rotation en degrés.
La rotation peut être effectuée autour de points sélectionnés, comme les centroïdes (panneau de droite de la figure \@ref(fig:affine-trans)).
Voir `vignette("sf3")` pour plus d'exemples.


```r
nz_rotate = (nz_sfc - nz_centroid_sfc) * rotation(30) + nz_centroid_sfc
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/affine-trans-1.png" alt="Illustrations des transformations affines : décalage, échelle et rotation." width="100%" />
<p class="caption">(\#fig:affine-trans)Illustrations des transformations affines : décalage, échelle et rotation.</p>
</div>





Enfin, les géométries nouvellement créées peuvent remplacer les anciennes avec la fonction `st_set_geometry()` : 


```r
nz_scale_sf = st_set_geometry(nz, nz_scale)
```

### Découper {#clipping}

\index{vector!clipping} 
\index{spatial!subsetting} 
Le découpage spatial est une forme de sélection spatiale qui implique des changements dans les colonnes `géométriques` d'au moins certaines des entités affectées.

Le découpage ne peut s'appliquer qu'à des éléments plus complexes que des points : 
les lignes, les polygones et leurs équivalents "multi".
Pour illustrer le concept, nous allons commencer par un exemple simple :
deux cercles superposés dont le point central est distant d'une unité et dont le rayon est de un (Figure \@ref(fig:points)).


```r
b = st_sfc(st_point(c(0, 1)), st_point(c(1, 1))) # créer 2 points
b = st_buffer(b, dist = 1) # convertir les points en cercles
plot(b, border = "grey")
text(x = c(-0.5, 1.5), y = 1, labels = c("x", "y"), cex = 3) # ajout du texte
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/points-1.png" alt="cercles superposés." width="100%" />
<p class="caption">(\#fig:points)cercles superposés.</p>
</div>

Imaginez que vous voulez sélectionner non pas un cercle ou l'autre, mais l'espace couvert par les deux `x` *et* `y`.
Cela peut être fait en utilisant la fonction `st_intersection()`\index{vecteur!intersection}, illustrée en utilisant des objets nommés `x` et `y` qui représentent les cercles de gauche et de droite (Figure \@ref(fig:circle-intersection)).


```r
x = b[1]
y = b[2]
x_and_y = st_intersection(x, y)
plot(b, border = "grey")
plot(x_and_y, col = "lightgrey", border = "grey", add = TRUE) # surface intersectée
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/circle-intersection-1.png" alt="Cercles superposés avec une couleur grise pour indiquer l'intersection entre eux" width="100%" />
<p class="caption">(\#fig:circle-intersection)Cercles superposés avec une couleur grise pour indiquer l'intersection entre eux</p>
</div>

Le passage de code suivant montre comment cela fonctionne pour toutes les combinaisons du diagramme de Venn représentant `x` et `y`, inspiré de la [Figure 5.1](http://r4ds.had.co.nz/transform.html#logical-operators) du livre *R for Data Science* [@grolemund_r_2016].

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/venn-clip-1.png" alt="Équivalents spatiaux des opérateurs logiques." width="100%" />
<p class="caption">(\#fig:venn-clip)Équivalents spatiaux des opérateurs logiques.</p>
</div>

### Sélection et découpage

Le découpage d'objets peut modifier leur géométrie, mais il peut également sélectionner des objets, en ne renvoyant que les entités qui intersectent (ou intersectent partiellement) un objet de découpage/sélection.
Pour illustrer ce point, nous allons sélectionner les points qui incluent dans le cadre englobant (*bounding box*) des cercles `x` et `y` de la figure \@ref(fig:venn-clip).
Certains points seront à l'intérieur d'un seul cercle, d'autres à l'intérieur des deux et d'autres encore à l'intérieur d'aucun.
`st_sample()` est utilisé ci-dessous pour générer une distribution *simple et aléatoire* de points à l'intérieur de l'étendue des cercles `x` et `y`, ce qui donne le résultat illustré dans la Figure \@ref(fig:venn-subset), ce qui soulève la question suivante : comment sous-ensembler les points pour ne renvoyer que le point qui intersecte *à la fois* `x` et `y` ?

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/venn-subset-1.png" alt="Points distribués de manière aléatoire dans le cadre englobant les cercles x et y. Les points qui croisent les deux objets x et y sont mis en évidence." width="100%" />
<p class="caption">(\#fig:venn-subset)Points distribués de manière aléatoire dans le cadre englobant les cercles x et y. Les points qui croisent les deux objets x et y sont mis en évidence.</p>
</div>



```r
bb = st_bbox(st_union(x, y))
box = st_as_sfc(bb)
set.seed(2017)
p = st_sample(x = box, size = 10)
x_and_y = st_intersection(x, y)
```

Le code ci-dessous montre trois façons d'obtenir le même résultat.
Nous pouvons utiliser directement l'intersection `index{vecteur!intersection} de `x` et `y` (représentée par `x_et_y` dans l'extrait de code précédent) comme objet de sélection, comme le montre la première ligne du morceau de code ci-dessous.
Nous pouvons également trouver l'intersection entre les points d'entrée représentés par `p` et l'objet de sélection et de découpage `x_et_y`, comme le montre la deuxième ligne du code ci-dessous.
Cette deuxième approche renvoie les entités qui ont une intersection partielle avec `x_and_y` mais avec des géométries modifiées pour les entités dont les surfaces recoupent celle de l'objet de sélection.
La troisième approche consiste à créer un objet de sélection en utilisant le prédicat spatial binaire `st_intersects()`, introduit dans le chapitre précédent.
Les résultats sont identiques (à l'exception de différences superficielles dans les noms d'attributs), mais l'implémentation diffère substantiellement :


```r
p_xy1 = p[x_and_y]
p_xy2 = st_intersection(p, x_and_y)
sel_p_xy = st_intersects(p, x, sparse = FALSE)[, 1] &
  st_intersects(p, y, sparse = FALSE)[, 1]
p_xy3 = p[sel_p_xy]
```



Bien que l'exemple ci-dessus soit plutôt trivial et fourni à des fins éducatives plutôt qu'appliquées, et que nous encouragions le lecteur à reproduire les résultats pour approfondir sa compréhension de la manipulation des objets vectoriels géographiques dans R, il soulève une question importante : quelle implémentation utiliser ?
En général, les implémentations les plus concises doivent être privilégiées, ce qui signifie la première approche ci-dessus.
Nous reviendrons sur la question du choix entre différentes implémentations d'une même technique ou d'un même algorithme au chapitre \@ref(algorithmes).

### GUnions de géométries

\index{vector!union} 
\index{aggregation!spatial} 
Comme nous l'avons vu dans la section \@ref(vector-attribute-aggregation), l'agrégation spatiale peut dissoudre silencieusement les géométries des polygones se touchant dans le même groupe.
Cela est démontré dans le code ci-dessous dans lequel 49 `us_states` sont agrégés en 4 régions à l'aide des fonctions de R base et du **tidyverse**\index{tidyverse (package)} (voir les résultats dans la figure \@ref(fig:us-regions)) :


```r
regions = aggregate(x = us_states[, "total_pop_15"], by = list(us_states$REGION),
                    FUN = sum, na.rm = TRUE)
regions2 = us_states %>% group_by(REGION) %>%
  summarize(pop = sum(total_pop_15, na.rm = TRUE))
```



<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/us-regions-1.png" alt="Agrégation spatiale sur des polygones contigus, illustrée par l'agrégation de la population des États américains en régions, la population étant représentée par une couleur. Notez que l'opération dissout automatiquement les frontières entre les états." width="100%" />
<p class="caption">(\#fig:us-regions)Agrégation spatiale sur des polygones contigus, illustrée par l'agrégation de la population des États américains en régions, la population étant représentée par une couleur. Notez que l'opération dissout automatiquement les frontières entre les états.</p>
</div>

Que se passe-t-il au niveau des géométries ?
En coulisses, `aggregate()` et `summarize()` combinent les géométries et dissolvent les frontières entre elles en utilisant `st_union()`.
Ceci est démontré par le code ci-dessous qui crée une union des Etats-Unis de l'Ouest :  


```r
us_west = us_states[us_states$REGION == "West", ]
us_west_union = st_union(us_west)
```

La fonction peut prendre deux géométries et les unir, comme le montre ll code ci-dessous qui crée un bloc occidental uni incorporant le Texas (défi : reproduire et représenter le résultat) :


```r
texas = us_states[us_states$NAME == "Texas", ]
texas_union = st_union(us_west_union, texas)
```



### Transformations de type {#type-trans}

\index{vector!geometry casting} 
La transformation d'un type de géométrie en un autre (*casting*)  est une opération puissante.
Elle est implémentée dans la fonction `st_cast()` du package **sf**.
Il est important de noter que la fonction `st_cast()` se comporte différemment selon qu'il s'agit d'un objet géométrique simple (`sfg`), d'une colonne géométrique simple (`sfc`) ou d'un objet simple.

Créons un multipoint pour illustrer le fonctionnement des transformations de type géométrique sur des objets de géométrie simple (`sfg`) :


```r
multipoint = st_multipoint(matrix(c(1, 3, 5, 1, 3, 1), ncol = 2))
```

Dans ce cas, `st_cast()` peut être utile pour transformer le nouvel objet en *linestring* (ligne) ou en polygone (Figure \@ref(fig:single-cast)) :


```r
linestring = st_cast(multipoint, "LINESTRING")
polyg = st_cast(multipoint, "POLYGON")
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/single-cast-1.png" alt="Exemples de lignes et de polygones créés à partir d'une géométrie multipoint" width="100%" />
<p class="caption">(\#fig:single-cast)Exemples de lignes et de polygones créés à partir d'une géométrie multipoint</p>
</div>

La conversion de multipoint en ligne est une opération courante qui crée un objet ligne à partir d'observations ponctuelles ordonnées, telles que des mesures GPS ou des sources géolocalisés.
Cela permet d'effectuer des opérations spatiales telles que la longueur du chemin parcouru.
La conversion de multipoint ou de *linestring* en polygone est souvent utilisée pour calculer une surface, par exemple à partir de l'ensemble des mesures GPS prises autour d'un lac ou des coins d'un terrain à bâtir.

Le processus de transformation peut également être inversé en utilisant `st_cast()` :


```r
multipoint_2 = st_cast(linestring, "MULTIPOINT")
multipoint_3 = st_cast(polyg, "MULTIPOINT")
all.equal(multipoint, multipoint_2)
#> [1] TRUE
all.equal(multipoint, multipoint_3)
#> [1] TRUE
```

\BeginKnitrBlock{rmdnote}<div class="rmdnote">Pour les géométries d´entités simples (`sfg`), `st_cast` permet également de transformer des géométries de non-multi-types vers des multi-types (par exemple, `POINT` vers `MULTIPOINT`) et de multi-types vers des non-multi-types.
Toutefois, dans le deuxième groupe de cas, seul le premier élément de l´ancien objet est conservé.</div>\EndKnitrBlock{rmdnote}



La transformation en différent types géométrique des colonnes géométriques d'entités simples (`sfc`) et des objets d'entités simples fonctionnent de la même manière que pour les géométries simples (`sfg`) dans la plupart des cas. 
Une différence importante est la conversion des multi-types en non-multi-types.
À la suite de ce processus, les multi-objets, `sf` ou `sfg` sont divisés en plusieurs non-multi-objets.

Le tableau \@ref(tab:sfs-st-cast) montre les transformations de type géométrique possibles sur les objets d'entités simples.
Les géométries d'entités simples (représentées par la première colonne du tableau) peuvent être transformées en plusieurs types de géométrie, représentés par les colonnes du tableau \@ref(tab:sfs-st-cast)
Plusieurs des transformations ne sont pas possibles, par exemple, vous ne pouvez pas convertir un point unique en un multilinestring ou un polygone (ainsi les cellules `[1, 4:5]` dans le tableau sont NA).
Certaines transformations divisent l'objet d'entrée: on passe d'un élément unique en un objet à éléments multiples.
Lorsqu'une géométrie multipoint constituée de cinq paires de coordonnées est transformée en géométrie "POINT", par exemple, la sortie contiendra cinq entités.

<table>
<caption>(\#tab:sfs-st-cast)Transformation de type de géométrie sur des entités simples (voir section 2.1) avec un type d'entrée par ligne et type de sortie par colonne</caption>
 <thead>
  <tr>
   <th style="text-align:left;">  </th>
   <th style="text-align:right;"> POI </th>
   <th style="text-align:right;"> MPOI </th>
   <th style="text-align:right;"> LIN </th>
   <th style="text-align:right;"> MLIN </th>
   <th style="text-align:right;"> POL </th>
   <th style="text-align:right;"> MPOL </th>
   <th style="text-align:right;"> GC </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> POI(1) </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MPOI(1) </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> LIN(1) </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MLIN(1) </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> POL(1) </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> MPOL(1) </td>
   <td style="text-align:right;"> 10 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> GC(1) </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
</tbody>
<tfoot>
<tr>
<td style = 'padding: 0; border:0;' colspan='100%'><sup></sup> Note : Les valeurs comme (1) représentent le nombre d'entités ; NA signifie que l'opération n'est pas possible. Abréviations : POI, LIN, POL et GC font référence à POINT, LINESTRING, POLYGON et GEOMETRYCOLLECTION. La version MULTI de ces types de géométrie est indiquée par un M précédent, par exemple, MPOI est l'acronyme de MULTIPOINT.</td>
</tr>
</tfoot>
</table>

Essayons d'appliquer des transformations de type géométrique sur un nouvel objet, `multilinestring_sf`, à titre d'exemple (à gauche sur la Figure \@ref(fig:line-cast)) :


```r
multilinestring_list = list(matrix(c(1, 4, 5, 3), ncol = 2), 
                            matrix(c(4, 4, 4, 1), ncol = 2),
                            matrix(c(2, 4, 2, 2), ncol = 2))
multilinestring = st_multilinestring((multilinestring_list))
multilinestring_sf = st_sf(geom = st_sfc(multilinestring))
multilinestring_sf
#> Simple feature collection with 1 feature and 0 fields
#> Geometry type: MULTILINESTRING
#> Dimension:     XY
#> Bounding box:  xmin: 1 ymin: 1 xmax: 4 ymax: 5
#> CRS:           NA
#>                             geom
#> 1 MULTILINESTRING ((1 5, 4 3)...
```

Vous pouvez l'imaginer comme un réseau routier ou fluvial. 
Le nouvel objet n'a qu'une seule ligne qui définit toutes les lignes.
Cela limite le nombre d'opérations qui peuvent être faites, par exemple, cela empêche d'ajouter des noms à chaque segment de ligne ou de calculer les longueurs des lignes individuelles.
La fonction `st_cast()` peut être utilisée dans cette situation, car elle sépare un mutlilinestring en trois linestrings :


```r
linestring_sf2 = st_cast(multilinestring_sf, "LINESTRING")
linestring_sf2
#> Simple feature collection with 3 features and 0 fields
#> Geometry type: LINESTRING
#> Dimension:     XY
#> Bounding box:  xmin: 1 ymin: 1 xmax: 4 ymax: 5
#> CRS:           NA
#>                    geom
#> 1 LINESTRING (1 5, 4 3)
#> 2 LINESTRING (4 4, 4 1)
#> 3 LINESTRING (2 2, 4 2)
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/line-cast-1.png" alt="Exemples de transformation de type de géométrie entre MULTILINESTRING (à gauche) et LINESTRING (à droite)." width="100%" />
<p class="caption">(\#fig:line-cast)Exemples de transformation de type de géométrie entre MULTILINESTRING (à gauche) et LINESTRING (à droite).</p>
</div>

Le nouvel objet permet la création d'attributs (voir la section \@ref(vec-attr-creation)) et la mesure de la longueur :


```r
linestring_sf2$name = c("Riddle Rd", "Marshall Ave", "Foulke St")
linestring_sf2$length = st_length(linestring_sf2)
linestring_sf2
#> Simple feature collection with 3 features and 2 fields
#> Geometry type: LINESTRING
#> Dimension:     XY
#> Bounding box:  xmin: 1 ymin: 1 xmax: 4 ymax: 5
#> CRS:           NA
#>                    geom         name length
#> 1 LINESTRING (1 5, 4 3)    Riddle Rd   3.61
#> 2 LINESTRING (4 4, 4 1) Marshall Ave   3.00
#> 3 LINESTRING (2 2, 4 2)    Foulke St   2.00
```

## Opérations géométriques sur les données raster {#geo-ras}

\index{raster!manipulation} 
Les opérations  géométriques sur des raster comprennent le décalage, le retournement, la mise en miroir, la mise à l'échelle, la rotation ou la déformation des images.
Ces opérations sont nécessaires pour une variété d'applications, y compris le géoréférencement, utilisé pour permettre aux images d'être superposées sur une carte précise avec un CRS connu [@liu_essential_2009].
Il existe une variété de techniques de géoréférencement, notamment :

- Géorectification basée sur des [points de contrôle au sol](https://www.qgistutorials.com/en/docs/3/georeferencing_basics.html) connus 
- Orthorectification, qui tient également compte de la topographie locale.
- L'[enregistrement](https://en.wikipedia.org/wiki/Image_registration) d'images  est utilisé pour combiner des images de la même chose mais prises par différents capteurs en alignant une image sur une autre (en termes de système de coordonnées et de résolution).

R est plutôt inadapté pour les deux premiers points car ceux-ci nécessitent souvent une intervention manuelle, c'est pourquoi ils sont généralement réalisés à l'aide d'un logiciel SIG dédié (voir également le chapitre : \@ref(gis)).
En revanche, l'alignement de plusieurs images est possible dans R et cette section montre entre autres comment le faire.
Cela implique souvent de modifier l'étendue, la résolution et l'origine d'une image.
Une projection correspondante est bien sûr également nécessaire, mais elle est déjà traitée dans la section \@ref(reproj-ras).

Dans tous les cas, il existe d'autres raisons d'effectuer une opération géométrique sur une seule image raster.
Par exemple, dans le chapitre \@ref(location) nous définissons les zones métropolitaines en Allemagne comme des pixels de 20 km^2^ avec plus de 500.000 habitants. 
La trame d'habitants d'origine a cependant une résolution de 1 km^2^, c'est pourquoi nous allons diminuer (agréger) la résolution d'un facteur 20 (voir le chapitre \@ref(define-metropolitan-areas)).
Une autre raison d'agréger une image matricielle est simplement de réduire le temps d'exécution ou d'économiser de l'espace disque.
Bien entendu, cela n'est possible que si la tâche à accomplir permet une résolution plus grossière.
Parfois, une résolution plus grossière est suffisante!

### Intersections géométriques

\index{raster!intersection} 
Dans la section \@ref(spatial-raster-subsetting), nous avons montré comment extraire des valeurs d'un raster superposé à d'autres objets spatiaux.
Pour récupérer une sortie spatiale, nous pouvons utiliser pratiquement la même syntaxe de sélection.
La seule différence est que nous devons préciser que nous souhaitons conserver la structure matricielle en mettant l'argument `drop` à `FALSE`.
Ceci retournera un objet raster contenant les cellules dont les points médians se chevauchent avec `clip`.


```r
elev = rast(system.file("raster/elev.tif", package = "spData"))
clip = rast(xmin = 0.9, xmax = 1.8, ymin = -0.45, ymax = 0.45,
            resolution = 0.3, vals = rep(1, 9))
elev[clip, drop = FALSE]
#> class       : SpatRaster 
#> dimensions  : 2, 1, 1  (nrow, ncol, nlyr)
#> resolution  : 0.5, 0.5  (x, y)
#> extent      : 1, 1.5, -0.5, 0.5  (xmin, xmax, ymin, ymax)
#> coord. ref. : lon/lat WGS 84 (EPSG:4326) 
#> source      : memory 
#> name        : elev 
#> min value   :   18 
#> max value   :   24
```

Pour la même opération, nous pouvons également utiliser les commandes `intersect()` et `crop()`.

### Étendue et origine

\index{raster!merging} 
Lors de la fusion ou de l'exécution de l'algèbre raster sur des rasters, leur résolution, leur projection, leur origine et/ou leur étendue doivent correspondre. Sinon, comment ajouter les valeurs d'un raster ayant une résolution de 0,2 degré décimal à un second raster ayant une résolution de 1 degré décimal ?
Le même problème se pose lorsque nous souhaitons fusionner des images satellite provenant de différents capteurs avec des projections et des résolutions différentes. 
Nous pouvons traiter de telles disparités en alignant les trames.

Dans le cas le plus simple, deux images ne diffèrent que par leur étendue.
Le code suivant ajoute une ligne et deux colonnes de chaque côté de l'image raster tout en fixant toutes les nouvelles valeurs à une altitude de 1000 mètres (Figure \@ref(fig:extend-example)).


```r
elev = rast(system.file("raster/elev.tif", package = "spData"))
elev_2 = extend(elev, c(1, 2))
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/extend-example-1.png" alt="Trame originale (à gauche) et la même trame (à droite) agrandie d'une ligne en haut et en bas et de deux colonnes à gauche et à droite." width="100%" />
<p class="caption">(\#fig:extend-example)Trame originale (à gauche) et la même trame (à droite) agrandie d'une ligne en haut et en bas et de deux colonnes à gauche et à droite.</p>
</div>

Performing an algebraic operation on two objects with differing extents in R, the **terra** package returns an error.


```r
elev_3 = elev + elev_2
#> Error: [+] extents do not match
```

Cependant, nous pouvons aligner l'étendue de deux rasters avec `extend()`. 
Au lieu d'indiquer à la fonction le nombre de lignes ou de colonnes à ajouter (comme nous l'avons fait précédemment), nous lui permettons de le déterminer en utilisant un autre objet raster.
Ici, nous étendons l'objet `elev` à l'étendue de `elev_2`. 
Les lignes et colonnes nouvellement ajoutées reçoivent  `NA`.


```r
elev_4 = extend(elev, elev_2)
```

L'origine d'un raster est le coin de la cellule le plus proche des coordonnées (0, 0).
La fonction `origin()` renvoie les coordonnées de l'origine.
Dans l'exemple ci-dessous, un coin de cellule existe avec les coordonnées (0, 0), mais ce n'est pas toujours le cas.


```r
origin(elev_4)
#> [1] 0 0
```

Si deux rasters ont des origines différentes, leurs cellules ne se chevauchent pas complètement, ce qui rends l'algèbre raster impossible.
Pour changer l'origine -- utilisez `origin()`.^[
Si les origines de deux données matricielles ne sont que marginalement éloignées, il suffit parfois d'augmenter l'argument `tolerance` de `terra::terraOptions()`.
]
La figure \@ref(fig:origin-example) révèle l'effet de la modification de l'origine de cette manière.


```r
# changer l'origine
origin(elev_4) = c(0.25, 0.25)
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/origin-example-1.png" alt="Rasters avec des valeurs identiques mais des origines différentes." width="100%" />
<p class="caption">(\#fig:origin-example)Rasters avec des valeurs identiques mais des origines différentes.</p>
</div>

Notez que le changement de résolution (section suivante) modifie souvent aussi l'origine.

### Agrégation et désagrégation

\index{raster!aggregation} 
\index{raster!disaggregation} 
Les jeux de données raster peuvent également différer en ce qui concerne leur résolution. 
Pour faire correspondre les résolutions, on peut soit diminuer (`aggregate()`) soit augmenter (`disagg()`) la résolution des rasters.^[
Nous faisons ici référence à la résolution spatiale.
En télédétection, les résolutions spectrale (bandes spectrales), temporelle (observations dans le temps de la même zone) et radiométrique (profondeur de couleur) sont également importantes.
Consultez l'exemple `tapp()` dans la documentation pour avoir une idée sur la façon de faire une agrégation de raster temporel.
]
À titre d'exemple, nous modifions ici la résolution spatiale de `dem` (trouvé dans le paquet **spDataLarge**) par un facteur 5 (Figure \@ref(fig:aggregate-example)).
De plus, la valeur de la cellule de sortie doit correspondre à la moyenne des cellules d'entrée (notez que l'on pourrait également utiliser d'autres fonctions, telles que `median()`, `sum()`, etc ):


```r
dem = rast(system.file("raster/dem.tif", package = "spDataLarge"))
dem_agg = aggregate(dem, fact = 5, fun = mean)
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/aggregate-example-1.png" alt="Raster original (gauche). Raster agrégé (droite)." width="100%" />
<p class="caption">(\#fig:aggregate-example)Raster original (gauche). Raster agrégé (droite).</p>
</div>

La fonction `disagg()` augmente la résolution des objets matriciels, en fournissant deux méthodes pour assigner des valeurs aux cellules nouvellement créées : la méthode par défaut (`method = "near"`) donne simplement à toutes les cellules de sortie la valeur de la cellule d'entrée, et donc duplique les valeurs, ce qui conduit à une sortie "en bloc".
La méthode `bilinear` utilise les quatre centres de pixels les plus proches de l'image d'entrée (points de couleur saumon sur la figure \@ref(fig:bilinear)) pour calculer une moyenne pondérée par la distance (flèches sur la figure \@ref(fig:bilinear).
La valeur de la cellule de sortie est représentée par un carré dans le coin supérieur gauche de la figure \@ref(fig:bilinear)).


```r
dem_disagg = disagg(dem_agg, fact = 5, method = "bilinear")
identical(dem, dem_disagg)
#> [1] FALSE
```

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/bilinear-1.png" alt="La moyenne pondérée par la distance des quatre cellules d'entrée les plus proches détermine la sortie lors de l'utilisation de la méthode bilinéaire pour la désagrégation." width="100%" />
<p class="caption">(\#fig:bilinear)La moyenne pondérée par la distance des quatre cellules d'entrée les plus proches détermine la sortie lors de l'utilisation de la méthode bilinéaire pour la désagrégation.</p>
</div>

En comparant les valeurs de `dem` et `dem_disagg`, on constate qu'elles ne sont pas identiques (vous pouvez aussi utiliser `compareGeom()` ou `all.equal()`).
Cependant, il ne fallait pas s'y attendre, puisque la désagrégation est une simple technique d'interpolation.
Il est important de garder à l'esprit que la désagrégation permet d'obtenir une résolution plus fine ; les valeurs correspondantes, cependant, ne peuvent qu'êtres aussi précises que leur source de résolution initiale.

### Rééchantillonnage

\index{raster!resampling}
Les méthodes d'agrégation et de désagrégation ci-dessus ne conviennent que lorsque nous voulons modifier la résolution de notre raster par le facteur d'agrégation/désagrégation. 
Cependant, que faire lorsque nous avons deux ou plusieurs raster avec des résolutions et des origines différentes ?
C'est le rôle du rééchantillonnage - un processus de calcul des valeurs pour les nouveaux emplacements des pixels.
En bref, ce processus prend les valeurs de notre raster original et recalcule de nouvelles valeurs pour un raster cible avec une résolution et une origine personnalisées.

<!--toDo: jn-->
<!-- consider if adding this new figure makes sense -->




Il existe plusieurs méthodes pour estimer les valeurs d'un raster avec différentes résolutions/origines, comme le montre la figure \@ref(fig:resampl).
Ces méthodes comprennent :

- Plus proche voisin - attribue la valeur de la cellule la plus proche du raster original à la cellule du raster cible.
Cette méthode est rapide et convient généralement aux raster de catégories.
- Interpolation bilinéaire - affecte une moyenne pondérée des quatre cellules les plus proches de l'image originale à la cellule de l'image cible (Figure \@ref(fig:bilinear)). La méthode la plus rapide pour les rasters continus
- Interpolation cubique - utilise les valeurs des 16 cellules les plus proches de la trame d'origine pour déterminer la valeur de la cellule de sortie, en appliquant des fonctions polynomiales du troisième ordre. Elle est aussi utilisée pour les raster continus. Elle permet d'obtenir une surface plus lissée que l'interpolation bilinéaire, mais elle est également plus exigeante en termes de calcul.
- Interpolation par spline cubique - utilise également les valeurs des 16 cellules les plus proches de la trame d'origine pour déterminer la valeur de la cellule de sortie, mais applique des splines cubiques (fonctions polynomiales du troisième ordre par morceaux) pour obtenir les résultats. Elle est utilisée pour les trames continues
- Rééchantillonnage par fenêtré de Lanczos - utilise les valeurs des 36 cellules les plus proches de la trame d'origine pour déterminer la valeur de la cellule de sortie. Il est tilisé pour les raster continues^ [Une explication plus détaillée de cette méthode peut être trouvée sur https://gis.stackexchange.com/a/14361/20955.
]

Les explications ci-dessus mettent en évidence le fait que seul le rééchantillonnage par *voisin le plus proche* est adapté aux rasters contenant des catégories, alors que toutes les méthodes peuvent être utilisées (avec des résultats différents) pour les matrices continues.
En outre, chaque méthode successive nécessite plus de temps de traitement.

Pour appliquer le rééchantillonnage, le package **terra** fournit une fonction `resample()`.
Elle accepte un raster d'entrée (`x`), un raster avec des propriétés spatiales cibles (`y`), et une méthode de rééchantillonnage (`method`).

Nous avons besoin d'un raster avec des propriétés spatiales cibles pour voir comment la fonction `resample()` fonctionne.
Pour cet exemple, nous créons `target_rast`, mais vous utiliserez souvent un objet raster déjà existant.


```r
target_rast = rast(xmin = 794600, xmax = 798200, 
                   ymin = 8931800, ymax = 8935400,
                   resolution = 150, crs = "EPSG:32717")
```

Ensuite, nous devons fournir nos deux objets rasters comme deux premiers arguments et l'une des méthodes de rééchantillonnage décrites ci-dessus.


```r
dem_resampl = resample(dem, y = target_rast, method = "bilinear")
```

La figure \@ref(fig:resampl) montre une comparaison de différentes méthodes de rééchantillonnage sur l'objet `dem`.

<div class="figure" style="text-align: center">
<img src="05-geometry-operations_files/figure-html/resampl-1.png" alt="Comparaison visuelle du raster d'entré et de cinq méthodes de rééchantillonnage différentes." width="100%" />
<p class="caption">(\#fig:resampl)Comparaison visuelle du raster d'entré et de cinq méthodes de rééchantillonnage différentes.</p>
</div>

Comme vous le verrez dans la section \@ref(reproj-ras), la reprojection de raster est un cas particulier de rééchantillonnage lorsque notre raster cible a un CRS différent de la trame d'origine.

<!--jn:toDo-->
<!-- decide -->
<!-- should we mention gdalUtils or gdalUtilities? -->
<!-- gdalUtils - https://cran.r-project.org/web/packages/gdalUtils/index.html - we mentioned it in geocompr 1; however it seems abandoned -->
<!-- gdalUtilities - https://cran.r-project.org/web/packages/gdalUtilities/index.html -->
<!-- also - add some reference to GDAL functions! -->
\index{GDAL}
\BeginKnitrBlock{rmdnote}<div class="rmdnote">La plupart des opérations géométriques dans **terra** sont conviviales, plutôt rapides, et fonctionnent sur de grands objets rasters.
Cependant, il peut y avoir des cas où **terra** n´est pas le plus performant, que ce soit pour des objets rasters étendus ou pour de nombreux fichiers rasters, et où des alternatives doivent être envisagées.

Les alternatives les plus établies sont fournies par la bibliothèque GDAL.
Elle contient plusieurs fonctions utilitaires, dont :

- `gdalinfo` - liste diverses informations sur un fichier raster, y compris sa résolution, son CRS, sa boîte de délimitation, et plus encore.
- `gdal_translate` - convertit les données raster entre différents formats de fichiers.
- `gdal_rasterize` - Convertit les données vectorielles en fichiers raster.
- `gdalwarp` - permet le mosaïquage, le rééchantillonnage, le recadrage et la reprojection de données matricielles.

Toutes les fonctions ci-dessus sont écrites en C++, mais peuvent être appelées dans R en utilisant le paquet **gdalUtilities**.
Il est important de noter que toutes ces fonctions attendent un chemin de fichier raster en entrée et retournent souvent leur sortie sous forme de fichier raster (par exemple, `gdalUtilities::gdal_translate("mon_fichier.tif", "nouveau_fichier.tif", t_srs = "EPSG:4326")`).
Ceci est très différent de l´approche habituelle de **terra**, qui attend des objets `SpatRaster` en entrée.</div>\EndKnitrBlock{rmdnote}

## Exercises


E1. Générer et représenter des versions simplifiées de l'ensemble de données `nz`.
Expérimentez avec différentes valeurs de `keep` (allant de 0,5 à 0,00005) pour `ms_simplify()` et `dTolerance` (de 100 à 100 000) pour `st_simplify()`.

- À partir de quelle valeur la forme du résultat commence-t-elle à se dégrader pour chaque méthode, rendant la Nouvelle-Zélande méconnaissable ?
- Avancé : Qu'est-ce qui est différent dans le type de géométrie des résultats de `st_simplify()` par rapport au type de géométrie de `ms_simplify()` ? Quels problèmes cela crée-t-il et comment peut-on les résoudre ?



E2. Dans le premier exercice du chapitre Opérations sur les données spatiales, il a été établi que la région de Canterbury comptait 70 des 101 points les plus élevés de Nouvelle-Zélande. 
En utilisant `st_buffer()`, combien de points dans `nz_height` sont à moins de 100 km de Canterbury ?



E3. Trouvez le centroïde géographique de la Nouvelle-Zélande. 
A quelle distance se trouve-t-il du centroïde géographique de Canterbury ?



E4. La plupart des cartes du monde sont orientées du nord vers le haut.
Une carte du monde orientée vers le sud pourrait être créée par une réflexion (une des transformations affines non mentionnées dans ce chapitre) de la géométrie de l'objet `world`.
Comment faire ?
Astuce : vous devez utiliser un vecteur à deux éléments pour cette transformation.
 Bonus : créez une carte de votre pays à l'envers.



E5. Sélectionnez le point dans `p` qui est contenu dans `x` *et* `y`.

- En utilisant les opérateurs de sélection de base.
- En utilisant un objet intermédiaire créé avec `st_intersection()`\index{vector!intersection}.





E6. Calculez la longueur des limites des États américains en mètres.
Quel État a la frontière la plus longue et quel État a la plus courte ?
Indice : La fonction `st_length` calcule la longueur d'une géométrie `LINESTRING` ou `MULTILINESTRING`.



E7. Lire le fichier srtm.tif dans R (`srtm = rast(system.file("raster/srtm.tif", package = "spDataLarge"))`).
Ce raster a une résolution de 0.00083 par 0.00083 degrés. 
Changez sa résolution à 0,01 par 0,01 degrés en utilisant toutes les méthodes disponibles dans le paquet **terra**.
Visualisez les résultats.
Pouvez-vous remarquer des différences entre les résultats de ces différentes méthodes de rééchantillonnage ?