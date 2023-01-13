--- 
title: 'Geocomputation avec R'
author: 'Robin Lovelace, Jakub Nowosad, Jannes Muenchow'
date: '2023-01-13'
site: bookdown::bookdown_site
output: bookdown::bs4_book
documentclass: krantz
monofont: "Source Code Pro"
monofontoptions: "Scale=0.7"
bibliography:
  - geocompr.bib
  - packages.bib
biblio-style: apalike
link-citations: yes
colorlinks: yes
graphics: yes
description: "Geocomputation avec R s´adresse aux personnes qui souhaitent analyser, visualiser et modéliser des données géographiques à l´aide de logiciels open source. Il est basé sur R, un langage de programmation statistique qui possède de puissantes capacités de traitement de données, de visualisation et de traitement géospatial. Ce livre vous donne les connaissances et les compétences nécessaires pour aborder un large éventail de questions qui se manifestent dans les données géographiques, notamment celles qui ont des implications scientifiques, sociales et environnementales. Ce livre intéressera des personnes de tous horizons, en particulier les utilisateurs de systèmes d´information géographique (SIG) désireux d´appliquer leurs connaissances spécifiques à un domaine dans un puissant langage open source pour la science des données, et les utilisateurs de R désireux d'étendre leurs compétences au traitement des données spatiales."
github-repo: "geocompr/fr"
cover-image: "https://geocompr.github.io/es/images/cover.png"
url: https://geocompr.github.io/fr/
---



# Bienvenue! {-}

