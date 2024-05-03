# Géotraitements {#spatial-operations}

## Prérequis {-}

- Ce chapitre nécessité les mêmes paquets que ceux utilisés dans le chapitre \@ref(attr): 


```r
library(sf)
library(terra)
library(dplyr)
library(spData)
```

- Vous devrez également charger deux jeux de données pour cette section \@ref(spatial-ras)


```r
elev = rast(system.file("raster/elev.tif", package = "spData"))
grain = rast(system.file("raster/grain.tif", package = "spData"))
```

## Introduction

Les opérations spatiales, y compris les jointures spatiales entre les ensembles de données vectorielles et les opérations locales et focales sur les ensembles de données raster, constituent une partie essentielle de la géocomputation\index{geocomputation}.
Ce chapitre montre comment les objets spatiaux peuvent être modifiés d'une multitude de façons en fonction de leur emplacement et de leur forme.
De nombreuses opérations spatiales ont un équivalent non spatial (par exemple via leurs attributs), de sorte que des concepts tels que la sélection et la jonction de jeux de données démontrés dans le chapitre précédent sont applicables ici.
Cela est particulièrement vrai pour les opérations *vectorielles* : La section \@ref(vector-attribute-manipulation) sur la manipulation des tables attributaires fournit la base pour comprendre son équivalent spatial, à savoir la sélection spatial (traitée dans la section \@ref(spatial-subsetting)).
La jointure spatiale (section \@ref(spatial-joining)) et l'agrégation (section \@ref(spatial-aggr)) ont également des contreparties non spatiales, traitées dans le chapitre précédent.

