Ulabox Segmentation
================

Appel des librairies
--------------------

``` r
library(statsr)
library("stats")
library(tidyverse)
```

    ## Warning: package 'tidyverse' was built under R version 3.4.4

    ## -- Attaching packages ------------------------------------------------------------------- tidyverse 1.2.1 --

    ## v ggplot2 2.2.1     v purrr   0.2.4
    ## v tibble  1.4.2     v dplyr   0.7.4
    ## v tidyr   0.8.1     v stringr 1.2.0
    ## v readr   1.1.1     v forcats 0.3.0

    ## Warning: package 'tibble' was built under R version 3.4.4

    ## Warning: package 'tidyr' was built under R version 3.4.4

    ## Warning: package 'forcats' was built under R version 3.4.4

    ## -- Conflicts ---------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

Import des données :

``` r
ulabox_orders_with_categories_partials_2017 <- read_csv("ulabox_orders_with_categories_partials_2017.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   customer = col_integer(),
    ##   order = col_integer(),
    ##   total_items = col_integer(),
    ##   `discount%` = col_double(),
    ##   weekday = col_integer(),
    ##   hour = col_integer(),
    ##   `Food%` = col_double(),
    ##   `Fresh%` = col_double(),
    ##   `Drinks%` = col_double(),
    ##   `Home%` = col_double(),
    ##   `Beauty%` = col_double(),
    ##   `Health%` = col_double(),
    ##   `Baby%` = col_double(),
    ##   `Pets%` = col_double()
    ## )

Création de la fonction mode :

``` r
getmode <- function(v) {
   uniqv <- unique(v)
   uniqv[which.max(tabulate(match(v, uniqv)))]
}
```

Tri des données par client
--------------------------

Afin de segmenter la base de données en différents profils clients, les données sont regroupées par ID client et non plus par achats.

Les ID clients sont enregistrés comme noms de lignes et non comme colonne à part entière envue de a segmentation :

``` r
remove_rownames(sommaire)
```

    ## # A tibble: 10,239 x 11
    ##    customer items discount  jour heure  food fresh drink  home beaute
    ##       <int> <dbl>    <dbl> <int> <int> <dbl> <dbl> <dbl> <dbl>  <dbl>
    ##  1        0  44.7    14.1      4    13 14.1   73.2  4.36  6.2   2.18 
    ##  2        1  31.2    17.8      1    11 17.8   52.9 17.8   3.21  2.31 
    ##  3        2  26       2.97     6    23 24.1   22.3 38.7  14.9   0    
    ##  4        3  27.8     4.10     1    12 23.8   51.3  8.22 14.8   0    
    ##  5        4  17.1     4.37     1    12 24.8   51.1 10.3  13.0   0.684
    ##  6        5  21      11.8      2    23  6.84   0   24.0  26.9  10.2  
    ##  7        6  40.8     2.68     1    12 39.3   23.8 19.7   5.63  2.19 
    ##  8        7  40       0.93     1    20 23.7   52.5 13.9   0     9.83 
    ##  9        8  35.2    23.7      1    12 24.1   37.6 22.4  12.0   3.92 
    ## 10        9  17.8     3.00     1     8  9.30  64.8 14.2   3.26  2.44 
    ## # ... with 10,229 more rows, and 1 more variable: sante <dbl>

``` r
sommaire_bis <- column_to_rownames(sommaire, var = "customer")
```

    ## Warning: Setting row names on a tibble is deprecated.

Détermination du nombre de clusters
-----------------------------------

Détermination automatique du nombre de clusters optimal en utilisant l'algorithme de segmentation k-means et en calculant le "With-In-Sum-Of-Squares (WSS)" pour chaque nombre de clusters.

``` r
library(factoextra)
```

    ## Warning: package 'factoextra' was built under R version 3.4.4

    ## Welcome! Related Books: `Practical Guide To Cluster Analysis in R` at https://goo.gl/13EFCZ

``` r
fviz_nbclust(sommaire_bis, kmeans, method = "wss") +
    geom_vline(xintercept = 4, linetype = 2)
```

![](segmentation_files/figure-markdown_github/unnamed-chunk-4-1.png)

Le nombre optimal de clusters est de 4.

On lance donc maintenant l'algorithme de segmentation k-means avec ce nombre de clusters :

``` r
set.seed(123)
km.res <- kmeans(sommaire_bis, 4, nstart = 25)
```

Voici la description des différents sous-groupes après segmentation :

``` r
segments <- aggregate(sommaire_bis, by=list(cluster=km.res$cluster), mean)

segments
```

    ##   cluster     items  discount     jour    heure      food      fresh
    ## 1       1 22.610390  8.932410 3.639959 15.18466 16.460171  3.8517951
    ## 2       2 43.506989  6.300423 3.692567 15.66748 25.767951 36.8069476
    ## 3       3 38.362375 55.679746 3.603667 15.01975 92.484909  0.6354741
    ## 4       4  5.406142  7.535662 2.988764 13.62022  1.949765  0.1855449
    ##       drink      home    beaute     sante
    ## 1 28.964459 19.528271  5.215726 1.4556143
    ## 2 17.164971 10.378696  4.607907 1.0751331
    ## 3  2.863220  1.989396  1.012610 0.3460243
    ## 4  0.859418  2.572805 90.518727 1.0694551

On crée une table reprenant la liste des clients mais en ajoutant le cluster auquel ils appartiennent dans une nouvelle colonne :

``` r
segmentation <- cbind(sommaire_bis, cluster = km.res$cluster)
```

Export des différents clusters :

``` r
write.csv(segments, file = "segments.csv", row.names = FALSE)
```