Il s'agit du site en français de *Geocomputation with R*, un livre sur l'analyse, la visualisation et la modélisation des données géographiques. Le site en anglais est [ici](https://geocompr.robinlovelace.net/).

<a href="https://www.routledge.com/9781138304512"><img src="images/cover.png" width="250" height="375" alt="The geocompr book cover" align="right" style="margin: 0 1em 0 1em" /></a>
  
**Note**: La première édition a été imprimée par CRC Press dans la collection [R Series](https://www.routledge.com/Chapman--HallCRC-The-R-Series/book-series/CRCTHERSER).
Il est possible de l'acheter chez [CRC Press](https://www.routledge.com/9781138304512), ou sur [Amazon](https://www.amazon.com/Geocomputation-R-Robin-Lovelace-dp-0367670577/dp/0367670577/), la **première version** est aussi hebergée sur [bookdown.org](https://bookdown.org/robinlovelace/geocompr/). 

Inspiré par le mouvement libriste et l'Open Source Software for Geospatial ([FOSS4G](https://foss4g.org/)), le code et le texte qui sous-tendent ce livre sont ouverts, ce qui garantit que le contenu est reproductible, transparent et accessible.
L'hébergement du code source sur [GitHub](https://github.com/Robinlovelace/geocompr/) permet à quiconque d'interagir avec le projet en ouvrant des questions ou en contribuant au nouveau contenu et à la correction des fautes de frappe pour le bénéfice de tous.

[![](https://img.shields.io/github/stars/robinlovelace/geocompr?style=for-the-badge)](https://github.com/robinlovelace/geocompr)
[![](https://img.shields.io/github/contributors/robinlovelace/geocompr?style=for-the-badge)](https://github.com/Robinlovelace/geocompr/graphs/contributors)

La version en ligne du livre est hébergée sur [geocompr.github.io](https://geocompr.github.io/fr/) et mise à jour par [GitHub Actions](https://github.com/geocompr/fr/actions).
L'état actuel de sa construction est le suivant : 

[![Actions](https://github.com/Robinlovelace/geocompr/workflows/Render/badge.svg)](https://github.com/geocompr/fr/actions)

Cette version du livre en français a été produite via l'action GitHub du 2023-01-13.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br /> Ce travail est sous licence : <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.

## Comment contribuer ? {-}

**bookdown** rend l'édition d'un livre aussi facile que l'édition d'un wiki, à condition d'avoir un compte GitHub ([inscription sur github.com](https://github.com/join)).
Une fois connecté à GitHub, cliquez sur l'icône "Edit this page" dans le panneau de droite du site Web du livre.
Vous accéderez ainsi à une version modifiable du fichier source [R Markdown](http://rmarkdown.rstudio.com/) qui a généré la page sur laquelle vous vous trouvez.

<!--[![](figures/editme.png)](https://github.com/Robinlovelace/geocompr/edit/main/index.Rmd)-->

Pour signaler un problème concernant le contenu du livre (par exemple, un code qui ne fonctionne pas) ou pour demander une fonctionnalité, consultez la [liste des issues] (https://github.com/Robinlovelace/geocompr/issues).

Les mainteneurs et les contributeurs doivent suivre les règles de conduite de ce dépôt : [CODE OF CONDUCT](https://github.com/Robinlovelace/geocompr/blob/main/CODE_OF_CONDUCT.md).

## Reproducibilité {-}

Le moyen le plus rapide de reproduire le contenu du livre si vous êtes novice en matière de données géographiques avec R peut être via le navigateur web, grâce à [Binder](https://mybinder.org/).
En cliquant sur le lien ci-dessous, vous ouvrirez une nouvelle fenêtre contenant RStudio Server dans votre navigateur Web, ce qui vous permettra d'ouvrir les fichiers du chapitre et d'exécuter des morceaux de code pour vérifier que le code est reproductible.

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/geocompr/fr/main?urlpath=rstudio)

Si vous voyez quelque chose comme l'image ci-dessous, félicitations, cela a fonctionné et vous pouvez commencer à explorer la géocomputation avec R dans un environnement cloud.(tout en étant conscient des consignes d'utilisation de [mybinder.org](https://mybinder.readthedocs.io/en/latest/about/user-guidelines.html)):

<!-- ![](https://user-images.githubusercontent.com/1825120/134802314-6dd368c7-f5eb-4cd7-b8ff-428dfa93954c.png) -->


<div class="figure" style="text-align: center">
<img src="https://user-images.githubusercontent.com/1825120/134802314-6dd368c7-f5eb-4cd7-b8ff-428dfa93954c.png" alt="Capture d'écran du code reproductible contenu dans Geocomputation avec R s'exécutant dans RStudio Server sur un navigateur servi par Binder." width="100%" />
<p class="caption">(\#fig:index-2-4)Capture d'écran du code reproductible contenu dans Geocomputation avec R s'exécutant dans RStudio Server sur un navigateur servi par Binder.</p>
</div>


Pour reproduire le code du livre sur votre propre ordinateur, vous avez besoin d'une version récente de [R](https://cran.r-project.org/) et des paquets à jour.
Ils peuvent être installés en utilisant le paquet [**remotes**](https://github.com/r-lib/remotes).


```r
install.packages("remotes")
remotes::install_github("geocompr/geocompkg")
```




Après avoir installé les dépendances du livre, vous pouvez reconstruire le livre à des fins de test et d'enseignement.
Pour ce faire, vous devez [télécharger](https://github.com/Robinlovelace/geocompr/archive/refs/heads/main.zip) et déziper ou [cloner](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) le code source du libre.
Après avoir ouvert le projet `geocompr.Rproj` dans RStudio (ou ouvert le dossier dans un autre IDE tel que [VS Code](https://github.com/REditorSupport/vscode-R)), vous devriez être en mesure de reproduire le contenu avec la commande suivante :


```r
bookdown::serve_book(".")
```



La page [GitHub](https://github.com/robinlovelace/geocompr#reproducing-the-book) contient plus de détail pour reproduire ce livre.

##  Soutenir le projet {-}

Si vous trouvez ce livre utile (sa traduction ou sa version originale), vous pouvez le soutenir :

- En en parlant à d'autres personnes
- En communiquant sur le livre dans les réseaux sociaux, par exemple, via [#geocompr hashtag](https://twitter.com/hashtag/geocompr) sur Twitter (cf. notre [**Guestbook** sur geocompr.github.io](https://geocompr.github.io/guestbook/)) ou en référençant les [cours](https://github.com/geocompr/geocompr.github.io/edit/source/content/guestbook/index.md) mobilisant le livre
- [Le citant](https://github.com/Robinlovelace/geocompr/raw/main/CITATION.bib) ou  [le metre en lien](https://geocompr.robinlovelace.net/)
- '[Starring](https://help.github.com/articles/about-stars/)' le [dépôt GitHub geocompr](https://github.com/robinlovelace/geocompr)
- Ecrivant un recension, sur Amazon ou [Goodreads](https://www.goodreads.com/book/show/42780859-geocomputation-with-r)
- Posant des questions ou faire des suggestions de contenus sur [GitHub](https://github.com/Robinlovelace/geocompr/issues/372) ou Twitter.
- [Achetant](https://www.amazon.com/Geocomputation-R-Robin-Lovelace-dp-0367670577/dp/0367670577) une copie

Plus de détails sur le [github.com/Robinlovelace/geocompr](https://github.com/Robinlovelace/geocompr#geocomputation-with-r).

<hr>

L'icône du globe utilisé en couverture de ce livre a été créé par [Jean-Marc Viglino](https://github.com/Viglino) et est sous licence [CC-BY 4.0 International](https://github.com/Viglino/font-gis/blob/main/LICENSE-CC-BY.md).

<a href="https://www.netlify.com"><img src="https://www.netlify.com/img/global/badges/netlify-color-accent.svg"/></a>



# Foreword (1st Edition) {-}

Doing 'spatial' in R has always been about being broad, seeking to provide and integrate tools from geography, geoinformatics, geocomputation and spatial statistics for anyone interested in joining in: joining in asking interesting questions, contributing fruitful research questions, and writing and improving code.
That is, doing 'spatial' in R has always included open source code, open data and reproducibility.

Doing 'spatial' in R has also sought to be open to interaction with many branches of applied spatial data analysis, and also to implement new advances in data representation and methods of analysis to expose them to cross-disciplinary scrutiny. 
As this book demonstrates, there are often alternative workflows from similar data to similar results, and we may learn from comparisons with how others create and understand their workflows.
This includes learning from similar communities around Open Source GIS and complementary languages such as Python, Java and so on.

R's wide range of spatial capabilities would never have evolved without people willing to share what they were creating or adapting.
This might include teaching materials, software, research practices (reproducible research, open data), and combinations of these. 
R users have also benefitted greatly from 'upstream' open source geo libraries such as GDAL, GEOS and PROJ.

This book is a clear example that, if you are curious and willing to join in, you can find things that need doing and that match your aptitudes.
With advances in data representation and workflow alternatives, and ever increasing numbers of new users often without applied quantitative command-line exposure, a book of this kind has really been needed.
Despite the effort involved, the authors have supported each other in pressing forward to publication.

So, this fresh book is ready to go; its authors have tried it out during many tutorials and workshops, so readers and instructors will be able to benefit from knowing that the contents have been and continue to be tried out on people like them.
Engage with the authors and the wider R-spatial community, see value in having more choice in building your workflows and most important, enjoy applying what you learn here to things you care about.

Roger Bivand

Bergen, September 2018

# Preface {-}

## Who this book is for {-}

This book is for people who want to analyze, visualize and model geographic data with open source software.
It is based on R, a statistical programming language that has powerful data processing, visualization and geospatial capabilities.
The book covers a wide range of topics and will be of interest to a wide range of people from many different backgrounds, especially:

- People who have learned spatial analysis skills using a desktop Geographic Information System (GIS), such as [QGIS](http://qgis.org/en/site/), [ArcGIS](http://desktop.arcgis.com/en/arcmap/), [GRASS](https://grass.osgeo.org/) or [SAGA](http://www.saga-gis.org/en/index.html), who want access to a powerful (geo)statistical and visualization programming language and the benefits of a command-line approach [@sherman_desktop_2008]:

  > With the advent of 'modern' GIS software, most people want to point and click their way through life. That’s good, but there is a tremendous amount of flexibility and power waiting for you with the command line.

- Graduate students and researchers from fields specializing in geographic data including Geography, Remote Sensing, Planning, GIS and Geographic Data Science
- Academics and post-graduate students working with geographic data --- in fields such as Geology, Regional Science, Biology and Ecology, Agricultural Sciences, Archaeology, Epidemiology, Transport Modeling, and broadly defined Data Science --- who require the power and flexibility of R for their research
- Applied researchers and analysts in public, private or third-sector organizations who need the reproducibility, speed and flexibility of a command-line language such as R in applications dealing with spatial data as diverse as Urban and Transport Planning, Logistics, Geo-marketing (store location analysis) and Emergency Planning

The book is designed for intermediate-to-advanced R users interested in geocomputation and R beginners who have prior experience with geographic data.
If you are new to both R and geographic data, do not be discouraged: we provide links to further materials and describe the nature of spatial data from a beginner's perspective in Chapter \@ref(spatial-class) and in links provided below.

## How to read this book {-}

The book is divided into three parts:

1. Part I: Foundations, aimed at getting you up-to-speed with geographic data in R.
2. Part II: Extensions, which covers advanced techniques.
3. Part III: Applications, to real-world problems.

The chapters get progressively harder in each so we recommend reading the book in order.
A major barrier to geographical analysis in R is its steep learning curve.
The chapters in Part I aim to address this by providing reproducible code on simple datasets that should ease the process of getting started.

An important aspect of the book from a teaching/learning perspective is the **exercises** at the end of each chapter.
Completing these will develop your skills and equip you with the confidence needed to tackle a range of geospatial problems.
Solutions to the exercises, and a number of extended examples, are provided on the book's supporting website, at [geocompr.github.io](https://geocompr.github.io/).

Impatient readers are welcome to dive straight into the practical examples, starting in Chapter \@ref(spatial-class).
However, we recommend reading about the wider context of *Geocomputation with R* in Chapter \@ref(intro) first.
If you are new to R, we also recommend learning more about the language before attempting to run the code chunks provided in each chapter (unless you're reading the book for an understanding of the concepts).
Fortunately for R beginners R has a supportive community that has developed a wealth of resources that can help.
We particularly recommend three tutorials:  [R for Data Science](http://r4ds.had.co.nz/) [@grolemund_r_2016] and [Efficient R Programming](https://csgillespie.github.io/efficientR/) [@gillespie_efficient_2016], especially [Chapter 2](https://csgillespie.github.io/efficientR/set-up.html#r-version) (on installing and setting-up R/RStudio) and [Chapter 10](https://csgillespie.github.io/efficientR/learning.html) (on learning to learn), and  [An introduction to R](http://colinfay.me/intro-to-r/) [@rcoreteam_introduction_2021].

## Why R? {-}

Although R has a steep learning curve, the command-line approach advocated in this book can quickly pay off.
As you'll learn in subsequent chapters, R is an effective tool for tackling a wide range of geographic data challenges.
We expect that, with practice, R will become the program of choice in your geospatial toolbox for many applications.
Typing and executing commands at the command-line is, in many cases, faster than pointing-and-clicking around the graphical user interface (GUI) of a desktop GIS.
For some applications such as Spatial Statistics and modeling R may be the *only* realistic way to get the work done.

As outlined in Section \@ref(why-use-r-for-geocomputation), there are many reasons for using R for geocomputation:
R is well-suited to the interactive use required in many geographic data analysis workflows compared with other languages.
R excels in the rapidly growing fields of Data Science (which includes data carpentry, statistical learning techniques and data visualization) and Big Data (via efficient interfaces to databases and distributed computing systems).
Furthermore R enables a reproducible workflow: sharing scripts underlying your analysis will allow others to build-on your work.
To ensure reproducibility in this book we have made its source code available at [github.com/Robinlovelace/geocompr](https://github.com/Robinlovelace/geocompr#geocomputation-with-r).
There you will find script files in the `code/` folder that generate figures:
when code generating a figure is not provided in the main text of the book, the name of the script file that generated it is provided in the caption (see for example the caption for Figure \@ref(fig:zones)).

Other languages such as Python, Java and C++ can be used for geocomputation and there are excellent resources for learning geocomputation *without R*, as discussed in Section \@ref(software-for-geocomputation).
None of these provide the unique combination of package ecosystem, statistical capabilities, visualization options, powerful IDEs offered by the R community.
Furthermore, by teaching how to use one language (R) in depth, this book will equip you with the concepts and confidence needed to do geocomputation in other languages.

## Real-world impact {-}

*Geocomputation with R* will equip you with knowledge and skills to tackle a wide range of issues, including those with scientific, societal and environmental implications, manifested in geographic data.
As described in Section \@ref(what-is-geocomputation), geocomputation is not only about using computers to process geographic data:
it is also about real-world impact.
If you are interested in the wider context and motivations behind this book, read on; these are covered in Chapter \@ref(intro)..

## Acknowledgements {-}