Les opérations spatiales diffèrent toutefois des opérations non spatiales à plusieurs égards :
Les jointures spatiales, par exemple, peuvent être effectuées de plusieurs manières --- y compris la mise en correspondance d'entités qui se croisent ou se trouvent à une certaine distance de l'ensemble de données cible --- alors que les jointures de table attributaire abordées dans la section \@ref(vector-attribute-joining) du chapitre précédent ne peuvent être effectuées que d'une seule manière (sauf lorsqu'on utilise des jointures floues, comme décrit dans la documentation du paquet [**fuzzyjoin**](https://cran.r-project.org/package=fuzzyjoin)).
Les différents *types* de relations spatiales entre objets comme les superpositions/intersections   et les objets disjoints, sont décrits dans la section \@ref(topological-relations).
\index{spatial operations}
Un autre aspect unique des objets spatiaux est la distance : tous les objets spatiaux sont liés par l'espace et les calculs de distance peuvent être utilisés pour explorer la force de cette relation, comme décrit dans le contexte des données vectorielles à la section \@ref(relations-distance).

Les opérations spatiales sur les rasters comprennent la sélection --- traité dans la section \@ref(spatial-raster-subsetting) --- et la fusion de plusieurs " tuiles " raster en un seul objet, comme le montre la section \@ref(merging-rasters).
*L'algèbre de raster* couvre une gamme d'opérations qui modifient les valeurs des cellules, avec ou sans référence aux valeurs des cellules environnantes.
Le concept d'algèbre de raster vital pour de nombreuses applications,  est présenté dans la section \@ref(map-algebra) ; les opérations d'algèbre de raster locales, focales et zonales sont traitées respectivement dans les sections \@ref(local-operations), \@ref(focal-operations) et \@ref(zonal-operations). Les opérations d'algèbre globales, qui génèrent des statistiques synthétiques représentant l'ensemble d'un jeu de données raster, et les calculs de distance sur les données raster, sont abordés dans la section \@ref(global-operations-and-distances).
Dans la dernière section avant les exercices (\@ref(merging-rasters)), le processus de fusion de deux ensembles de données raster est abordé et démontré à l'aide d'un exemple reproductible.

\BeginKnitrBlock{rmdnote}<div class="rmdnote">Il est important de noter que les opérations spatiales qui utilisent deux objets spatiaux reposent sur le fait que les deux objets ont le même système  de coordonnées de référence, un sujet qui a été introduit dans la section \@ref(crs-intro) et qui sera traité plus en profondeur dans le chapitre \@ref(reproj-geo-data).</div>\EndKnitrBlock{rmdnote}

## Géotraitements sur des données vectorielles {#spatial-vec}

Cette section fournit une vue d'ensemble des opérations spatiales sur les données géographiques vectorielles représentées sous forme de *simple features* du package **sf**.
La section \@ref(spatial-ras) présente les opérations spatiales sur les ensembles de données raster à l'aide des classes et des fonctions du paquet **terra**.

### Sélection spatiale

La sélection spatial est le processus qui consiste à prendre un objet spatial et à renvoyer un nouvel objet contenant uniquement les caractéristiques en relation dans l'espace à un autre objet.
De manière analogue à la  *sélection d'attributs* (traité dans la section \@ref(vector-attribute-subsetting)), des sélection de jeux de données `sf` peuvent être créés avec l'opérateur de crochets (`[`) en utilisant la syntaxe `x[y, , op = st_intersects]`, où `x` est un objet `sf` à partir duquel un sous-ensemble de lignes sera retourné, `y` est l'objet de sous-ensemble et `, op = st_intersects` est un argument optionnel qui spécifie la relation topologique (également connue sous le nom de prédicat binaire) utilisée pour faire la sélection.
La relation topologique par défaut utilisée lorsqu'un argument `op` n'est pas fourni est `st_intersects()` : la commande `x[y, ]` est identique à `x[y, , op = st_intersects]` montrée ci-dessus mais pas à `x[y, , op = st_disjoint]` (la signification de ces relations topologiques et des autres est décrite dans la section suivante).
La fonction `filter()` du **tidyverse**\index{tidyverse (package)} peut également être utilisée mais cette approche est plus verbeuse, comme nous le verrons dans les exemples ci-dessous.
\index{vector!subsetting}
\index{spatial!subsetting}

To demonstrate spatial subsetting, we will use the `nz` and `nz_height` datasets in the **spData** package, which contain geographic data on the 16 main regions and 101 highest points in New Zealand, respectively (Figure \@ref(fig:nz-subset)), in a projected coordinate system.
The following code chunk creates an object representing Canterbury, then uses spatial subsetting to return all high points in the region:


```r
canterbury = nz %>% filter(Name == "Canterbury")
canterbury_height = nz_height[canterbury, ]
```

<div class="figure" style="text-align: center">
<img src="04-spatial-operations_files/figure-html/nz-subset-1.png" alt="Exemple de sélection spatiale avec des triangles rouges représentant 101 points hauts en Nouvelle-Zélande, regroupés près de la région centrale de Canterbury (à gauche). Les points dans la région de Canterbury ont été créés avec l'opérateur de sélection `[` (surligné en gris, à droite)." width="100%" />
<p class="caption">(\#fig:nz-subset)Exemple de sélection spatiale avec des triangles rouges représentant 101 points hauts en Nouvelle-Zélande, regroupés près de la région centrale de Canterbury (à gauche). Les points dans la région de Canterbury ont été créés avec l'opérateur de sélection `[` (surligné en gris, à droite).</p>
</div>

Comme pour la sélection d'attributs, la commande `x[y, ]` (équivalente à `nz_height[canterbury, ]`) sélectionne les caractéristiques d'une *cible* `x` en utilisant le contenu d'un objet *source* `y`.
Cependant, au lieu que `y` soit un vecteur de classe `logical` ou `integer`, pour la sélection spatiale, `x` et `y` doivent être des objets géographiques.
Plus précisément, les objets utilisés pour la sélection spatiale de cette manière doivent avoir la classe `sf` ou `sfc` : `nz` et `nz_height` sont tous deux des jeux de données vectorielles géographiques et ont la classe `sf`, et le résultat de l'opération renvoie un autre objet `sf` représentant les caractéristiques de l'objet cible `nz_height` qui intersectent (dans ce cas, les points hauts qui sont situés dans) la région de `canterbury`.

Diverses *relations topologiques* peuvent être utilisées pour le sélection spatiale. Elles déterminent le type de relation spatiale que les caractéristiques de l'objet cible doivent avoir avec l'objet de  sélection.
Il peut s'agir de *touches* (touche), *crosses* (croisse) ou *within* (dedans), comme nous le verrons bientôt dans la section \@ref(topological-relations). 
Le paramètre par défaut `st_intersects` est une relation topologique 'attrape-tout' qui retournera les éléments de la cible qui *touchent*, *croissent* ou sont *within* (dedans) l'objet source 'sélectionnant'.
Comme indiqué ci-dessus, d'autres opérateurs spatiaux peuvent être spécifiés avec l'argument `op =`, comme le montre la commande suivante qui renvoie l'opposé de `st_intersects()`, les points qui ne sont pas en intersection avec `Canterbury` (voir la section \@ref(topological-relations)) :


```r
nz_height[canterbury, , op = st_disjoint]
```

\BeginKnitrBlock{rmdnote}<div class="rmdnote">Notez que l´argument vide --- dénoté par `, ,` --- dans l´extrait de code précédent est inclus pour mettre en évidence `op`, le troisième argument dans `[` pour les objets `sf`.
On peut l´utiliser pour modifier l´opération de sélection de plusieurs façons.
`nz_height[canterbury, 2, op = st_disjoint]`, par exemple, retourne les mêmes lignes mais n´inclut que la deuxième colonne d´attributs (voir ``sf:::`[.sf`` et le `?sf`` pour plus de détails).</div>\EndKnitrBlock{rmdnote}

Pour de nombreuses applications, c'est tout ce que vous aurez besoin de savoir sur les sélections spatiales avec les données vectorielles !
Si vous êtes impatient d'en savoir plus sur les relations topologiques, au-delà de `st_intersects()` et `st_disjoint()`, passez à la section suivante (\@ref(topological-relations)).
Si vous êtes intéressé par les détails, y compris les autres façons de faire des sélections, c'est par ici.

Une autre façon d'effectuer une sélection spatiale est d'utiliser les objets retournés par les opérateurs topologiques.
Ces objets peuvent être utiles en soi, par exemple lors de l'exploration du réseau de relations entre des régions contiguës, mais ils peuvent également être utilisés pour sélectionner comme le montre le morceau de code ci-dessous :


```r
sel_sgbp = st_intersects(x = nz_height, y = canterbury)
class(sel_sgbp)
#> [1] "sgbp" "list"
sel_sgbp
#> Sparse geometry binary predicate list of length 101, where the
#> predicate was `intersects'
#> first 10 elements:
#>  1: (empty)
#>  2: (empty)
#>  3: (empty)
#>  4: (empty)
#>  5: 1
#>  6: 1
#>  7: 1
#>  8: 1
#>  9: 1
#>  10: 1
sel_logical = lengths(sel_sgbp) > 0
canterbury_height2 = nz_height[sel_logical, ]
```

Le code ci-dessus crée un objet de classe `sgbp` (un prédicat binaire de géométrie "creuse", une liste de longueur `x` dans l'opération spatiale) et le convertit ensuite en un vecteur logique `sel_logical` (contenant seulement les valeurs `TRUE` et `FALSE`, quelque chose qui peut aussi être utilisé par la fonction filtre de **dplyr**).
\index{binary predicate|seealso {topological relations}}
La fonction `lengths()` identifie les éléments de `nz_height` qui ont une intersection avec *tout* objet de `y`.
Dans ce cas, 1 est la plus grande valeur possible, mais pour des opérations plus complexes, on peut utiliser la méthode pour sélectionner uniquement les caractéristiques qui ont une intersection avec, par exemple, 2 caractéristiques ou plus de l'objet source.

\BeginKnitrBlock{rmdnote}<div class="rmdnote">Note : une autre façon de retourner une sortie logique est de mettre `sparse = FALSE` (ce qui signifie retourner une matrice dense et non une matrice 'creuse') dans des opérateurs tels que `st_intersects()`. La commande `st_intersects(x = nz_height, y = canterbury, sparse = FALSE)[, 1]`, par exemple, retournerait une sortie identique à `sel_logical`.
Note : la solution impliquant les objets `sgbp` est cependant plus généralisable, car elle fonctionne pour les opérations *many-to-many* et a des besoins en mémoire plus faibles.</div>\EndKnitrBlock{rmdnote}

Le même résultat peut être obtenu avec la fonction de **sf** `st_filter()` qui a été [créée](https://github.com/r-spatial/sf/issues/1148) pour augmenter la compatibilité entre les objets `sf` et les manipulation de données de **dplyr** :


```r
canterbury_height3 = nz_height |>
  st_filter(y = canterbury, .predicate = st_intersects)
```

<!--toDo:jn-->
<!-- fix pipes -->



A ce stade, il y a trois versions identiques (à l'exception des noms de lignes) de `canterbury_height`, une créée en utilisant l'opérateur `[`, une créée via un objet de sélection intermédiaire, et une autre utilisant la fonction de commodité de **sf** `st_filter()`.
<!-- RL: commented out for now as old. Todo: if we ever update that vignette uncomment the next line. -->
<!-- To explore spatial subsetting in more detail, see the supplementary vignettes on `subsetting` and [`tidyverse-pitfalls`](https://geocompr.github.io/geocompkg/articles/) on the [geocompkg website](https://geocompr.github.io/geocompkg/articles/). -->
La section suivante explore différents types de relations spatiales, également connues sous le nom de prédicats binaires, qui peuvent être utilisées pour identifier si deux éléments sont spatialement liés ou non.

### Relations topologiques

Les relations topologiques décrivent les relations spatiales entre les objets.
Les "relations topologiques binaires", pour leur donner leur nom complet, sont des énoncés logiques (en ce sens que la réponse ne peut être que `VRAI` ou `FAUX`) sur les relations spatiales entre deux objets définis par des ensembles ordonnés de points (formant typiquement des points, des lignes et des polygones) en deux dimensions ou plus [@egenhofer_mathematical_1990].
Cela peut sembler plutôt abstrait et, en effet, la définition et la classification des relations topologiques reposent sur des fondements mathématiques publiés pour la première fois sous forme de livre en 1966 [@spanier_algebraic_1995], le domaine de la topologie algébrique se poursuivant au 21^e^ siècle [@dieck_algebraic_2008].

Malgré leur origine mathématique, les relations topologiques peuvent être comprises intuitivement en se référant à des visualisations de fonctions couramment utilisées qui testent les types courants de relations spatiales.
La figure \@ref(fig:relations) montre une variété de paires géométriques et leurs relations associées.
Les troisième et quatrième paires de la figure \@ref(fig:relations) (de gauche à droite puis vers le bas) montrent que, pour certaines relations, l'ordre est important : alors que les relations *equals*, *intersects*, *crosses*, *touches* et *overlaps* sont symétriques, ce qui signifie que si `function(x, y)` est vraie, `function(y, x)` le sera aussi, les relations dans lesquelles l'ordre des géométries est important, comme *contains* et *within*, ne le sont pas.
Remarquez que chaque paire de géométries possède une chaîne "DE-9IM" telle que FF2F11212, décrite dans la section suivante.
\index{topological relations}

<div class="figure" style="text-align: center">
<img src="04-spatial-operations_files/figure-html/relations-1.png" alt="Relations topologiques entre géométries vectorielles, inspirées des figures 1 et 2 d'Egenhofer et Herring (1990). Les relations pour lesquelles la fonction(x, y) est vraie sont imprimées pour chaque paire de géométries, x étant représenté en rose et y en bleu. La nature de la relation spatiale pour chaque paire est décrite par la chaîne de caractères du Dimensionally Extended 9-Intersection Model." width="100%" />
<p class="caption">(\#fig:relations)Relations topologiques entre géométries vectorielles, inspirées des figures 1 et 2 d'Egenhofer et Herring (1990). Les relations pour lesquelles la fonction(x, y) est vraie sont imprimées pour chaque paire de géométries, x étant représenté en rose et y en bleu. La nature de la relation spatiale pour chaque paire est décrite par la chaîne de caractères du Dimensionally Extended 9-Intersection Model.</p>
</div>

Dans `sf`, les fonctions testant les différents types de relations topologiques sont appelées binary predicates", comme décrit dans la vignette *Manipulating Simple Feature Geometries*, qui peut être consultée avec la commande [`vignette("sf3")`](https://r-spatial.github.io/sf/articles/sf3.html), et dans la page d'aide [`?geos_binary_pred`](https://r-spatial.github.io/sf/reference/geos_binary_ops.html).
Pour voir comment les relations topologiques fonctionnent en pratique, créons un exemple simple et reproductible, en nous appuyant sur les relations illustrées dans la Figure \@ref(fig:relations) et en consolidant les connaissances sur la représentation des géométries vectorielles acquises dans un chapitre précédent (Section \@ref(geometry)).
Notez que pour créer des données tabulaires représentant les coordonnées (x et y) des sommets du polygone, nous utilisons la fonction R de base `cbind()` pour créer une matrice représentant les points de coordonnées, un `POLYGON`, et enfin un objet `sfc`, comme décrit au chapitre \@ref(spatial-class)) :


```r
polygon_matrix = cbind(
  x = c(0, 0, 1, 1,   0),
  y = c(0, 1, 1, 0.5, 0)
)
polygon_sfc = st_sfc(st_polygon(list(polygon_matrix)))
```

Nous allons créer des géométries supplémentaires pour démontrer les relations spatiales à l'aide des commandes suivantes qui, lorsqu'elles sont tracées sur le polygone créé ci-dessus, se rapportent les unes aux autres dans l'espace, comme le montre la Figure \@ref(fig:relation-objects).
Notez l'utilisation de la fonction `st_as_sf()` et de l'argument `coords` pour convertir efficacement un tableau de données contenant des colonnes représentant des coordonnées en un objet `sf` contenant des points :


```r
line_sfc = st_sfc(st_linestring(cbind(
  x = c(0.4, 1),
  y = c(0.2, 0.5)
)))
# créer des points
point_df = data.frame(
  x = c(0.2, 0.7, 0.4),
  y = c(0.1, 0.2, 0.8)
)
point_sf = st_as_sf(point_df, coords = c("x", "y"))
```

<div class="figure" style="text-align: center">
<img src="04-spatial-operations_files/figure-html/relation-objects-1.png" alt="Points (`point_df` 1 à 3), ligne et polygones arrangés pour illustrer les relations topologiques." width="50%" />
<p class="caption">(\#fig:relation-objects)Points (`point_df` 1 à 3), ligne et polygones arrangés pour illustrer les relations topologiques.</p>
</div>

Une première question simple pourrait être : quels sont les points de `point_sf` en intersection avec le polygone `polygon_sfc` ?
On peut répondre à cette question par inspection (les points 1 et 3 sont respectivement en contact et à l'intérieur du polygone).
On peut répondre à cette question avec le prédicat spatial `st_intersects()` comme suit:


```r
st_intersects(point_sf, polygon_sfc)
#> Sparse geometry binary predicate... `intersects'
#>  1: 1
#>  2: (empty)
#>  3: 1
```

Le résultat devrait correspondre à votre intuition :
des résultats positifs (`1`) sont retournés pour le premier et le troisième point, et un résultat négatif (représenté par un vecteur vide "(empty)") pour le deuxième  en dehors de la frontière du polygone.
Ce qui peut être inattendu, c'est que le résultat se présente sous la forme d'une liste de vecteurs.
Cette sortie *matrice creuse* n'enregistre une relation que si elle existe, ce qui réduit les besoins en mémoire des opérations topologiques sur les objets avec de nombreuses entités.
Comme nous l'avons vu dans la section précédente, une *matrice dense* composée de valeurs `TRUE` ou `FALSE` est retournée lorsque `sparse = FALSE` :


```r
st_intersects(point_sf, polygon_sfc, sparse = FALSE)
#>       [,1]
#> [1,]  TRUE
#> [2,] FALSE
#> [3,]  TRUE
```

Dans la sortie ci-dessus, chaque ligne représente un élément dans l'objet cible (l'argument `x`) et chaque colonne représente un élément dans l'objet de sélection (`y`). 
Dans ce cas, il n'y a qu'un seul élément dans l'objet `y` `polygon_sfc`, donc le résultat, qui peut être utilisé pour la sélection comme nous l'avons vu dans la section \@ref(spatial-subsetting), n'a qu'une seule colonne.

`st_intersects()` renvoie `TRUE` même dans les cas où les éléments se touchent juste : *intersects* est une opération topologique "fourre-tout" qui identifie de nombreux types de relations spatiales, comme l'illustre la figure \@ref(fig:relations).
Il y a des questions plus restrictives, par exemple  : quels sont les points situés à l'intérieur du polygone, et quelles sont les caractéristiques qui sont sur ou qui contiennent une frontière partagée avec `y` ?
On peut répondre à ces questions de la manière suivante (résultats non montrés) :


```r
st_within(point_sf, polygon_sfc)
st_touches(point_sf, polygon_sfc)
```

Notez que bien que le premier point *touche* la limite du polygone, il n'est pas à l'intérieur de celui-ci ; le troisième point est à l'intérieur du polygone mais ne touche aucune partie de sa frontière.
L'opposé de `st_intersects()` est `st_disjoint()`, qui retourne uniquement les objets qui n'ont aucun rapport spatial avec l'objet sélectionné (ici `[, 1]` convertit le résultat en vecteur) :


```r
st_disjoint(point_sf, polygon_sfc, sparse = FALSE)[, 1]
#> [1] FALSE  TRUE FALSE
```

La fonction `st_is_within_distance()` détecte les éléments qui  touchent *presque* l'objet de sélection. La fonction a un argument supplémentaire `dist`.
Il peut être utilisé pour définir la distance à laquelle les objets cibles doivent se trouver avant d'être sélectionnés.
Remarquez que bien que le point 2 soit à plus de 0,2 unités de distance du sommet le plus proche de `polygon_sfc`, il est quand même sélectionné lorsque la distance est fixée à 0,2. 
En effet, la distance est mesurée par rapport à l'arête la plus proche, dans ce cas la partie du polygone qui se trouve directement au-dessus du point 2 dans la figure \@ref(fig:relation-objets).
(Vous pouvez vérifier que la distance réelle entre le point 2 et le polygone est de 0,13 avec la commande `st_distance(point_sf, polygon_sfc)`).
Le prédicat spatial binaire "is within distance" (es à distance de) est démontré dans l'extrait de code ci-dessous, dont les résultats montrent que chaque point est à moins de 0,2 unité du polygone :


```r
st_is_within_distance(point_sf, polygon_sfc, dist = 0.2, sparse = FALSE)[, 1]
#> [1] TRUE TRUE TRUE
```




\BeginKnitrBlock{rmdnote}<div class="rmdnote">Les fonctions de calcul des relations topologiques utilisent des indices spatiaux pour accélérer considérablement les performances des requêtes spatiales.
Elles y parviennent en utilisant l´algorithme *Sort-Tile-Recursive* (STR).
La fonction `st_join`, mentionnée dans la section suivante, utilise également l´indexation spatiale. 
Vous pouvez en savoir plus à l´adresse suivante https://www.r-spatial.org/r/2017/06/22/spatial-index.html.</div>\EndKnitrBlock{rmdnote}







### Les chaines DE-9IM

Les prédicats binaires présentés dans la section précédente reposent sur le modèle *Dimensionally Extended 9-Intersection Model* (DE-9IM).
Comme le suggère son nom cryptique, ce n'est pas un sujet facile.
Y consacrer du temps peut être utile afin d'améliorer notre compréhension des relations spatiales.
En outre, les utilisations avancées de DE-9IM incluent la création de prédicats spatiaux personnalisés.
Ce modèle était à l'origine intitulé " DE + 9IM " par ses inventeurs, en référence à la " dimension des intersections des limites, des intérieurs et des extérieurs de deux entités " [@clementini_comparison_1995], mais il est désormais désigné par DE-9IM [@shen_classification_2018].
<!-- The model's workings can be demonstrated with reference to two intersecting polygons, as illustrated in Figure \@ref(fig:de-9im). -->



Pour démontrer le fonctionnement des chaînes DE-9IM, examinons les différentes façons dont la première paire de géométries peut-être reliée dans la figure \@ref(fig:relations).
La figure \@ref(fig:de9imgg) illustre le modèle à 9 intersections (9IM).  Elle montre les intersections entre chaque combinaison possible entre l'intérieur, la limite et l'extérieur de chaque objet. Chaque composant du premier objet `x` est disposé en colonnes et que chaque composant de `y` est disposé en lignes, un graphique à facettes est créé avec les intersections entre chaque élément mises en évidence.

<div class="figure" style="text-align: center">
<img src="04-spatial-operations_files/figure-html/de9imgg-1.png" alt="Illustration du fonctionnement du Modèle Dimensionnel Étendu à 9 Intersections (DE-9IM). Les couleurs qui ne figurent pas dans la légende représentent le chevauchement entre les différentes composantes. Les lignes épaisses mettent en évidence les intersections bidimensionnelles, par exemple entre la limite de l'objet x et l'intérieur de l'objet y, illustrées dans la facette supérieure du milieu." width="100%" />
<p class="caption">(\#fig:de9imgg)Illustration du fonctionnement du Modèle Dimensionnel Étendu à 9 Intersections (DE-9IM). Les couleurs qui ne figurent pas dans la légende représentent le chevauchement entre les différentes composantes. Les lignes épaisses mettent en évidence les intersections bidimensionnelles, par exemple entre la limite de l'objet x et l'intérieur de l'objet y, illustrées dans la facette supérieure du milieu.</p>
</div>

Les chaînes DE-9IM sont dérivées de la dimension de chaque type de relation.
Dans ce cas, les intersections rouges de la figure \@ref(fig:de9imgg) ont des dimensions de 0 (points), 1 (lignes) et 2 (polygones), comme le montre le tableau \@ref(tab:de9emtable).



Table: (\#tab:de9emtable)Tableau montrant les relations entre les intérieurs, les limites et les extérieurs des géométries x et y.

|              |Intérieur (x) |Limite (x) |Extérieur (x) |
|:-------------|:-------------|:----------|:-------------|
|Intérieur (y) |2             |1          |2             |
|Limite (y)    |1             |1          |1             |
|Extérieur (y) |2             |1          |2             |



En aplatissant cette matrice "ligne par ligne" (c'est-à-dire en concaténant la première ligne, puis la deuxième, puis la troisième), on obtient la chaîne `212111212`.
Un autre exemple va permettre d'expliciter ce système :
la relation représentée sur la figure \@ref(fig:relations) (la troisième paire de polygones dans la troisième colonne et la première ligne) peut être définie dans le système DE-9IM comme suit :

- Les intersections entre l'*intérieur* du grand objet `x` et l'intérieur, la limite et l'extérieur de `y` ont des dimensions respectives de 2, 1 et 2.
- Les intersections entre la *frontière* du grand objet `x` et l'intérieur, la frontière et l'extérieur de `y` ont des dimensions respectives de F, F et 1, où "F" signifie "faux", les objets sont disjoints.
- Les intersections entre l'*extérieur* de `x` et l'intérieur, la limite et l'extérieur de `y` ont des dimensions respectives de F, F et 2 : l'extérieur du plus grand objet ne touche pas l'intérieur ou la limite de `y`, mais l'extérieur du plus petit et du plus grand objet couvre la même surface.

Ces trois composants, une fois concaténés, créent la chaîne `212`, `FF1`, et `FF2`.
C'est le même résultat que celui obtenu par la fonction `st_relate()` (voir le code source de ce chapitre pour voir comment les autres géométries de la figure \@ref(fig:relations) ont été créées) :


```r
xy2sfc = function(x, y) st_sfc(st_polygon(list(cbind(x, y))))
x = xy2sfc(x = c(0, 0, 1, 1,   0), y = c(0, 1, 1, 0.5, 0))
y = xy2sfc(x = c(0.7, 0.7, 0.9, 0.7), y = c(0.8, 0.5, 0.5, 0.8))
st_relate(x, y)
#>      [,1]       
#> [1,] "212FF1FF2"
```

La compréhension des chaînes DE-9IM permet de développer de nouveaux prédicats spatiaux binaires.
La page d'aide `?st_relate` contient des définitions de fonctions pour les relations "reine" et "tour" dans lesquelles les polygones partagent une frontière ou seulement un point, respectivement.
Les relations "reine" signifient que les relations "frontière-frontière" (la cellule de la deuxième colonne et de la deuxième ligne de la table \@ref(tab:de9emtable), ou le cinquième élément de la chaîne DE-9IM) ne doivent pas être vides, ce qui correspond au *pattern*  `F***T****`, tandis que pour les relations "tour", le même élément doit être 1 (ce qui signifie une intersection linéaire).
Ces relations sont implémentées comme suit :


```r
st_queen = function(x, y) st_relate(x, y, pattern = "F***T****")
st_rook = function(x, y) st_relate(x, y, pattern = "F***1****")
```

A partir de l'objet `x` créé précédemment, nous pouvons utiliser les fonctions nouvellement créées pour trouver quels éléments de la grille sont une 'reine' et une 'tour' par rapport à la case centrale de la grille comme suit :


```r
grid = st_make_grid(x, n = 3)
grid_sf = st_sf(grid)
grid_sf$queens = lengths(st_queen(grid, grid[5])) > 0
plot(grid, col = grid_sf$queens)
grid_sf$rooks = lengths(st_rook(grid, grid[5])) > 0
plot(grid, col = grid_sf$rooks)
```


```
#> -- tmap v3 code detected --
#> [v3->v4] tm_polygons(): migrate the argument(s) related to the scale of the visual variable 'fill', namely 'palette' (rename to 'values') to 'fill.scale = tm_scale(<HERE>)'
#> [v3->v4] tm_polygons(): use 'fill' for the fill color of polygons/symbols (instead of 'col'), and 'col' for the outlines (instead of 'border.col')
```

<div class="figure" style="text-align: center">
<img src="04-spatial-operations_files/figure-html/queens-1.png" alt="Démonstration de prédicats spatiaux binaires personnalisés permettant de trouver les relations 'reine' (à gauche) et 'tour' (à droite) par rapport à la case centrale dans une grille à 9 géométries." width="100%" />
<p class="caption">(\#fig:queens)Démonstration de prédicats spatiaux binaires personnalisés permettant de trouver les relations 'reine' (à gauche) et 'tour' (à droite) par rapport à la case centrale dans une grille à 9 géométries.</p>
</div>


<!-- Another of a custom binary spatial predicate is 'overlapping lines' which detects lines that overlap for some or all of another line's geometry. -->
<!-- This can be implemented as follows, with the pattern signifying that the intersection between the two line interiors must be a line: -->



### Jointure spatiale 

La jointure de deux jeux de données non spatiales repose sur une variable "clé" partagée, comme décrit dans la section \@ref(vector-attribute-joining).
La jointure de données spatiales applique le même concept, mais s'appuie sur les relations spatiales, décrites dans la section précédente.
Comme pour les données attributaires, la jointure ajoute de nouvelles colonnes à l'objet cible (l'argument `x` dans les fonctions de jointure), à partir d'un objet source (`y`).
\index{join!spatial}
\index{spatial!join}

Le processus est illustré par l'exemple suivant : imaginez que vous disposez de dix points répartis au hasard sur la surface de la Terre et que vous demandez, pour les points qui se trouvent sur la terre ferme, dans quels pays se trouvent-ils ?
La mise en œuvre de cette idée dans un [exemple reproductible] (https://github.com/Robinlovelace/geocompr/blob/main/code/04-spatial-join.R) renforcera vos compétences en matière de traitement des données géographiques et vous montrera comment fonctionnent les jointures spatiales.
Le point de départ consiste à créer des points dispersés de manière aléatoire sur la surface de la Terre :


```r
set.seed(2018) # définir la seed pour la reproductibilité
(bb = st_bbox(world)) # les limites de la terre
#>   xmin   ymin   xmax   ymax 
#> -180.0  -89.9  180.0   83.6
random_df = data.frame(
  x = runif(n = 10, min = bb[1], max = bb[3]),
  y = runif(n = 10, min = bb[2], max = bb[4])
)
random_points = random_df |> 
  st_as_sf(coords = c("x", "y")) |> # définir les coordonnés
  st_set_crs("EPSG:4326") # définir le CRS
```

Le scénario illustré dans la Figure \@ref(fig:spatial-join) montre que l'objet `random_points` (en haut à gauche) n'a pas d'attributs, alors que le `world` (en haut à droite) a des attributs, y compris les noms de pays indiqués pour un échantillon de pays dans la légende.
Les jointures spatiales sont implémentées avec `st_join()`, comme illustré dans l'extrait de code ci-dessous.
La sortie est l'objet `random_joined` qui est illustré dans la Figure \@ref(fig:spatial-join) (en bas à gauche).
Avant de créer l'ensemble de données jointes, nous utilisons la sélection spatiale pour créer `world_random`, qui contient uniquement les pays qui contiennent des points aléatoires, afin de vérifier que le nombre de noms de pays retournés dans l'ensemble de données jointes doit être de quatre (cf. le panneau supérieur droit de la Figure \@ref(fig:spatial-join)).


```r
world_random = world[random_points, ]
nrow(world_random)
#> [1] 4
random_joined = st_join(random_points, world["name_long"])
```

<div class="figure" style="text-align: center">
<img src="04-spatial-operations_files/figure-html/spatial-join-1.png" alt="Illustration d'une jointure spatiale. Une nouvelle variable attributaire est ajoutée aux points aléatoires (en haut à gauche) de l'objet monde source (en haut à droite), ce qui donne les données représentées dans le dernier panneau." width="100%" />
<p class="caption">(\#fig:spatial-join)Illustration d'une jointure spatiale. Une nouvelle variable attributaire est ajoutée aux points aléatoires (en haut à gauche) de l'objet monde source (en haut à droite), ce qui donne les données représentées dans le dernier panneau.</p>
</div>

Par défaut, `st_join()` effectue une jointure à gauche (*left join*), ce qui signifie que le résultat est un objet contenant toutes les lignes de `x`, y compris les lignes sans correspondance dans `y` (voir la section \@ref(vector-attribute-joining)), mais il peut également effectuer des jointures internes en définissant l'argument `left = FALSE`.
Comme pour les sélections spatiales, l'opérateur topologique par défaut utilisé par `st_join()` est `st_intersects()`, qui peut être modifié en définissant l'argument `join` (cf. `?st_join` pour plus de détails).
L'exemple ci-dessus montre l'ajout d'une colonne d'une couche de polygones à une couche de points, mais la même approche fonctionne indépendamment des types de géométrie.
Dans de tels cas, par exemple lorsque `x` contient des polygones, dont chacun correspond à plusieurs objets dans `y`, les jointures spatiales résulteront en des caractéristiques dupliquées, crée une nouvelle ligne pour chaque correspondance dans `y`.

<!-- Idea: demonstrate what happens when there are multiple matches with reprex (low priority, RL: 2021-12) -->

### Jointure sans chevauchement

Parfois, deux jeux de données géographiques ne se touchent pas mais ont quand même une forte relation géographique.
Les jeux de données `cycle_hire` et `cycle_hire_osm`, présent dans le paquet **spData**, en sont un bon exemple.
Leur tracé montre qu'ils sont souvent étroitement liés mais qu'ils ne se touchent pas, comme le montre la figure \@ref(fig:cycle-hire), dont une version de base est créée avec le code suivant ci-dessous :
\index{join!non-overlapping}


```r
plot(st_geometry(cycle_hire), col = "blue")
plot(st_geometry(cycle_hire_osm), add = TRUE, pch = 3, col = "red")
```

Nous pouvons vérifier si certains points se superposent  avec `st_intersects()` :


```r
any(st_touches(cycle_hire, cycle_hire_osm, sparse = FALSE))
#> [1] FALSE
```



<div class="figure" style="text-align: center">

```{=html}
<div class="leaflet html-widget html-fill-item" id="htmlwidget-9580c84d0c32f0a22a8c" style="width:100%;height:415.296px;"></div>
<script type="application/json" data-for="htmlwidget-9580c84d0c32f0a22a8c">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addCircles","args":[[51.52916347,51.49960695,51.52128377,51.53005939,51.49313,51.51811784,51.53430039,51.52834133,51.5073853,51.50597426,51.52395143,51.52168078,51.51991453,51.52994371,51.51772703,51.52635795,51.5216612,51.51477076,51.52505093,51.52773634,51.53007835,51.5222641,51.51943538,51.51908011,51.5288338,51.52728093,51.51382102,51.52351808,51.513735,51.52915444,51.52953709,51.52469624,51.5341235,51.50173726,51.49159394,51.4973875,51.5263778,51.52127071,51.52001715,51.53099181,51.52026,51.51073687,51.522511,51.507131,51.52334476,51.51248445,51.50706909,51.52867339,51.52671796,51.52295439,51.5099923,51.52174785,51.51707521,51.52058381,51.52334672,51.52452699,51.53176825,51.52644828,51.49738251,51.4907579,51.53089041,51.50946212,51.52522753,51.51795029,51.51882555,51.52059681,51.5262363,51.53136059,51.5154186,51.52352001,51.52572618,51.48591714,51.53219984,51.52559505,51.52341837,51.52486887,51.50069361,51.52025302,51.514274,51.50963938,51.51593725,51.50064702,51.48947903,51.51646835,51.51858757,51.5262503,51.53301907,51.49368637,51.49889832,51.53440868,51.49506109,51.5208417,51.53095071,51.49792478,51.52554222,51.51457763,51.49043573,51.51155322,51.51340693,51.50472376,51.51159481,51.51552971,51.51410514,51.52600832,51.49812559,51.51563144,51.53304322,51.5100172,51.51580998,51.49646288,51.52451738,51.51423368,51.51449962,51.49288067,51.49582705,51.52589324,51.51573534,51.51891348,51.52111369,51.52836014,51.49654462,51.50069491,51.51782144,51.51700801,51.49536226,51.5118973,51.50950627,51.53300545,51.52364804,51.50136494,51.504904,51.52326004,51.51196176,51.49087475,51.51227622,51.49488108,51.52096262,51.51530805,51.49372451,51.4968865,51.48894022,51.50074359,51.48836528,51.51494305,51.49211134,51.48478899,51.49705603,51.51213691,51.49217002,51.51183419,51.50379168,51.49586666,51.49443626,51.50039792,51.49085368,51.51461995,51.50663341,51.49234577,51.51760685,51.4931848,51.515607,51.517932,51.50185512,51.49395092,51.50040123,51.51474612,51.52784273,51.4916156,51.49121192,51.50486,51.512529,51.52174384,51.51791921,51.49616092,51.48985626,51.5129118,51.49941247,51.52202903,51.52071513,51.48805753,51.51733558,51.49247977,51.5165179,51.53166681,51.48997562,51.50311799,51.51251523,51.50581776,51.50462759,51.50724437,51.50368837,51.50556905,51.51048489,51.51492456,51.5225965,51.51236389,51.51821864,51.52659961,51.518144,51.518154,51.50135267,51.52505151,51.49358391,51.51681444,51.49464523,51.50658458,51.50274025,51.52683806,51.51906932,51.49094565,51.51615461,51.48971651,51.49016361,51.49066456,51.49481649,51.50275704,51.49148474,51.51476963,51.50935342,51.50844614,51.52891573,51.50742485,51.50654321,51.50669284,51.49396755,51.51501025,51.50777049,51.53450449,51.49571828,51.51838043,51.5084448,51.5233534,51.52443845,51.50545935,51.52200801,51.49096258,51.51611887,51.4853572,51.52285301,51.530052,51.50645179,51.49859784,51.489932,51.518908,51.5046364,51.53404294,51.53051587,51.52557531,51.51362054,51.52248185,51.50143293,51.50494561,51.5136846,51.5134891,51.49874469,51.51422502,51.52644342,51.51595344,51.50102668,51.49206037,51.49320445,51.50082346,51.4863434,51.53583617,51.50144456,51.50613324,51.49671237,51.52004497,51.50930161,51.50315739,51.5034938,51.51862243,51.490083,51.49580589,51.51906446,51.49914063,51.5272947,51.52367314,51.49236962,51.50923022,51.51678023,51.48483991,51.49888404,51.49815779,51.488226,51.50029631,51.49363156,51.49612799,51.50227992,51.49398524,51.50501351,51.51475963,51.466907,51.50295379,51.51211869,51.48677988,51.51816295,51.50990837,51.49907558,51.50963123,51.4908679,51.51918144,51.53088935,51.51734403,51.50088934,51.52536703,51.49768448,51.49679128,51.51004801,51.52702563,51.49357351,51.49785559,51.526293,51.523196,51.49652013,51.50908747,51.53266186,51.53114,51.53095,51.53589283,51.51641749,51.52085887,51.49334336,51.50860544,51.505044,51.51196803,51.504942,51.51108452,51.51175646,51.5333196,51.51444134,51.50810309,51.53692216,51.528246,51.48802358,51.50013942,51.51217033,51.51196,51.5017154373867,51.529423,51.486965,51.49459148,51.506767,51.486575,51.494412,51.520994,51.51643491,51.49782999,51.49675303,51.50391972,51.536264,51.5291212008901,51.519656,51.530344,51.5109192966489,51.5173721,51.50024195,51.520205,51.49775,51.515208,51.49980661,51.50402793,51.50194596,51.49188409,51.50535447,51.49559291,51.51431171,51.50686435,51.51953043,51.50935171,51.51310333,51.496481,51.51352755,51.49369988,51.51070161,51.51013066,51.521776,51.51066202,51.49942855,51.51733427,51.524826,51.492462,51.51809,51.51348,51.502319,51.52261762,51.517703,51.528187,51.52289229,51.51689296,51.50204238,51.49418566,51.512303,51.51222,51.51994326,51.493146,51.519968,51.49337264,51.488105,51.485821,51.504043,51.504044,51.49456127,51.490491,51.533379,51.493072,51.51397065,51.499917,51.48902,51.534474,51.496957,51.524564,51.488852,51.51824,51.488124,51.5338,51.483145,51.495656,51.510101,51.515256,51.52568,51.52388,51.528936,51.493381,51.50623,51.511088,51.515975,51.504719,51.505697,51.508447,51.487679,51.49447,51.538071,51.542138,51.504749,51.535179,51.516196,51.516,51.541603,51.534776,51.51417,51.53558,51.534464,51.523538,51.521564,51.513757,51.530535,51.533283,51.529452,51.496137,51.498125,51.499041,51.489096,51.49109,51.521889,51.527152,51.5128711,51.487129,51.509843,51.51328,51.528828,51.531127,51.525645,51.511811,51.506946,51.513074,51.508622,51.522507,51.524677,51.50196,51.528169,51.52512,51.527058,51.519265,51.522561,51.502635,51.520893,51.496454,51.520398,51.517475,51.528692,51.519362,51.517842,51.509303,51.511066,51.532091,51.5142228,51.516204,51.500088,51.5112,51.532513,51.528224,51.518811,51.526041,51.534137,51.525941,51.51549,51.511654,51.504714,51.503143,51.503802,51.507326,51.486892,51.508896,51.530326,51.50357,51.528222,51.531864,51.537349,51.51793,51.508981,51.531091,51.528302,51.506613,51.514115,51.509591,51.526153,51.539957,51.517428,51.509474,51.498386,51.49605,51.521564,51.503447,51.511542,51.535678,51.513548,51.502661,51.51601,51.493978,51.501391,51.511624,51.518369,51.51746,51.499286,51.509943,51.521905,51.509158,51.51616,51.53213,51.503083,51.506256,51.539099,51.485587,51.53356,51.497304,51.528869,51.527607,51.511246,51.48692917,51.497622,51.51244,51.490645,51.50964,51.52959,51.487196,51.53256,51.506093,51.5171,51.531066,51.513875,51.493267,51.472817,51.473471,51.494499,51.485743,51.481747,51.514767,51.472993,51.477839,51.538792,51.520331,51.504199,51.468814,51.491093,51.46512358,51.48256792,51.50646524,51.46925984,51.50403821,51.47817208,51.4768851,51.48102131,51.47107905,51.47625965,51.4737636,51.47518024,51.46086446,51.50748124,51.46193072,51.4729184,51.47761941,51.48438657,51.4687905,51.46881971,51.45995384,51.47073264,51.4795017,51.47053858,51.48959104,51.49434708,51.48606206,51.46706414,51.45787019,51.46663393,51.48357068,51.47286577,51.49824168,51.46916161,51.5190427,51.48373225,51.50173215,51.49886563,51.50035306,51.46904022,51.48180515,51.51687069,51.48089844,51.51148696,51.47084722,51.48267821,51.48294452,51.46841875,51.4996806,51.47729232,51.46437067,51.49087074,51.51632095,51.48498496,51.51323001,51.474376,51.46108367,51.49610093,51.5015946,51.49422354,51.47614939,51.46718562,51.475089,51.4646884,51.46517078,51.53546778,51.46348914,51.47787084,51.46866929,51.46231278,51.47453545,51.47768469,51.47303687,51.48810829,51.46506424,51.45475251,51.46067005,51.48814438,51.49760804,51.46095151,51.45922541,51.47047503,51.47946386,51.54211855,51.464786,51.53638435,51.48728535,51.53639219,51.53908372,51.5366541,51.47696496,51.47287627,51.52868155,51.51505991,51.45682071,51.45971528,51.50215353,51.49021762,51.46161068,51.46321128,51.47439218,51.48335692,51.51854104,51.54100708,51.47311696,51.51563007,51.51542791,51.53658514,51.53571683,51.53642464,51.48724429,51.53603947,51.52458353,51.46822047,51.45799126,51.47732253,51.47505096,51.47569809,51.46199911,51.47893931,51.49208492,51.47727637,51.50630441,51.50542628,51.46230566,51.46489445,51.47993289,51.47514228,51.48176572,51.51092871,51.5129814,51.51510818,51.46079243,51.46745485,51.47816972,51.4795738,51.48796408,51.53727795,51.53932857,51.4710956,51.51612862,51.45816465,51.49263658,51.5129006,51.48512191,51.47515398,51.48795853,51.51787005,51.52456169,51.52526975,51.48321729,51.50070305,51.52059714,51.46239255,51.46760141,51.45752945,51.45705988,51.46134382,51.473611,51.491026,51.509224,51.47250956,51.511891,51.470131,51.496664,51.460333,51.4619230679],[-0.109970527,-0.197574246,-0.084605692,-0.120973687,-0.156876,-0.144228881,-0.1680743,-0.170134484,-0.09644075100000001,-0.092754157,-0.122502346,-0.130431727,-0.136039674,-0.123616824,-0.127854211,-0.125979294,-0.109006325,-0.12221963,-0.131161087,-0.135273468,-0.13884627,-0.114079481,-0.119123345,-0.124678402,-0.132250369,-0.11829517,-0.107927706,-0.143613641,-0.193487,-0.093421615,-0.08335332300000001,-0.084439283,-0.129386874,-0.184980612,-0.192369256,-0.197245586,-0.07813092100000001,-0.0755789,-0.08391116799999999,-0.093903825,-0.157183945,-0.144165239,-0.162298,-0.06691,-0.183846408,-0.099141408,-0.145904427,-0.08745937600000001,-0.104298194,-0.094934859,-0.143495266,-0.09447507199999999,-0.086685542,-0.154701411,-0.120202614,-0.079248081,-0.114329032,-0.172190727,-0.089446947,-0.106323685,-0.089782579,-0.124749274,-0.13518856,-0.108657431,-0.108028472,-0.116688468,-0.134407652,-0.117069978,-0.098850915,-0.108340165,-0.08848618799999999,-0.124469948,-0.105480698,-0.144083893,-0.124121774,-0.099489485,-0.102091246,-0.141327271,-0.111257,-0.131510949,-0.111778348,-0.078600401,-0.115156562,-0.079684557,-0.132053392,-0.123509611,-0.139174593,-0.111014912,-0.100440521,-0.109025404,-0.08581448899999999,-0.09734016199999999,-0.078505384,-0.183834706,-0.138231303,-0.158264483,-0.122806861,-0.0929401,-0.076793375,-0.192538767,-0.07712132200000001,-0.190240716,-0.147301667,-0.096317627,-0.132102166,-0.132328837,-0.172528678,-0.157275636,-0.105270275,-0.183289032,-0.158963647,-0.07353765399999999,-0.141423695,-0.114934001,-0.13547809,-0.090847761,-0.093080779,-0.156166631,-0.078869751,-0.104724625,-0.150905245,-0.094524319,-0.096496865,-0.09388536,-0.185296516,-0.137043852,-0.07545948199999999,-0.136792671,-0.074754872,-0.191462381,-0.06797,-0.104708922,-0.097441687,-0.153319609,-0.157436972,-0.117974901,-0.085634242,-0.147203711,-0.198286569,-0.161203828,-0.111435796,-0.202759212,-0.129361842,-0.11614642,-0.138364847,-0.110683213,-0.168917077,-0.201554966,-0.101536865,-0.174292825,-0.11282408,-0.191933711,-0.092921165,-0.193068385,-0.196170309,-0.137841333,-0.131773845,-0.141334487,-0.121328408,-0.167894973,-0.183118788,-0.183716959,-0.159237081,-0.147624377,-0.195455928,-0.165164288,-0.108068155,-0.186753859,-0.173715911,-0.113001,-0.115163,-0.08111889999999999,-0.188098863,-0.140947636,-0.141923621,-0.153645496,-0.152317537,-0.165842551,-0.14521173,-0.140741432,-0.175810943,-0.178433004,-0.164393768,-0.109914711,-0.132845681,-0.153520935,-0.133201961,-0.100186337,-0.091773776,-0.106237501,-0.098497684,-0.111606696,-0.082989638,-0.06607803700000001,-0.161113413,-0.06954201,-0.100791005,-0.112432615,-0.06275,-0.062697,-0.153194766,-0.166304359,-0.165101392,-0.151926305,-0.158105512,-0.199004026,-0.149569201,-0.130504336,-0.088285377,-0.181190899,-0.08242239899999999,-0.170194408,-0.19039362,-0.166485083,-0.13045856,-0.155349725,-0.090220911,-0.188129731,-0.196422,-0.131961389,-0.115480888,-0.134621209,-0.123179697,-0.103137426,-0.17873226,-0.112753217,-0.130699733,-0.106992706,-0.110889274,-0.073438925,-0.067176443,-0.175116099,-0.138019439,-0.10569204,-0.151359288,-0.139625122,-0.128585022,-0.142207481,-0.099994052,-0.168314,-0.170279555,-0.096191134,-0.162727,-0.079249,-0.116542278,-0.08637971699999999,-0.106408455,-0.179592915,-0.116764211,-0.154907218,-0.178656971,-0.123247648,-0.135580879,-0.191351186,-0.103132904,-0.08066008299999999,-0.109256828,-0.169249375,-0.180246101,-0.132224622,-0.144132875,-0.089740764,-0.122492418,-0.156285395,-0.110699309,-0.114686385,-0.20528437,-0.09217644699999999,-0.084985356,-0.191496313,-0.07962099,-0.176645823,-0.162418,-0.127575233,-0.059642081,-0.112031483,-0.174653609,-0.128377673,-0.147478734,-0.151296092,-0.175488803,-0.138089062,-0.165471605,-0.209494128,-0.135635511,-0.092762704,-0.190603326,-0.106000855,-0.074189225,-0.136928582,-0.172729559,-0.148105415,-0.216573,-0.158456089,-0.16209757,-0.115853961,-0.135025698,-0.187842717,-0.08566631600000001,-0.119047563,-0.116911864,-0.140485596,-0.176770502,-0.138072691,-0.08315935200000001,-0.153463612,-0.141943703,-0.093913472,-0.138846453,-0.08854277100000001,-0.139956043,-0.081608045,-0.07395500000000001,-0.083067,-0.101384068,-0.129697889,-0.099981142,-0.086016,-0.085603,-0.160854428,-0.179135079,-0.089887855,-0.194757949,-0.193764092,-0.115851,-0.120718759,-0.115533,-0.197524944,-0.119643424,-0.111781191,-0.087587447,-0.12602103,-0.150181444,-0.10102611,-0.166878535,-0.113936001,-0.150481272,-0.142783033,-0.1798541843891,-0.097122,-0.116625,-0.134234258,-0.123702,-0.117286,-0.173881,-0.139016,-0.124332175,-0.135440826,-0.138733562,-0.11342628,-0.133952,-0.171185284853,-0.132339,-0.100168,-0.1511263847351,-0.1642075,-0.15934065,-0.174593,-0.10988,-0.117863,-0.176415994,-0.11386435,-0.194392952,-0.125674815,-0.113656543,-0.179077626,-0.200838199,-0.150666888,-0.13577731,-0.14744969,-0.13121385,-0.192404,-0.130110822,-0.121394101,-0.121723604,-0.155757901,-0.068856,-0.142345694,-0.179702476,-0.103604248,-0.176268,-0.159919,-0.163609,-0.17977,-0.200742,-0.071653961,-0.154106,-0.075375,-0.171681991,-0.158249929,-0.184400221,-0.18267094,-0.159988,-0.160785,-0.170704337,-0.099828,-0.169774,-0.09968067,-0.110121,-0.149004,-0.105312,-0.104778,-0.15393398,-0.149186,-0.139159,-0.129925,-0.09294031,-0.174554,-0.17524,-0.122203,-0.173894,-0.116279,-0.105593,-0.11655,-0.120903,-0.118677,-0.113134,-0.114605,-0.211358,-0.058641,-0.055312,-0.06507599999999999,-0.055894,-0.007542,-0.02296,-0.057159,-0.053177,-0.063531,-0.07054199999999999,-0.055167,-0.021582,-0.014409,-0.144664,-0.145393,-0.057544,-0.03338,-0.029138,-0.038775,-0.138853,-0.071881,-0.052099,-0.08248999999999999,-0.07634100000000001,-0.030556,-0.022694,-0.020467,-0.025492,-0.028155,-0.027616,-0.019355,-0.011457,-0.020157,-0.009205,-0.018716,-0.04667,-0.058005,-0.0389866,-0.009001,-0.02377,-0.047784,-0.013258,-0.048017,-0.06954299999999999,-0.025626,-0.058681,-0.064094,-0.06500599999999999,-0.041378,-0.03562,-0.016251,-0.018703,-0.015578,-0.025296,-0.021345,-0.054883,-0.022702,-0.051394,-0.009506000000000001,-0.026768,-0.07585500000000001,-0.059091,-0.074431,-0.090075,-0.025996,-0.053558,-0.06142,-0.055656,-0.155525,-0.211316,-0.014438,-0.033085,-0.037471,-0.011662,-0.047218,-0.037366,-0.036017,-0.013475,-0.179668,-0.014293,-0.008428,-0.215808,-0.145827,-0.170983,-0.012413,-0.042744,-0.020068,-0.069743,-0.066035,-0.147154,-0.067937,-0.00699,-0.075901,-0.144466,-0.142844,-0.033828,-0.204666,-0.102208,-0.145246,-0.107987,-0.002275,-0.107913,-0.104193,-0.039264,-0.016233,-0.056667,-0.062546,-0.005659,-0.021596,-0.0985,-0.127554,-0.205991,-0.205921,-0.043371,-0.12335,-0.009152,-0.117619,-0.063386,-0.224103,-0.18697,-0.08298999999999999,-0.017676,-0.218337,-0.141728,-0.18119,-0.09315,-0.022793,-0.047548,-0.057133,-0.09305099999999999,-0.102996299,-0.125978,-0.19096,-0.014582,-0.08497,-0.0801,-0.179369,-0.16862,-0.2242237,-0.18377,-0.11934,-0.117774,-0.21985,-0.199783,-0.20782,-0.228188,-0.223616,-0.124642,-0.225787,-0.133972,-0.116493,-0.138535,-0.163667,-0.210941,-0.210279,-0.216493,-0.157788279,-0.172078187,-0.208486599,-0.141812513,-0.217400093,-0.144690541,-0.215895601,-0.209973497,-0.207842908,-0.193254007,-0.197010096,-0.167160736,-0.187427294,-0.205535908,-0.180791784,-0.132102704,-0.149551631,-0.20481514,-0.158230901,-0.184318843,-0.190184054,-0.126994068,-0.141770709,-0.163041605,-0.209378594,-0.215804559,-0.214428378,-0.193502076,-0.174691623,-0.169821175,-0.202038682,-0.148059277,-0.117495865,-0.174485792,-0.204764421,-0.223852256,-0.100292412,-0.137424571,-0.217515071,-0.19627483,-0.18027465,-0.213872396,-0.183853573,-0.218190203,-0.17070367,-0.117661574,-0.219346128,-0.199135704,-0.221791552,-0.16478637,-0.174619404,-0.206029743,-0.202608612,-0.167919869,-0.211593602,-0.155442787,-0.191722864,-0.208158259,-0.222293381,-0.236769936,-0.1232585,-0.152248582,-0.201968,-0.173656546,-0.18038939,-0.11619105,-0.182126248,-0.126874471,-0.146544642,-0.211468596,-0.170210533,-0.170329317,-0.214749808,-0.22660621,-0.163750945,-0.195197203,-0.198735357,-0.222456468,-0.21145598,-0.20066766,-0.180884959,-0.152130083,-0.195777222,-0.028941601,-0.215618902,-0.102757578,-0.217995921,-0.112721065,-0.070329419,-0.07023031,-0.174347066,-0.176267008,-0.06555032099999999,-0.10534448,-0.202802098,-0.212145939,-0.083632928,-0.215087092,-0.21614583,-0.215550761,-0.163347594,-0.216305546,-0.034903714,-0.14326094,-0.137235175,-0.049067243,-0.02356501,-0.07588568599999999,-0.060291813,-0.054162264,-0.205279052,-0.026262677,-0.058631453,-0.190346493,-0.184806157,-0.138748723,-0.150908371,-0.20587627,-0.206240805,-0.208485293,-0.229116862,-0.189210466,-0.087262995,-0.150817316,-0.175407201,-0.17302926,-0.19411695,-0.187278987,-0.185273723,-0.214594781,-0.219486603,-0.208565479,-0.212607684,-0.172293499,-0.18243547,-0.17903854,-0.161765173,-0.079201849,-0.07428467499999999,-0.157850096,-0.120909408,-0.20600248,-0.234094148,-0.214762686,-0.174971902,-0.159169801,-0.187404506,-0.201005397,-0.165668686,-0.163795009,-0.211860644,-0.129698963,-0.032566533,-0.16829214,-0.20682737,-0.192165613,-0.200806304,-0.159322467,-0.191803,-0.209121,-0.216016,-0.122831913,-0.107349,-0.20464,-0.223868,-0.167029,-0.165297856693],10,null,null,{"interactive":true,"className":"","stroke":true,"color":"#03F","weight":5,"opacity":0.5,"fill":true,"fillColor":"#03F","fillOpacity":0.2},null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]},{"method":"addCircles","args":[[51.52912521362305,51.53401565551758,51.52729034423828,51.52582931518555,51.53001403808594,51.52594375610352,51.52664947509766,51.52085876464844,51.51682281494141,51.51473617553711,51.49416351318359,51.50061798095703,51.51780319213867,51.51329803466797,51.51405334472656,51.51224899291992,51.4882698059082,51.50081253051758,51.52448272705078,51.50056076049805,51.50337982177734,51.49507141113281,51.50747680664062,51.51416015625,51.51640701293945,51.51606750488281,51.50171279907227,51.50201034545898,51.49529647827148,51.51899337768555,51.51996994018555,51.51194000244141,51.52170944213867,51.51566314697266,51.51248550415039,51.497802734375,51.5099983215332,51.49079895019531,51.49155807495117,51.51231002807617,51.49150466918945,51.51769638061523,51.51214218139648,51.4901008605957,51.51478576660156,51.49358367919922,51.49903106689453,51.49145126342773,51.50217819213867,51.5250129699707,51.52245712280273,51.52287673950195,51.52532196044922,51.52196884155273,51.5164794921875,51.5145263671875,51.51681137084961,51.53588104248047,51.51811218261719,51.52334213256836,51.52565383911133,51.51673126220703,51.52835464477539,51.52250289916992,51.53087615966797,51.51727676391602,51.51192092895508,51.53435897827148,51.51531219482422,51.52731704711914,51.52647399902344,51.52053451538086,51.53308486938477,51.52260208129883,51.53691482543945,51.51469421386719,51.52555847167969,51.52023696899414,51.51406478881836,51.52341842651367,51.5207405090332,51.52516174316406,51.51856994628906,51.52337646484375,51.50148010253906,51.52470016479492,51.49089813232422,51.51391220092773,51.51790237426758,51.5232048034668,51.51870346069336,51.51491165161133,51.52326965332031,51.52390670776367,51.53091430664062,51.53261947631836,51.52450561523438,51.50960540771484,51.5159797668457,51.50600814819336,51.50090026855469,51.50944519042969,51.51948928833008,51.51821517944336,51.53089904785156,51.52178573608398,51.52351760864258,51.5269660949707,51.50862121582031,51.50478363037109,51.50655364990234,51.50936508178711,51.51110458374023,51.50312423706055,51.50276184082031,51.5063591003418,51.50270462036133,51.50302886962891,51.52287292480469,51.50719833374023,51.50673675537109,51.53334808349609,51.53291702270508,51.51734924316406,51.52548599243164,51.52022933959961,51.5236930847168,51.52767562866211,51.53001022338867,51.52880477905273,51.52627182006836,51.50925064086914,51.51859283447266,51.53307342529297,51.5191764831543,51.52675247192383,51.51288986206055,51.51224899291992,51.51071548461914,51.5118522644043,51.50993728637695,51.50992584228516,51.51190948486328,51.5057373046875,51.50535202026367,51.51589202880859,51.51157760620117,51.51702117919922,51.5191650390625,51.52096939086914,51.52167510986328,51.53197479248047,51.49435043334961,51.52944183349609,51.51446151733398,51.51610946655273,51.51904678344727,51.52439880371094,51.52506637573242,51.5257682800293,51.52671051025391,51.52104187011719,51.49912261962891,51.49487686157227,51.4920654296875,51.49595642089844,51.49212265014648,51.53086471557617,51.50924682617188,51.52347564697266,51.51543426513672,51.50062561035156,51.49895477294922,51.50366973876953,51.49112319946289,51.52059555053711,51.49082183837891,51.49552536010742,51.50117111206055,51.49394226074219,51.51353073120117,51.49674224853516,51.49799728393555,51.50186538696289,51.49391937255859,51.49413681030273,51.49236297607422,51.507080078125,51.49700927734375,51.49364471435547,51.49655914306641,51.49946212768555,51.49881744384766,51.50128936767578,51.49464797973633,51.49319076538086,51.49240493774414,51.49787902832031,51.49010848999023,51.48990249633789,51.49111938476562,51.48974227905273,51.48804092407227,51.49075317382812,51.50144577026367,51.51197814941406,51.51037216186523,51.49218368530273,51.51503753662109,51.50650405883789,51.51491928100586,51.5050163269043,51.51174926757812,51.51435470581055,51.506591796875,51.50767517089844,51.50740432739258,51.50800323486328,51.50844955444336,51.50907516479492,51.50968933105469,51.51758193969727,51.51374435424805,51.51470565795898,51.51447296142578,51.53133010864258,51.52838516235352,51.52645874023438,51.5267333984375,51.53044891357422,51.52826690673828,51.52888107299805,51.5291633605957,51.5278205871582,51.53181838989258,51.53451538085938,51.53160095214844,51.53448486328125,51.51836013793945,51.51002502441406,51.52993011474609,51.52622222900391,51.50460433959961,51.49643325805664,51.51578903198242,51.51375961303711,51.51592254638672,51.50947189331055,51.49486923217773,51.49581527709961,51.49396896362305,51.49613571166992,51.52635192871094,51.48826599121094,51.49322128295898,51.49234771728516,51.49362945556641,51.48994445800781,51.50579833984375,51.48537826538086,51.48810958862305,51.48983383178711,51.4981575012207,51.51644515991211,51.53055953979492,51.48485565185547,51.51791381835938,51.51488494873047,51.51557922363281,51.51791763305664,51.51560592651367,51.51712799072266,51.51890182495117,51.51425933837891,51.51215744018555,51.52226638793945,51.49868392944336,51.49961090087891,51.50031661987305,51.49734878540039,51.4966926574707,51.50032806396484,51.49629974365234,51.50531768798828,51.50063705444336,51.49807739257812,51.51834106445312,51.49370193481445,51.49331665039062,51.4958381652832,51.50140762329102,51.51897048950195,51.50468826293945,51.52130126953125,51.49744033813477,51.49075317382812,51.48680877685547,51.48625946044922,51.49372863769531,51.48944473266602,51.49286270141602,51.48483657836914,51.49043655395508,51.49571228027344,51.48894882202148,51.48603057861328,51.4908561706543,51.50009155273438,51.51220703125,51.50178909301758,51.51728439331055,51.50294494628906,51.51157760620117,51.51779556274414,51.51667022705078,51.49860000610352,51.49674987792969,51.51644897460938,51.51970672607422,51.49796676635742,51.50197982788086,51.52048873901367,51.49689483642578,51.51947021484375,51.52097702026367,51.51348495483398,51.51299285888672,51.52950668334961,51.51092147827148,51.51813125610352,51.50935363769531,51.51070404052734,51.50686645507812,51.49713134765625,51.4997673034668,51.50234985351562,51.53605651855469,51.5134391784668,51.51768112182617,51.51806259155273,51.51361083984375,51.51009750366211,51.51062393188477,51.49942779541016,51.49175262451172,51.51727676391602,51.5125617980957,51.5200080871582,51.50026702880859,51.52265167236328,51.51991271972656,51.52025604248047,51.51988983154297,51.51507186889648,51.50411224365234,51.52243041992188,51.52361297607422,51.5246696472168,51.52186584472656,51.50592422485352,51.49340438842773,51.49310684204102,51.53213119506836,51.53573226928711,51.51632308959961,51.53445053100586,51.53379821777344,51.49440765380859,51.52646636962891,51.53155136108398,51.53030395507812,51.53110504150391,51.51013946533203,51.5091667175293,51.50912857055664,51.51172637939453,51.51735305786133,51.50736999511719,51.50645065307617,51.50137329101562,51.50635147094727,51.53342437744141,51.51915740966797,51.5282096862793,51.50611114501953,51.50505065917969,51.50486373901367,51.50396728515625,51.50378799438477,51.51714706420898,51.51609420776367,51.51236724853516,51.52350616455078,51.51633834838867,51.51122665405273,51.51162338256836,51.51406478881836,51.51754379272461,51.52091598510742,51.52569198608398,51.52255630493164,51.53409194946289,51.52899169921875,51.53515625,51.53250503540039,51.53331756591797,51.53050994873047,51.5294303894043,51.52883529663086,51.52602005004883,51.49992370605469,51.52867889404297,51.51124572753906,51.51420593261719,51.51108169555664,51.51920318603516,51.5088996887207,51.5390510559082,51.50929641723633,51.48754119873047,51.5162467956543,51.53112411499023,51.53093719482422,51.50707626342773,51.49605560302734,51.52891540527344,51.52882385253906,51.52891540527344,51.52885055541992,51.52882766723633,51.52885818481445,51.52874755859375,51.52875518798828,51.49691772460938,51.50173568725586,51.45926666259766,51.46195983886719,51.46232986450195,51.4643669128418,51.46463394165039,51.46493530273438,51.51509857177734,51.49330902099609,51.49103164672852,51.54203796386719,51.53428268432617,51.53427886962891,51.53406143188477,51.53406143188477,51.4881706237793,51.51298141479492,51.53634262084961,51.49882507324219,51.50459289550781,51.52611923217773,51.5133056640625,51.5135383605957,51.50959014892578,51.51366806030273,51.52515029907227,51.5159912109375,51.46982192993164,51.46978759765625,51.46990585327148,51.46994018554688,51.48889923095703,51.52953720092773,51.51374435424805,51.52818298339844,51.50558471679688,51.50534057617188,51.5048713684082,51.50386810302734,51.51820755004883,51.51516342163086,51.52561187744141,51.52750015258789,51.52717208862305,51.49206924438477,51.47304916381836,51.51152038574219,51.52157974243164,51.51946258544922,51.53804016113281,51.49589157104492,51.50403213500977,51.50404739379883,51.50413131713867,51.52257537841797,51.50473785400391,51.50476455688477,51.50480651855469,51.50482940673828,51.52818298339844,51.47513198852539,51.53466033935547,51.52820587158203,51.54682540893555,51.54629898071289,51.50030136108398,51.53590393066406,51.52753829956055,51.540283203125,51.53598022460938,51.51189804077148,51.49393081665039,51.48147964477539,51.54103469848633,51.54169464111328,51.48582458496094,51.46757125854492,51.53876876831055,51.51645278930664,51.50397491455078,51.47622680664062,51.47941589355469,51.47994232177734,51.48358535766602,51.48390197753906,51.50931167602539,51.50931930541992,51.50932312011719,51.50933837890625,51.50934219360352,51.50983428955078,51.50983810424805,51.50997161865234,51.50997543334961,51.54087066650391,51.54308319091797,51.53842926025391,51.52247619628906,51.48233413696289,51.51785659790039,51.51787567138672,51.51789855957031,51.51802825927734,51.49266815185547,51.50943374633789,51.50895309448242],[-0.09338779747486115,-0.129309207201004,-0.1182352006435394,-0.09083600342273712,-0.1210571974515915,-0.1038272008299828,-0.1123251020908356,-0.08988530188798904,-0.1582169979810715,-0.1221619993448257,-0.1825312972068787,-0.09451589733362198,-0.09637469798326492,-0.07670540362596512,-0.07358500361442566,-0.06940989941358566,-0.1356287002563477,-0.0898251011967659,-0.1588800996541977,-0.07856839895248413,-0.07957950234413147,-0.08594369888305664,-0.09640560299158096,-0.08061859756708145,-0.07962480187416077,-0.08212079852819443,-0.1848890036344528,-0.1843793988227844,-0.1852709054946899,-0.1247294023633003,-0.1358640938997269,-0.1207367032766342,-0.1304273009300232,-0.1322115063667297,-0.1331615000963211,-0.08164320141077042,-0.157212495803833,-0.196125403046608,-0.186663806438446,-0.1597979962825775,-0.1924726963043213,-0.1280096024274826,-0.1620880961418152,-0.1904651969671249,-0.1652061939239502,-0.1906657963991165,-0.08559329807758331,-0.09012889862060547,-0.07422350347042084,-0.1662103980779648,-0.1548451036214828,-0.171658992767334,-0.153420701622963,-0.1513393968343735,-0.1644168943166733,-0.1582493036985397,-0.1518975049257278,-0.160705104470253,-0.1441397964954376,-0.1838506013154984,-0.1439844071865082,-0.1755494028329849,-0.1700911968946457,-0.1622716933488846,-0.1768050044775009,-0.1758725941181183,-0.174384206533432,-0.1681527942419052,-0.1470558047294617,-0.1746665984392166,-0.1721556931734085,-0.1547646969556808,-0.1726416945457458,-0.1610399037599564,-0.150157704949379,-0.1480903029441833,-0.1795047968626022,-0.1570902019739151,-0.1472848057746887,-0.175151601433754,-0.1450770050287247,-0.135173499584198,-0.1766694933176041,-0.1241701990365982,-0.1107200980186462,-0.08494079858064651,-0.1394933015108109,-0.09281720221042633,-0.1084283962845802,-0.1047310009598732,-0.108025997877121,-0.06618840247392654,-0.1204117983579636,-0.1224915981292725,-0.09387639909982681,-0.09995549917221069,-0.07924109697341919,-0.07465779781341553,-0.1692546010017395,-0.09271299839019775,-0.08347310125827789,-0.1243832036852837,-0.1191532015800476,-0.06269069761037827,-0.07854320108890533,-0.1092348992824554,-0.1083744987845421,-0.08861500024795532,-0.1937263011932373,-0.1925584971904755,-0.1989907026290894,-0.1963624060153961,-0.1974851042032242,-0.1535256057977676,-0.1494038999080658,-0.1701322048902512,-0.1552689969539642,-0.191404402256012,-0.09995950013399124,-0.1061898022890091,-0.1032897979021072,-0.1117890030145645,-0.1367686986923218,-0.1381060928106308,-0.1382132023572922,-0.1412868946790695,-0.1284389048814774,-0.1353798061609268,-0.1387432068586349,-0.1322190016508102,-0.1342364996671677,-0.151116207242012,-0.1319689005613327,-0.1391572952270508,-0.1405548006296158,-0.1305487006902695,-0.1536446958780289,-0.1575558930635452,-0.1441929936408997,-0.142809197306633,-0.1388102024793625,-0.1879072934389114,-0.1370330005884171,-0.09979409724473953,-0.105653703212738,-0.09304329752922058,-0.09291200339794159,-0.09384860098361969,-0.08823960274457932,-0.08566469699144363,-0.09442149847745895,-0.1054814979434013,-0.09292580187320709,-0.08337190002202988,-0.08765500038862228,-0.1285894960165024,-0.05971070006489754,-0.1381545066833496,-0.1311517059803009,-0.0884820967912674,-0.0781090036034584,-0.07875949889421463,-0.1120520979166031,-0.1179369017481804,-0.1321800053119659,-0.1353285014629364,-0.1383174955844879,-0.08993770182132721,-0.08472809940576553,-0.1435666978359222,-0.09880249947309494,-0.1019288972020149,-0.1001908034086227,-0.09848310053348541,-0.09712529927492142,-0.116697296500206,-0.1813008040189743,-0.1791877001523972,-0.1802363991737366,-0.1787330061197281,-0.1913225948810577,-0.1388126015663147,-0.1438243985176086,-0.1592289954423904,-0.1476075947284698,-0.1546085029840469,-0.1476241052150726,-0.1459303945302963,-0.1688434034585953,-0.1649394929409027,-0.1509203016757965,-0.152325302362442,-0.165447399020195,-0.1532440930604935,-0.1580667048692703,-0.1680227965116501,-0.1784097999334335,-0.1838334947824478,-0.1625753045082092,-0.1625996977090836,-0.1736893951892853,-0.1702129989862442,-0.1668481975793839,-0.1662604063749313,-0.1784280985593796,-0.09751179814338684,-0.08290299773216248,-0.1015529036521912,-0.1126210987567902,-0.1232742965221405,-0.1162168979644775,-0.1727204024791718,-0.1197232007980347,-0.1184678003191948,-0.1317393034696579,-0.131008505821228,-0.1346541047096252,-0.1259603053331375,-0.1318870931863785,-0.1297453045845032,-0.1314696967601776,-0.1213387995958328,-0.1354451030492783,-0.1375370025634766,-0.1414363980293274,-0.1170132011175156,-0.1010444983839989,-0.1091234982013702,-0.1042855009436607,-0.1063415035605431,-0.1047158986330032,-0.1154187023639679,-0.1099506989121437,-0.1080510020256042,-0.1142947971820831,-0.1090492978692055,-0.1098138988018036,-0.10696080327034,-0.0730184018611908,-0.1434559971094131,-0.1235487014055252,-0.1234614998102188,-0.09175200015306473,-0.1016196012496948,-0.1052607968449593,-0.1078827977180481,-0.1117516979575157,-0.1190444976091385,-0.1305709928274155,-0.1275409013032913,-0.1368478983640671,-0.1408818960189819,-0.1259883940219879,-0.1292545050382614,-0.1441729068756104,-0.1413037925958633,-0.1400548070669174,-0.132818803191185,-0.136564701795578,-0.1421748995780945,-0.1405968070030212,-0.1420411020517349,-0.1319441050291061,-0.1791722029447556,-0.1674931049346924,-0.138083204627037,-0.1879784017801285,-0.1881255954504013,-0.1831178069114685,-0.1836283057928085,-0.190236896276474,-0.08666019886732101,-0.1560654938220978,-0.2009900957345963,-0.201490193605423,-0.11407820135355,-0.1039230972528458,-0.1975031048059464,-0.1930058002471924,-0.1972882002592087,-0.2052401006221771,-0.195428803563118,-0.1058785989880562,-0.112193301320076,-0.2025800049304962,-0.2094330042600632,-0.1006769984960556,-0.198381707072258,-0.1947298943996429,-0.1918330043554306,-0.1915650963783264,-0.07889190316200256,-0.1232506036758423,-0.08454930037260056,-0.08951500058174133,-0.1062221974134445,-0.1158493012189865,-0.1222822964191437,-0.1110500991344452,-0.1150930970907211,-0.1149597987532616,-0.1106337010860443,-0.1227603033185005,-0.1110092028975487,-0.1114194020628929,-0.1243795976042747,-0.1168795973062515,-0.1139174029231071,-0.1504797041416168,-0.1797240972518921,-0.1644082069396973,-0.1585202068090439,-0.0769961029291153,-0.09002619981765747,-0.1795178949832916,-0.09621690213680267,-0.09396609663963318,-0.1242700964212418,-0.1323571056127548,-0.1350177973508835,-0.194334402680397,-0.09731210023164749,-0.1613789945840836,-0.1357111930847168,-0.1389185041189194,-0.1300491988658905,-0.1311278939247131,-0.09709309786558151,-0.1512179970741272,-0.1349456012248993,-0.1474936008453369,-0.1421763002872467,-0.1506551057100296,-0.1922765970230103,-0.1763094961643219,-0.2007306963205338,-0.133782297372818,-0.1797408014535904,-0.1540451943874359,-0.1635331958532333,-0.1934694051742554,-0.1556915044784546,-0.1216448023915291,-0.1797050982713699,-0.1256998926401138,-0.1035635992884636,-0.115013100206852,-0.09218680113554001,-0.09272850304841995,-0.07158850133419037,-0.1697838008403778,-0.1745657026767731,-0.1705185025930405,-0.1181337013840675,-0.2174990028142929,-0.04176019877195358,-0.07494830340147018,-0.03569810092449188,-0.04663209989666939,-0.07068169862031937,-0.09982819855213165,-0.09984319657087326,-0.06135139986872673,-0.06271539628505707,-0.09847539663314819,-0.1220870018005371,-0.118647001683712,-0.1735181957483292,-0.0286301001906395,-0.06618860363960266,-0.04279280081391335,-0.04798439890146255,-0.2114519029855728,-0.2242292016744614,-0.2196248024702072,-0.2059940993785858,-0.1231873035430908,-0.1457414031028748,-0.1430086046457291,-0.2059323042631149,-0.2184094041585922,-0.09320160001516342,-0.1477314978837967,-0.03735850006341934,-0.1146802976727486,-0.1159529015421867,-0.1153946965932846,-0.1138684973120689,-0.1130378022789955,-0.1834370046854019,-0.1873559057712555,-0.1909584999084473,-0.0305780004709959,-0.02917139977216721,-0.09299620240926743,-0.06898610293865204,-0.1111695989966393,-0.1079486981034279,-0.05132989957928658,-0.05524060130119324,-0.05480609834194183,-0.03731809929013252,-0.05588379874825478,-0.03340740129351616,-0.03296750038862228,-0.02815829962491989,-0.02544780075550079,-0.02760040014982224,-0.0477617010474205,-0.0358303003013134,-0.1745879054069519,-0.08740100264549255,-0.01404820010066032,-0.03362049907445908,-0.05736390128731728,-0.02137940004467964,-0.01240990031510592,-0.1416262984275818,-0.02587329968810081,-0.1168603971600533,-0.1552148014307022,-0.08595810085535049,-0.08555299788713455,-0.06689970195293427,-0.1042110025882721,-0.01334539987146854,-0.01331450045108795,-0.01330539956688881,-0.01330920029431581,-0.01334969978779554,-0.01334539987146854,-0.01334269996732473,-0.01337889954447746,-0.1738771051168442,-0.1003087982535362,-0.1808453053236008,-0.1808360069990158,-0.1753010004758835,-0.1747123003005981,-0.1738660931587219,-0.1728828996419907,-0.1054434031248093,-0.2191217988729477,-0.216540202498436,-0.02883020043373108,-0.08636900037527084,-0.08640450239181519,-0.08638259768486023,-0.08634699881076813,-0.2223017960786819,-0.06408549845218658,-0.1025628000497818,-0.1373721957206726,-0.1165795028209686,-0.04696319997310638,-0.04797070100903511,-0.1167192980647087,-0.08465509861707687,-0.1176247969269753,-0.015591099858284,-0.1208008974790573,-0.1408015936613083,-0.1407676041126251,-0.1404854953289032,-0.140521302819252,-0.105547197163105,-0.08009859919548035,-0.02040489949285984,-0.07548379898071289,-0.1117931008338928,-0.1136531010270119,-0.1129046976566315,-0.1134240031242371,-0.1165020987391472,-0.05844509974122047,-0.06955330073833466,-0.05706809833645821,-0.05791860073804855,-0.2291229963302612,-0.2147257030010223,-0.05670920014381409,-0.02237650007009506,-0.0744313970208168,-0.1446426063776016,-0.1728768050670624,-0.2174569964408875,-0.2173631936311722,-0.2174052000045776,-0.04101530089974403,-0.06754639744758606,-0.06752920150756836,-0.06778910011053085,-0.06777189671993256,-0.06979779899120331,-0.1592990010976791,-0.124966099858284,-0.06943450123071671,-0.01454740017652512,-0.01002360042184591,-0.1590677946805954,-0.1560537070035934,-0.1348813027143478,-0.02166059985756874,-0.02660050056874752,-0.1071562990546227,-0.1274670958518982,-0.1380670964717865,-0.1431670933961868,-0.1390734016895294,-0.1488102972507477,-0.2066828012466431,-0.1384492963552475,-0.1183784976601601,-0.01320760045200586,-0.1932822018861771,-0.1957414001226425,-0.1941318064928055,-0.2020470052957535,-0.1975594013929367,-0.02592330053448677,-0.02581189945340157,-0.02573350071907043,-0.02573220059275627,-0.02582200057804585,-0.02373870089650154,-0.0237748995423317,-0.02368880063295364,-0.02372509986162186,-0.01074429973959923,-0.007984300144016743,-0.01189500000327826,-0.04172470048069954,-0.1362718045711517,-0.04322149977087975,-0.04317649826407433,-0.04341059923171997,-0.04321610182523727,-0.0923537015914917,-0.002419099910184741,-0.006909300107508898],10,null,null,{"interactive":true,"className":"","stroke":true,"color":"red","weight":5,"opacity":0.5,"fill":true,"fillColor":"red","fillOpacity":0.2},null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]}],"limits":{"lat":[51.45475251,51.54682540893555],"lng":[-0.236769936,-0.002275]}},"evals":[],"jsHooks":[]}</script>
```

<p class="caption">(\#fig:cycle-hire)La distribution spatiale des points de location de vélos à Londres, basée sur les données officielles (bleu) et les données OpenStreetMap (rouge).</p>
</div>

Imaginons que nous ayons besoin de joindre la variable "capacité" de `cycle_hire_osm` aux données officielles "cible" contenues dans `cycle_hire`.
Dans ce cas, une jointure sans chevauchement est nécessaire.
La méthode la plus simple est d'utiliser l'opérateur topologique `st_is_within_distance()`, comme démontré ci-dessous en utilisant une distance seuil de 20 m (notez que cela fonctionne avec des données projetées et non projetées).


```r
sel = st_is_within_distance(cycle_hire, cycle_hire_osm, dist = 20)
summary(lengths(sel) > 0)
#>    Mode   FALSE    TRUE 
#> logical     304     438
```






Cela montre qu'il y a 438 des points dans l'objet cible `cycle_hire` dans la distance seuil (20 m) de `cycle_hire_osm`.
Comment récupérer les *valeurs* associées aux points respectifs de `cycle_hire_osm` ?
La solution est à nouveau avec `st_join()`, mais avec un argument `dist` supplémentaire (fixé à 20 m en dessous) :


```r
z = st_join(cycle_hire, cycle_hire_osm, st_is_within_distance, dist = 20)
nrow(cycle_hire)
#> [1] 742
nrow(z)
#> [1] 762
```

Remarquez que le nombre de lignes dans le résultat joint est supérieur à la cible.
Cela est dû au fait que certaines stations de location de vélos dans `cycle_hire` ont plusieurs correspondances dans `cycle_hire_osm`.
Pour agréger les valeurs des points qui se chevauchent et renvoyer la moyenne, nous pouvons utiliser les méthodes d'agrégation apprises au chapitre \@ref(attr), ce qui donne un objet avec le même nombre de lignes que la cible :


```r
z = z %>% 
  group_by(id) %>% 
  summarize(capacity = mean(capacity))
nrow(z) == nrow(cycle_hire)
#> [1] TRUE
```

La capacité des stations proches peut être vérifiée en comparant les cartes de la capacité des données source `cycle_hire_osm` avec les résultats dans ce nouvel objet (cartes non montrées) :


```r
plot(cycle_hire_osm["capacity"])
plot(z["capacity"])
```

Le résultat de cette jointure a utilisé une opération spatiale pour modifier les données attributaires associées aux entités simples ; la géométrie associée à chaque entité est restée inchangée.

### Agrégats spatiaux {#spatial-aggr}

Comme pour l'agrégation de données attributaires, l'agrégation de données spatiales *condense* les données : les sorties agrégées comportent moins de lignes que les entrées non agrégées.
Les *fonctions d'agrégation* statistiques, telles que la moyenne ou la somme, résument plusieurs valeurs \index{statistiques} d'une variable et renvoient une seule valeur par *variable de regroupement*.
La section \@ref(vector-attribute-aggregation) a montré comment `aggregate()` et `group_by() |> summarize()` condensent les données basées sur des variables d'attributs, cette section montre comment les mêmes fonctions fonctionnent avec des objets spatiaux.
\index{aggregation!spatial}

Pour revenir à l'exemple de la Nouvelle-Zélande, imaginez que vous voulez connaître la hauteur moyenne des points hauts de chaque région : c'est la géométrie de la source (`y` ou `nz` dans ce cas) qui définit comment les valeurs de l'objet cible (`x` ou `nz_height`) sont regroupées.
Ceci peut être fait en une seule ligne de code avec la méthode `aggregate()` de la base R :


```r
nz_agg = aggregate(x = nz_height, by = nz, FUN = mean)
```

Le résultat de la commande précédente est un objet `sf` ayant la même géométrie que l'objet d'agrégation (spatiale) (`nz`), ce que vous pouvez vérifier avec la commande `identical(st_geometry(nz), st_geometry(nz_agg))`.
Le résultat de l'opération précédente est illustré dans la Figure \@ref(fig:spatial-aggregation), qui montre la valeur moyenne des caractéristiques de `nz_height` dans chacune des 16 régions de la Nouvelle-Zélande.
Le même résultat peut également être généré en passant la sortie de `st_join()` dans les fonctions 'tidy' `group_by()` et `summarize()` comme suit :

<div class="figure" style="text-align: center">
<img src="04-spatial-operations_files/figure-html/spatial-aggregation-1.png" alt="Hauteur moyenne des 101 points culminants par régions de la Nouvelle-Zélande." width="50%" />
<p class="caption">(\#fig:spatial-aggregation)Hauteur moyenne des 101 points culminants par régions de la Nouvelle-Zélande.</p>
</div>


```r
nz_agg2 = st_join(x = nz, y = nz_height) |>
  group_by(Name) |>
  summarize(elevation = mean(elevation, na.rm = TRUE))
```



Les entités `nz_agg` résultants ont la même géométrie que l'objet d'agrégation `nz` mais avec une nouvelle colonne résumant les valeurs de `x` dans chaque région en utilisant la fonction `mean()`.
D'autres fonctions peuvent, bien sûr, remplacer `mean()` comme la `median()`, `sd()` ou d'autres fonctions qui retournent une seule valeur par groupe.
Remarque : une différence entre les approches `aggregate()` et `group_by() |> summarize()` est que la première donne des valeurs `NA` pour les noms de régions non correspondantes, tandis que la seconde préserve les noms de régions.
L'approche "tidy" est plus flexible en termes de fonctions d'agrégation et de noms de colonnes des résultats.
Les opérations d'agrégation créant  de nouvelles géométries sont décrites dans la section \@ref(geometry-unions). 


### Jointure de couches sans superposition parfaite  {#incongruent}

La congruence spatiale\index{congruence spatiale} est un concept important lié à l'agrégation spatiale.
Un *objet d'agrégation* (que nous appellerons `y`) est *congruent* avec l'objet cible (`x`) si les deux objets ont des frontières communes.
C'est souvent le cas des données sur les limites administratives, où des unités plus grandes --- telles que les Middle Layer Super Output Areas ([MSOAs](https://www.ons.gov.uk/methodology/geography/ukgeographies/censusgeography)) au Royaume-Uni ou les districts dans de nombreux autres pays européens --- sont composées de nombreuses unités plus petites.

Les objets d'agrégation *non congruent*, en revanche, ne partagent pas de frontières communes avec la cible [@qiu_development_2012].
C'est un problème pour l'agrégation spatiale (et d'autres opérations spatiales) illustrée dans la figure \@ref(fig:areal-example) : agréger le centroïde de chaque sous-zone ne donnera pas de résultats précis.
L'interpolation surfacique résout ce problème en transférant les valeurs d'un ensemble d'unités surfaciques à un autre, à l'aide d'une gamme d'algorithmes comprenant des approches simples de pondération de la surface et des approches plus sophistiquées telles que les méthodes " pycnophylactiques " [@tobler_smooth_1979].

<div class="figure" style="text-align: center">
<img src="04-spatial-operations_files/figure-html/areal-example-1.png" alt="Illustration des entités surfaciques congruentes (à gauche) et non congruentes (à droite) par rapport à des zones d'agrégation plus grandes (bordures bleues translucides)." width="100%" />
<p class="caption">(\#fig:areal-example)Illustration des entités surfaciques congruentes (à gauche) et non congruentes (à droite) par rapport à des zones d'agrégation plus grandes (bordures bleues translucides).</p>
</div>

Le paquet **spData** contient un jeux de données nommé `incongruent` (polygones colorés avec des bordures noires dans le panneau de droite de la Figure \@ref(fig:areal-example)) et un ensemble de données nommé `aggregating_zones` (les deux polygones avec la bordure bleue translucide dans le panneau de droite de la Figure \@ref(fig:areal-example)).
Supposons que la colonne `valeur` de `incongruent` se réfère au revenu régional total en millions d'euros.
Comment pouvons-nous transférer les valeurs des neuf polygones spatiaux sous-jacents dans les deux polygones de `aggregating_zones` ?

La méthode la plus simple est l'interpolation spatiale pondérée par la surface, qui transfère les valeurs de l'objet `non congruent` vers une nouvelle colonne dans `aggregating_zones` proportionnellement à la surface de recouvrement : plus l'intersection spatiale entre les caractéristiques d'entrée et de sortie est grande, plus la valeur correspondante est grande.
Ceci est implémenté dans `st_interpolate_aw()`, comme démontré dans le morceau de code ci-dessous.


```r
iv = incongruent["value"] # garde uniquement les valeurs à transférer
agg_aw = st_interpolate_aw(iv, aggregating_zones, extensive = TRUE)
#> Warning in st_interpolate_aw.sf(iv, aggregating_zones, extensive = TRUE):
#> st_interpolate_aw assumes attributes are constant or uniform over areas of x
agg_aw$value
#> [1] 19.6 25.7
```

Dans notre cas, il est utile d'additionner les valeurs des intersections qui se trouvent dans les zones d'agrégation, car le revenu total est une variable dite spatialement extensive (qui augmente avec la superficie), en supposant que le revenu est réparti uniformément dans les zones plus petites (d'où le message d'avertissement ci-dessus).
Il en va différemment pour les variables spatialement [intensives](https://geodacenter.github.io/workbook/3b_rates/lab3b.html#spatially-extensive-and-spatially-intensive-variables) telles que le revenu *moyen* ou les pourcentages, qui n'augmentent pas avec la superficie.
`st_interpolate_aw()` fonctionne également avec des variables spatialement intensives : mettez le paramètre `extensive` à `FALSE` et il utilisera une moyenne plutôt qu'une fonction de somme pour faire l'agrégation.

### Les relations de distance 

Alors que les relations topologiques sont binaires --- une caractéristique est soit en intersection avec une autre ou non --- les relations de distance sont continues.
La distance entre deux objets est calculée avec la fonction `st_distance()`.
Ceci est illustré dans l'extrait de code ci-dessous, qui trouve la distance entre le point le plus élevé de Nouvelle-Zélande et le centroïde géographique de la région de Canterbury, créé dans la section \@ref(spatial-subsetting) :
\index{sf!distance relations}


```r
nz_highest = nz_height |> slice_max(n = 1, order_by = elevation)
canterbury_centroid = st_centroid(canterbury)
st_distance(nz_highest, canterbury_centroid)
#> Units: [m]
#>        [,1]
#> [1,] 115540
```

Il y a deux choses potentiellement surprenantes dans ce résultat :

- Il a des `unités`, ce qui nous indique que la distance est de 100 000 mètres, et non de 100 000 pouces, ou toute autre mesure de distance.
- Elle est retournée sous forme de matrice, même si le résultat ne contient qu'une seule valeur.

Cette deuxième caractéristique laisse entrevoir une autre fonctionnalité utile de `st_distance()`, sa capacité à retourner des *matrices de distance* entre toutes les combinaisons de caractéristiques dans les objets `x` et `y`.
Ceci est illustré dans la commande ci-dessous, qui trouve les distances entre les trois premières caractéristiques de `nz_height` et les régions d'Otago et de Canterbury en Nouvelle-Zélande représentées par l'objet `co`.


```r
co = filter(nz, grepl("Canter|Otag", Name))
st_distance(nz_height[1:3, ], co)
#> Units: [m]
#>        [,1]  [,2]
#> [1,] 123537 15498
#> [2,]  94283     0
#> [3,]  93019     0
```

Remarquez que le distance entre les deuxième et troisième éléments de `nz_height` et le deuxième élément de `co` est de zéro.
Cela démontre le fait que les distances entre les points et les polygones se réfèrent à la distance à *n'importe quelle partie du polygone* :
Les deuxième et troisième points dans `nz_height` sont *dans* Otago, ce qui peut être vérifié en les traçant (résultat non montré) :


```r
plot(st_geometry(co)[2])
plot(st_geometry(nz_height)[2:3], add = TRUE)
```

## Opérations spatiales sur les données raster {#spatial-ras}

Cette section s'appuie sur la section \@ref(manipulating-raster-objects), qui met en évidence diverses méthodes de base pour manipuler des jeux de données raster. Elle illustre des opérations raster plus avancées et explicitement spatiales, et utilise les objets `elev` et `grain` créés manuellement dans la section \@ref(manipulating-raster-objects).
Pour la commodité du lecteur, ces jeux de données se trouvent également dans le paquet **spData**.

### Sélection spatiale {#spatial-raster-subsetting}

Le chapitre précédent (Section \@ref(manipulating-raster-objects)) a montré comment extraire les valeurs associées à des ID de cellules spécifiques ou à des combinaisons de lignes et de colonnes.
Les objets raster peuvent également être extraits par leur emplacement (coordonnées) et via d'autres objets spatiaux.
Pour utiliser les coordonnées pour la sélection, on peut "traduire" les coordonnées en un ID de cellule avec la fonction **terra** `cellFromXY()`.
Une alternative est d'utiliser `terra::extract()` (attention, il existe aussi une fonction appelée `extract()` dans le **tidyverse**\index{tidyverse (package)}) pour extraire des valeurs.
Les deux méthodes sont démontrées ci-dessous pour trouver la valeur de la cellule qui couvre un point situé aux coordonnées 0,1, 0,1.
\index{raster!subsetting}
\index{spatial!subsetting}


```r
id = cellFromXY(elev, xy = matrix(c(0.1, 0.1), ncol = 2))
elev[id]
# the same as
terra::extract(elev, matrix(c(0.1, 0.1), ncol = 2))
```

<!--jn:toDo-->
<!-- to update? -->
<!-- It is convenient that both functions also accept objects of class `Spatial* Objects`. -->
Les objets raster peuvent également être sélectionnés avec un autre objet raster, comme le montre le code ci-dessous :


```r
clip = rast(xmin = 0.9, xmax = 1.8, ymin = -0.45, ymax = 0.45,
            resolution = 0.3, vals = rep(1, 9))
elev[clip]
# on peut aussi utiliser extract
# terra::extract(elev, ext(clip))
```

Cela revient à récupérer les valeurs du premier objet raster (dans ce cas, `elev`) qui se trouvent dans l'étendue d'un second objet raster (ici : `clip`), comme illustré dans la figure \@ref(fig:raster-subset).

<div class="figure" style="text-align: center">
<img src="figures/04_raster_subset.png" alt="Raster originale (à gauche). Masque raster (au milieu). Résultat du clip (à droite)." width="100%" />
<p class="caption">(\#fig:raster-subset)Raster originale (à gauche). Masque raster (au milieu). Résultat du clip (à droite).</p>
</div>

L'exemple ci-dessus a retourné les valeurs de cellules spécifiques, mais dans de nombreux cas, ce sont des sorties spatiales qui sont nécessaires.
Cela peut être fait en utilisant l'opérateur `[`, avec `drop = FALSE`, comme indiqué dans la section \@ref(manipulating-raster-objects), qui montre également comment les objets raster peuvent être sélectionnés par divers objets.
Le code ci-dessous en est un exemple  en retournant les deux premières cellules (de la ligne supérieure) de `elev` en tant qu'objet raster  (seules les 2 premières lignes de la sortie sont montrées) :


```r
elev[1:2, drop = FALSE]    # sélection spatiale par ID
#> class       : SpatRaster 
#> dimensions  : 1, 2, 1  (nrow, ncol, nlyr)
#> ...
```



Un autre cas d'utilisation courante de sélection spatiale est celle où une image raster avec des valeurs `logiques` (ou `NA`) est utilisée pour masquer une autre image raster avec la même étendue et la même résolution, comme illustré dans la Figure \@ref(fig:raster-subset).
Dans ce cas, les fonctions `[` et `mask()` peuvent être utilisées (résultats non montrés) :


```r
# créer un masque raster
rmask = elev
values(rmask) = sample(c(NA, TRUE), 36, replace = TRUE)
```

Dans le morceau de code ci-dessus, nous avons créé un objet masque appelé `rmask` avec des valeurs assignées aléatoirement à `NA` et `TRUE`.
Ensuite, nous voulons garder les valeurs de `elev` qui sont `TRUE` dans `rmask`.
En d'autres termes, nous voulons masquer `elev` avec `rmask`.


```r
# sélection spatiale
elev[rmask, drop = FALSE]           # avec l'opérateur [ 
mask(elev, rmask)                   # avec mask()
```

L'approche ci-dessus peut également être utilisée pour remplacer certaines valeurs (par exemple, celles qui seraient fausses) par NA.  


```r
elev[elev < 20] = NA
```

Ces opérations sont en fait des opérations booléennes locales puisque nous comparons deux rasters par cellule.
La sous-section suivante explore ces opérations et d'autres opérations connexes plus en détail.

### Algèbre raster

\index{map algebra}
Le terme "algèbre raster" a été inventé à la fin des années 1970 pour décrire un "ensemble de conventions, de capacités et de techniques" pour l'analyse des données géographiques raster *et* (bien que moins marquées) vectorielles [@tomlin_map_1994].
<!-- Although the concept never became widely adopted, the term usefully encapsulates and helps classify the range operations that can be undertaken on raster datasets. -->
Dans ce contexte, nous définissons l'algèbre raster plus strictement, comme des opérations qui modifient ou résument les valeurs des cellules raster, en référence aux cellules environnantes, aux zones ou aux fonctions statistiques s'appliquant à chaque cellule.

Les opérations d'algèbre raster ont tendance à être rapides, car les jeux de données raster ne stockent qu'implicitement les coordonnées, d'où le [vieil adage](https://geozoneblog.wordpress.com/2013/04/19/raster-vs-vector/) "raster is faster but vector is corrector".
La position des cellules dans les jeux de données raster peut être calculée à l'aide de leur position matricielle, de la résolution et de l'origine du jeu de données (stockées dans l'en-tête).
Pour le traitement, cependant, la position géographique d'une cellule n'est guère pertinente tant que nous nous assurons que la position de la cellule est toujours la même après le traitement.
De plus, si deux ou plusieurs jeux de données raster partagent la même étendue, projection et résolution, on peut les traiter comme des matrices pour le traitement.

C'est de cette manière que l'algèbre raster fonctionne avec le paquet **terra**.
Premièrement, les en-têtes des jeux de données raster sont interrogés et (dans les cas où les opérations d'algèbre raster fonctionnent sur plus d'un ensemble de données) vérifiés pour s'assurer que les jeux de données sont compatibles.
Deuxièmement, l'algèbre raster conserve ce que l'on appelle la correspondance de localisation une à une, ce qui signifie que les cellules ne peuvent pas se déplacer.
Cela diffère de l'algèbre matricielle, dans laquelle les valeurs changent de position, par exemple lors de la multiplication ou de la division de matrices.

L'algèbre raster (ou modélisation cartographique avec des données raster) divise les opérations sur des rasters en quatre sous-classes [@tomlin_geographic_1990], chacune travaillant sur une ou plusieurs grilles simultanément :

1. Opérations *locales* ou par cellule
2. Les opérations *Focales* ou de voisinage.
Le plus souvent la valeur de la cellule de sortie est le résultat d'un bloc de cellules d'entrée de 3 x 3 cellules
3. Les opérations *Zonales* sont similaires aux opérations focales, mais la grille de pixels environnante sur laquelle les nouvelles valeurs sont calculées peut avoir des tailles et des formes irrégulières.
4. Les opérations *Globales* ou par-raster. 
Ici la cellule de sortie dérive potentiellement sa valeur d'un ou de plusieurs rasters entiers.

Cette typologie classe les opérations d'algèbre raster en fonction du nombre de cellules utilisées pour chaque étape de traitement des pixels et du type de sortie.
Par souci d'exhaustivité, nous devons mentionner que les opérations sur des rasters peuvent également être classées par discipline, comme le terrain, l'analyse hydrologique ou la classification des images.
Les sections suivantes expliquent comment chaque type d'opérations d'algèbre raster peut être utilisé, en se référant à des exemples documentés.

### Opérations locales

\index{map algebra!local operations}
Les opérations **locales** comprennent toutes les opérations cellule par cellule dans une ou plusieurs couches.
Elles ont un cas typique d'algèbre raster et comprennent l'ajout ou la soustraction de valeurs d'une image raster, l'élévation au carré et la multiplication d'images raster.
L'algèbre raster permet également des opérations logiques telles que la recherche de toutes les cellules qui sont supérieures à une valeur spécifique (5 dans notre exemple ci-dessous).
Le paquet **terra** prend en charge toutes ces opérations et bien plus encore, comme le montre la figure ci-dessous (\@ref(fig:04-local-operations)):


```r
elev + elev
elev^2
log(elev)
elev > 5
```

<div class="figure" style="text-align: center">
<img src="figures/04-local-operations.png" alt="Exemples de différentes opérations locales de l'objet raster elev : additionner deux rasters, élever au carré, appliquer une transformation logarithmique et effectuer une opération logique." width="100%" />
<p class="caption">(\#fig:04-local-operations)Exemples de différentes opérations locales de l'objet raster elev : additionner deux rasters, élever au carré, appliquer une transformation logarithmique et effectuer une opération logique.</p>
</div>

Un autre bon exemple d'opérations locales est la classification d'intervalles de valeurs numériques en groupes, comme le regroupement d'un modèle numérique d'élévation en altitudes basses (classe 1), moyennes (classe 2) et hautes (classe 3).
En utilisant la commande `classify()`, nous devons d'abord construire une matrice de classification, où la première colonne correspond à l'extrémité inférieure et la deuxième colonne à l'extrémité supérieure de la classe.
La troisième colonne représente la nouvelle valeur pour les plages spécifiées dans les colonnes un et deux.


```r
rcl = matrix(c(0, 12, 1, 12, 24, 2, 24, 36, 3), ncol = 3, byrow = TRUE)
rcl
#>      [,1] [,2] [,3]
#> [1,]    0   12    1
#> [2,]   12   24    2
#> [3,]   24   36    3
```

Ici, nous affectons les valeurs du raster dans les plages 0--12, 12--24 et 24--36 et les *reclassons* pour prendre les valeurs 1, 2 et 3, respectivement.


```r
recl = classify(elev, rcl = rcl)
```

La fonction `classify()` peut également être utilisée lorsque nous voulons réduire le nombre de classes dans nos rasters catégorisés.
Nous effectuerons plusieurs reclassements supplémentaires dans le chapitre \@ref(location)'.

En dehors des opérateurs arithmétiques, on peut aussi utiliser les fonctions `app()`, `tapp()` et `lapp()`.
Elles sont plus efficaces, et donc préférables pour les grands jeux de données raster. 
De plus, elles vous permettent d'enregistrer directement un fichier de sortie.
La fonction `app()` applique une fonction à chaque cellule d'une couche matricielle et est utilisée pour résumer (par exemple, en calculant la somme) les valeurs de plusieurs couches en une seule couche.
`tapp()` est une extension de `app()`, nous permettant de sélectionner un sous-ensemble de couches (voir l'argument `index`) pour lesquelles nous voulons effectuer une certaine opération.
Enfin, la fonction `lapp()` permet d'appliquer une fonction à chaque cellule en utilisant les couches comme arguments -- une application de `lapp()` est présentée ci-dessous.

Le calcul du *normalized difference vegetation index* (NDVI) est une opération raster locale (pixel par pixel) bien connue.
Elle produit un raster dont les valeurs sont comprises entre -1 et 1 ; les valeurs positives indiquent la présence de plantes vivantes (le plus souvent > 0,2).
Le NDVI est calculé à partir des bandes rouge et proche infrarouge (NIR) d'images de télédétection, généralement issues de systèmes satellitaires tels que Landsat ou Sentinel.
La végétation absorbe fortement la lumière dans le spectre de la lumière visible, et surtout dans le canal rouge, tout en réfléchissant la lumière NIR, ce qui explique la formule du NDVI :

$$
\begin{split}
NDVI&= \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}\\
\end{split}
$$

Calculons le NDVI pour l'image satellite multispectrale du parc National de Zion.


```r
multi_raster_file = system.file("raster/landsat.tif", package = "spDataLarge")
multi_rast = rast(multi_raster_file)
```

L'objet raster comporte quatre bandes de satellite - bleu, vert, rouge et proche infrarouge (NIR).
Notre prochaine étape va être d'implémenter la formule NDVI dans une fonction R :


```r
ndvi_fun = function(nir, red){
  (nir - red) / (nir + red)
}
```

Cette fonction accepte deux arguments numériques, `nir` et `red`, et retourne un vecteur numérique avec les valeurs NDVI.
Elle peut être utilisée comme argument `fun` de `lapp()`.
Nous devons juste nous rappeler que notre fonction n'a besoin que de deux bandes (et non de quatre comme dans le raster original), et qu'elles doivent être dans l'ordre NIR puis `red`.
C'est pourquoi nous sélection la grille d'entrée avec `multi_rast[[c(4, 3)]]` avant d'effectuer tout calcul.


```r
ndvi_rast = lapp(multi_rast[[c(4, 3)]], fun = ndvi_fun)
```

Le résultat, présenté sur le panneau de droite de la figure \@ref(fig:04-ndvi), peut être comparé à l'image RVB de la même zone (panneau de gauche de la même figure).
Il nous permet de constater que les plus grandes valeurs de NDVI sont liées aux zones de forêt dense dans les parties nord de la zone, tandis que les valeurs les plus faibles sont liées au lac au nord et aux crêtes montagneuses enneigées.

<div class="figure" style="text-align: center">
<img src="figures/04-ndvi.png" alt="Image RVB (à gauche) et valeurs NDVI (à droite) calculées pour l'exemple de l'image satellite du parc National de Zion." width="100%" />
<p class="caption">(\#fig:04-ndvi)Image RVB (à gauche) et valeurs NDVI (à droite) calculées pour l'exemple de l'image satellite du parc National de Zion.</p>
</div>

La cartographie prédictive est une autre application intéressante des opérations raster locales.
La variable de réponse correspond à des points mesurés ou observés dans l'espace, par exemple, la richesse des espèces, la présence de glissements de terrain, les maladies des arbres ou le rendement des cultures.
Par conséquent, nous pouvons facilement récupérer des variables prédictives spatiales ou aériennes à partir de divers rasters (élévation, pH, précipitations, température, couverture végétale, classe de sol, etc.)
Ensuite, nous modélisons notre réponse en fonction de nos prédicteurs en utilisant `lm()`, `glm()`, `gam()` ou une technique d'apprentissage automatique. 
Les prédictions spatiales sur les objets raster peuvent donc être réalisées en appliquant des coefficients estimés aux valeurs de chaque cellule, et en additionnant les valeurs raster de sortie (voir chapitre \@ref(eco)).

### Operations focales

\index{map algebra!focal operations}
Alors que les fonctions locales opèrent sur une cellule, bien que pouvant provenir de plusieurs couches, les opérations **focales** prennent en compte une cellule centrale (focale) et ses voisines.
Le voisinage (également appelé noyau, filtre ou fenêtre mobile) utilisé est généralement de le taille 3 par 3 cellules (c'est-à-dire la cellule centrale et ses huit voisines), mais peut prendre toute autre forme (pas nécessairement rectangulaire) définie par l'utilisateur.
Une opération focale applique une fonction d'agrégation à toutes les cellules du voisinage spécifié, utilise la sortie correspondante comme nouvelle valeur pour la cellule centrale, puis passe à la cellule centrale suivante (figure \@ref(fig:focal-example)).
Cette opération est également appelée filtrage spatial et convolution [@burrough_principles_2015].

Dans R, nous pouvons utiliser la fonction `focal()` pour effectuer un filtrage spatial. 
Nous définissons la forme de la fenêtre mobile avec une `matrice` dont les valeurs correspondent aux poids (voir le paramètre `w` dans le code ci-dessous).
Ensuite, le paramètre `fun` nous permet de spécifier la fonction que nous souhaitons appliquer à ce voisinage.
Ici, nous choisissons le minimum, mais toute autre fonction de résumé, y compris `sum()`, `mean()`, ou `var()` peut être utilisée.


```r
r_focal = focal(elev, w = matrix(1, nrow = 3, ncol = 3), fun = min)
```

Cette fonction accepte également des arguments supplémentaires, par exemple, si elle doit supprimer les NAs dans le processus (`na.rm = TRUE`) ou non (`na.rm = FALSE`).

<div class="figure" style="text-align: center">
<img src="figures/04_focal_example.png" alt="Raster d'entrée (gauche) et raster de sortie (droite) suite à une opération focale - trouver la valeur minimale dans des fenêtres mobiles 3 par 3." width="100%" />
<p class="caption">(\#fig:focal-example)Raster d'entrée (gauche) et raster de sortie (droite) suite à une opération focale - trouver la valeur minimale dans des fenêtres mobiles 3 par 3.</p>
</div>

Nous pouvons rapidement vérifier si le résultat correspond à nos attentes.
En effet, dans notre exemple, la valeur minimale doit toujours se situer dans le coin supérieur gauche de la fenêtre mobile (rappelez-vous que nous avons créé le raster d'entrée en incrémentant les valeurs des cellules d'une unité par ligne, en commençant par le coin supérieur gauche).
Dans cet exemple, la matrice de pondération est composée uniquement de 1, ce qui signifie que chaque cellule a le même poids sur la sortie, mais cela peut être modifié.

Les fonctions ou filtres focaux jouent un rôle dominant dans le traitement des images.
Les filtres "passe-bas" (*low-pass*) ou de lissage utilisent la fonction moyenne pour éliminer les valeurs extrêmes.
Dans le cas de données catégorielles, on peut remplacer la moyenne par le mode, qui est la valeur la plus courante.
En revanche, les filtres "passe-haut" accentuent les entités.
Les filtres Laplace et Sobel de détection de lignes peuvent servir d'exemple ici.
Consultez la page d'aide `focal()` pour savoir comment les utiliser dans R (ils seront également utilisés dans les exercices à la fin de ce chapitre).

Le traitement des données numériques de terrain, le calcul des caractéristiques topographiques telles que la pente, l'aspect et les directions d'écoulement, repose sur des fonctions focales.
La fonction de **terra** `terrain()` peut être utilisé pour calculer ces métriques, bien que certains algorithmes de traitement des données numériques de terrain, y compris la méthode de Zevenbergen et Thorne pour calculer la pente, n'y soient pas implémentés.
De nombreux autres algorithmes --- notamment les courbures, les zones contributives et les indices d'humidité --- sont mis en œuvre dans des logiciels libres de systèmes d'information géographique (SIG) de bureau.
Le chapitre \@ref(gis) montre comment accéder à ces fonctionnalités SIG à partir de R.

### Opérations Zonales

\index{map algebra!zonal operations}
Tout comme les opérations focales, les opérations *zonales* appliquent une fonction d'agrégation à plusieurs cellules.
Cependant, un deuxième raster, généralement avec des valeurs catégorielles, définit les *filtres zonaux* (ou 'zones') dans le cas des opérations zonales, en opposition à une fenêtre de voisinage prédéfinie dans le cas de l'opération focale.
Par conséquent, les cellules  définissant le filtre zonal ne doivent pas nécessairement être voisines.
Notre raster `grain` en est un bon exemple, comme l'illustre le panneau de droite de la figure \@ref(fig:cont-raster) : différentes tailles de granulométrie sont réparties de manière irrégulière dans le raster.
Enfin, le résultat d'une opération zonale est un tableau récapitulatif groupé par zone, c'est pourquoi cette opération est également connue sous le nom de *statistiques zonales* dans le monde des SIG\index{GIS}. 
Ceci est à l'opposé des opérations focales qui retournent un objet matriciel.

Le  code suivant utilise la fonction `zonal()` pour calculer l'altitude moyenne associée à chaque classe de taille de grain.


```r
z = zonal(elev, grain, fun = "mean")
z
#>   grain elev
#> 1  clay 14.8
#> 2  silt 21.2
#> 3  sand 18.7
```

ceci renvoie les statistiques\index{statistics} pour chaque catégorie, ici l'altitude moyenne pour chaque classe de granulométrie.
Remarque : il est aussi possible d'obtenir un raster avec les statistiques calculées pour chaque zone en mettant l'argument `as.raster` à `TRUE`.

### Opérations globales et distances

Les opérations *globales* sont un cas particulier des opérations zonales, l'ensemble des données raster représentant une seule zone.
Les opérations globales les plus courantes sont des statistiques descriptives pour l'ensemble des données raster, telles que le minimum ou le maximum, que nous avons déjà abordées dans la section \@ref(summarizing-raster-objects).

En dehors de cela, les opérations globales sont également utiles pour le calcul des rasters de distance et de poids.
Dans le premier cas, on peut calculer la distance entre chaque cellule et une cellule cible spécifique.
Par exemple, on peut vouloir calculer la distance à la côte la plus proche (voir aussi `terra::distance()`).
On peut aussi vouloir prendre en compte la topographie, c'est-à-dire qu'on n'est pas seulement intéressé par la distance pure mais qu'on voudrait aussi éviter de traverser des chaînes de montagnes en allant vers la côte.
Pour ce faire, nous pouvons pondérer la distance par l'altitude de sorte que chaque mètre d'altitude supplémentaire "prolonge" la distance euclidienne.
Les calculs de visibilité et de bassin visuel appartiennent également à la famille des opérations globales (dans les exercices du chapitre \@ref(gis), vous calculerez un raster de bassin visuel).

### Les contreparties de l'algèbre raster dans le traitement vectoriel

De nombreuses opérations d'algèbre raster ont une contrepartie dans le traitement vectoriel [@liu_essential_2009].
Le calcul d'une distance raster (opération globale) en ne considérant qu'une distance maximale (opération focale logique) est l'équivalent d'une opération de mise en mémoire tampon vectorielle (section \@ref(clipping)).
Le reclassement de données raster (fonction locale ou zonale selon l'entrée) est équivalent à la dissolution de données vectorielles (section \@ref(spatial-joining)). 
La superposition de deux données matricielles (opération locale), dont l'une contient des valeurs `NULL` ou `NA` représentant un masque, est similaire au découpage vectoriel (Section `@ref(clipping)).
L'intersection de deux couches est tout à fait similaire au détourage spatial (section \@ref(spatial-subsetting)). 
La différence est que ces deux couches (vectorielles ou matricielles) partagent simplement une zone de chevauchement (voir la figure \@ref(fig:venn-clip) pour un exemple).
Cependant, faites attention à la formulation.
Parfois, les mêmes mots ont des significations légèrement différentes pour les modèles de données rasters et vectorielles.
Dans le cas des données vectorielles, l'agrégation consiste à dissoudre les polygones, tandis que dans le cas des données rasters, elle consiste à augmenter la résolution.
En fait, on pourrait considérer que dissoudre ou agréger des polygones revient à diminuer la résolution. 
Cependant, les opérations zonales semble être le meilleur équivalent raster par rapport à la modification de la résolution des cellules. 
Les opérations zonales peuvent dissoudre les cellules d'un raster en fonction des zones (catégories) d'un autre raster en utilisant une fonction d'agrégation (voir ci-dessus).

### Fusionner des rasters

\index{raster!merge}
Supposons que nous voulions calculer le NDVI (voir la section \@ref(local-operations)), et que nous voulions en plus calculer les attributs du terrain à partir des données d'altitude pour des observations dans une zone d'étude.
Ces calculs reposent sur des informations de télédétection. 
L'imagerie correspondante est souvent divisée en tuiles couvrant une étendue spatiale spécifique, et fréquemment, une zone d'étude couvre plus d'une tuile.
Dans ce cas, nous devons fusionner les tuiles couvertes par notre zone d'étude. 
Le plus simple est alors de fusionner ces scènes, c'est-à-dire les mettre côte à côte.
Cela est possible, par exemple, avec les données numériques d'altitude (SRTM, ASTER).
Dans l'extrait de code suivant, nous téléchargeons d'abord les données d'élévation SRTM pour l'Autriche et la Suisse (pour les codes pays, voir la fonction **geodata** `country_codes()`).
Dans une seconde étape, nous fusionnons les deux rasters en un seul.


```r
aut = geodata::elevation_30s(country = "AUT", path = tempdir())
ch = geodata::elevation_30s(country = "CHE", path = tempdir())
aut_ch = merge(aut, ch)
```

La commande `merge()` de **terra** combine deux images, et dans le cas où elles se chevauchent, elle utilise la valeur du premier raster.
<!--jn:toDo-->
<!-- gdalUtils is slower (for this files): -->
<!-- two_rast = c(terra::sources(aut)$source, terra::sources(ch)$source) -->
<!-- tf = tempfile(fileext = ".tif") -->
<!-- bench::mark({gdalUtils::mosaic_rasters(two_rast, tf)}) -->
<!-- You can do exactly the same with `gdalUtils::mosaic_rasters()` which is faster, and therefore recommended if you have to merge a multitude of large rasters stored on disk. -->

cette approche de fusion est peu utile lorsque les valeurs qui se chevauchent ne correspondent pas les unes aux autres.
C'est souvent le cas lorsque vous voulez combiner des images spectrales provenant de scènes qui ont été prises à des dates différentes.
La commande `merge()` fonctionnera toujours mais vous verrez une frontière nette dans l'image résultante.
D'autre part, la commande `mosaic()` vous permet de définir une fonction pour la zone de recouvrement. 
Par exemple, nous pourrions calculer la valeur moyenne -- cela pourrait lisser la bordure claire dans le résultat fusionné, mais ne la fera probablement pas disparaître.
<!-- The following sentences have been commented out and can be removed because the packages, and info, is now out of date -->
<!-- See https://github.com/Robinlovelace/geocompr/pull/424 for discussion -->
<!-- To do so, we need a more advanced approach.  -->
<!-- Remote sensing scientists frequently apply histogram matching or use regression techniques to align the values of the first image with those of the second image. -->
<!-- The packages **landsat** (`histmatch()`, `relnorm()`, `PIF()`), **satellite** (`calcHistMatch()`) and **RStoolbox** (`histMatch()`, `pifMatch()`) provide the corresponding functions for the **raster**'s package objects. -->
Pour une introduction plus détaillée à la télédétection avec R, il est possible de consulter @wegmann_remote_2016.
<!--jn:toDo-->
<!--update the above reference to the 2nd edition-->

## Exercises



```r
library(sf)
library(dplyr)
data(nz, package = "spData")
data(nz_height, package = "spData")
```

E1. Il a été établi dans la section \@ref(spatial-vec) que Canterbury était la région de Nouvelle-Zélande contenant la plupart des 100 points les plus élevés du pays.
Combien de ces points culminants en contient-elle ?

**Bonus:** Représentez le résultat en utilisant la fonction `plot()` en montrant toute la Nouvelle-Zélande, la région `canterbury` surlignée en jaune, les points hauts de Canterbury représentés par des points noirs




E2. Dans quelle région se trouve le deuxième plus grand nombre de points `nz_height`, et combien en compte-t-elle ?



E3. En généralisant la question à toutes les régions : combien les 16 régions de la Nouvelle-Zélande contiennent des points qui font partie des 100 plus hauts points du pays ? Quelles régions ?

- Bonus: créer un tableau listant ces régions dans l'ordre du nombre de points et de leurs noms.


E4. Testez vos connaissances des prédicats spatiaux en découvrant et en représentant graphiquement les relations entre les États américains et d'autres objets spatiaux.

Le point de départ de cet exercice est de créer un objet représentant l'état du Colorado aux USA. Faites-le avec la commande 
`colorado = us_states[us_states$NAME == "Colorado",]` (base R) ou avec la fonction `filter()` (tidyverse) et affichez l'objet résultant dans le contexte des états américains. 

- Créez un nouvel objet représentant tous les états qui ont une intersection géographique avec le Colorado et tracez le résultat (astuce : la façon la plus concise de le faire est d'utiliser la méthode de sous-ensemble `[`).
- Créez un autre objet représentant tous les objets qui touchent (ont une frontière commune avec) le Colorado et tracez le résultat (conseil : n'oubliez pas que vous pouvez utiliser l'argument `op = st_intersects` et d'autres relations spatiales pendant les opérations de sous-ensembles spatiaux dans R de base).
- Bonus : créez une ligne droite du centroïde du District de Columbia, près de la côte Est, au centroïde de la Californie, près de la côte Ouest des Etats-Unis (astuce : les fonctions `st_centroid()`, `st_union()` et `st_cast()` décrites au Chapitre 5 peuvent vous aider) et identifiez les états que cette longue ligne Est-Ouest traverse.












E5. Utilisez le `dem = rast(system.file("raster/dem.tif", package = "spDataLarge"))`, et reclassifiez l'altitude en trois classes : basse (<300), moyenne et haute (>500).
Ensuite, Chargez le raster NDVI (`ndvi = rast(system.file("raster/ndvi.tif", package = "spDataLarge"))`) et calculez le NDVI moyen et l'altitude moyenne pour chaque classe altitudinale.



E6. Appliquez un filtre de détection de ligne à `rast(system.file("ex/logo.tif", package = "terra"))`.
Affichez le résultat.
Astuce : lisez `?terra::focal()`.



E7. Calculez l'indice  *Normalized Difference Water Index* (NDWI ; `(green - nir)/(green + nir)`) d'une image Landsat. 
Utilisez l'image Landsat fournie par le paquet **spDataLarge** (`system.file("raster/landsat.tif", package = "spDataLarge")`).
Calculez également une corrélation entre le NDVI et le NDWI pour cette zone.



E8. Un billet de [StackOverflow](https://stackoverflow.com/questions/35555709/global-raster-of-geographic-distances) montre comment calculer les distances à la côte la plus proche en utilisant `raster::distance()`.
Essayez de faire quelque chose de similaire mais avec `terra::distance()` : récupérez le modèle numérique de terrain espagnole, et obtenez un raster qui représente les distances à la côte à travers le pays (astuce : utilisez `geodata::elevation_30s()`).
Convertissez les distances résultantes de mètres en kilomètres.
Remarque : il peut être judicieux d'augmenter la taille des cellules de l'image matricielle d'entrée pour réduire le temps de calcul pendant cette opération.



E9. Essayez de modifier l'approche utilisée dans l'exercice ci-dessus en pondérant le raster de distance avec le raster d'altitude ; chaque 100 mètres d'altitude devrait augmenter la distance à la côte de 10 km.
Ensuite, calculez et visualisez la différence entre le raster créé en utilisant la distance euclidienne (E7) et le raster pondéré par l'altitude.
