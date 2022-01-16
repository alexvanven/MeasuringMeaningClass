Measuring Meaning in Mixed Methods - Week 2
================

Week 2 Network analysis Session 1
=================================

Topics:
- whole network measures
- centrality measures
- finding subgroups

Importing data
--------------

The following 3 steps are the same as last session. Just to be sure, we do them here again.

We are going to use the igraph package in R to analyze the campnet dataset discussed in Borgatti. The dataset (campnet.csv) and attribute file (campattr.txt) can be downloaded from last week's Canvas module. For more information on the data, see: <https://sites.google.com/site/ucinetsoftware/datasets/campdata>

``` r
#last session we installed igraph so no need to do that again
#install.packages("igraph")
library(igraph)
```

    ## 
    ## Attaching package: 'igraph'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     decompose, spectrum

    ## The following object is masked from 'package:base':
    ## 
    ##     union

``` r
#be sure that the two files are in your R working directory and in the Data folder 
#header = False because we don't have a name to the two columns
campnet <- read.csv2("Data/campnet.csv", header=FALSE)
#the attribute file contains information on the nodes such as their gender, their role. It also already includes a centrality measure (betweenness) but this will be calculated separately later on in the script. 
#import the attribute file
campattr <- read.csv("Data/campattr.txt")
```

We need to transform the data frame format into an igraph object to be able to use igraph's functions. We also add the attributes.

``` r
#as this is a directed network we set directed to TRUE
g <- graph_from_data_frame(campnet, directed = TRUE, vertices = campattr)
#look at the file format
class(g)
```

    ## [1] "igraph"

``` r
#now has become an igraph format
```

Basic network analysis
----------------------

The graph representation of the network.

``` r
#we can plot the network with the kamada-kawai layout. 
#We replicate the figure 2.3 in borgatti which shows the directed network in graph format
plot(g,edge.arrow.size=.4,layout=layout_with_kk,main="campnet dataset")
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-3-1.png)

``` r
#replicate figure 9.3 with gender attribute
plot(g,edge.arrow.size=.4,layout=layout_with_kk,main="campnet dataset with gender",vertex.color=vertex_attr(g)$Gender)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-3-2.png)

We count the number of nodes and edges.

``` r
#we can  count the number of nodes
vcount(g)
```

    ## [1] 18

``` r
#and count the number of edges
ecount(g)
```

    ## [1] 54

We also make an undirected version of the network by "collapsing" the matrix and making it symmetric.

``` r
#Change into an undirected network by collapsing
g_undirected <- as.undirected(g, mode = "collapse")
plot(g_undirected,edge.arrow.size=.4,layout=layout_with_kk,main="campnet dataset undirected")
#if you look at the matrix representation you see that it has become symmetric
g_undirected[]
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-5-1.png)

    ## 18 x 18 sparse Matrix of class "dgCMatrix"

    ##    [[ suppressing 18 column names 'HOLLY', 'BRAZEY', 'CAROL' ... ]]

    ##                                            
    ## HOLLY   . . . 1 1 . . . 1 . . 1 . 1 . . . .
    ## BRAZEY  . . . . . . . . . . 1 . . . . 1 1 .
    ## CAROL   . . . 1 1 . 1 . . . . . . . . . . .
    ## PAM     1 . 1 . . 1 1 1 . . . . . . . . . .
    ## PAT     1 . 1 . . 1 1 . . . . . . . . . . .
    ## JENNIE  . . . 1 1 . . 1 . . . . . . . . . .
    ## PAULINE . . 1 1 1 . . 1 . . . . 1 . . . . .
    ## ANN     . . . 1 . 1 1 . . . . . . . . . . .
    ## MICHAEL 1 . . . . . . . . 1 . 1 . 1 1 . . .
    ## BILL    . . . . . . . . 1 . . 1 . 1 . . . .
    ## LEE     . 1 . . . . . . . . . . . . . 1 1 .
    ## DON     1 . . . . . . . 1 1 . . . 1 . . . .
    ## JOHN    . . . . . . 1 . . . . . . . 1 . . 1
    ## HARRY   1 . . . . . . . 1 1 . 1 . . . . . .
    ## GERY    . . . . . . . . 1 . . . 1 . . 1 . 1
    ## STEVE   . 1 . . . . . . . . 1 . . . 1 . 1 1
    ## BERT    . 1 . . . . . . . . 1 . . . . 1 . 1
    ## RUSS    . . . . . . . . . . . . 1 . 1 1 1 .

Paths, trails, walks
--------------------

A key concept in graph theory is the notion of a path. This refers to the "route" that connects two nodes in a network. A path is a particular route, namely one that does not revisit the same node or edge twice. Paths are therefore different from "walks" and "trails". In "walks" both nodes and edges can be repeated.

In the network below, 1-&gt;2-&gt;3-&gt;4-&gt;2-&gt;1-&gt;3 is a walk.

<img src="Images/walks.png" alt="Walks" style="width:50.0%" />

Trails can revisit the same node, but not the same edge. So here, 1-&gt;3-&gt;8-&gt;6-&gt;3-&gt;2 is a trail.

<img src="Images/trails.png" alt="Trails" style="width:50.0%" />

As said, a path never revisits the same nodes or edges. So 6-&gt;8-&gt;3-&gt;1-&gt;2-&gt;4 is a path.

<img src="Images/paths.png" alt="Paths" style="width:50.0%" />

The shortest path between nodes is called a "geodesic".

To get the paths from Holly to the other nodes in the campnet network, we can list all paths from this node to all others. You can verify that, as mentioned by Borgatti (p.16), there is no path in the network from Holly to Brazey.

``` r
all_simple_paths(g, "HOLLY", to = V(g), mode = c("out"))
```

We can get the geodesic distances between pairs of nodes (cf. matrix 2.2 in Borgatti). We take the directed graph and therefore add mode=out and mode=in.

``` r
distances(g,mode="out")
```

    ##         HOLLY BRAZEY CAROL PAM PAT JENNIE PAULINE ANN MICHAEL BILL LEE DON JOHN
    ## HOLLY       0    Inf     2   1   1      2       2   2       2  Inf Inf   1  Inf
    ## BRAZEY      5      0     7   6   6      7       7   7       4  Inf   1   5  Inf
    ## CAROL       2    Inf     0   1   1      2       1   2       4  Inf Inf   3  Inf
    ## PAM         3    Inf     2   0   2      1       1   1       5  Inf Inf   4  Inf
    ## PAT         1    Inf     1   2   0      1       2   2       3  Inf Inf   2  Inf
    ## JENNIE      2    Inf     2   1   1      0       2   1       4  Inf Inf   3  Inf
    ## PAULINE     2    Inf     1   1   1      2       0   2       4  Inf Inf   3  Inf
    ## ANN         3    Inf     2   1   2      1       1   0       5  Inf Inf   4  Inf
    ## MICHAEL     1    Inf     3   2   2      3       3   3       0  Inf Inf   1  Inf
    ## BILL        2    Inf     4   3   3      4       4   4       1    0 Inf   1  Inf
    ## LEE         5      1     7   6   6      7       7   7       4  Inf   0   5  Inf
    ## DON         1    Inf     3   2   2      3       3   3       1  Inf Inf   0  Inf
    ## JOHN        3      4     2   2   2      3       1   3       2  Inf   3   3    0
    ## HARRY       1    Inf     3   2   2      3       3   3       1  Inf Inf   1  Inf
    ## GERY        2      3     4   3   3      4       4   4       1  Inf   2   2  Inf
    ## STEVE       4      2     6   5   5      6       6   6       3  Inf   1   4  Inf
    ## BERT        4      2     6   5   5      6       6   6       3  Inf   1   4  Inf
    ## RUSS        3      3     5   4   4      5       5   5       2  Inf   2   3  Inf
    ##         HARRY GERY STEVE BERT RUSS
    ## HOLLY       2  Inf   Inf  Inf  Inf
    ## BRAZEY      5    3     1    1    2
    ## CAROL       4  Inf   Inf  Inf  Inf
    ## PAM         5  Inf   Inf  Inf  Inf
    ## PAT         3  Inf   Inf  Inf  Inf
    ## JENNIE      4  Inf   Inf  Inf  Inf
    ## PAULINE     4  Inf   Inf  Inf  Inf
    ## ANN         5  Inf   Inf  Inf  Inf
    ## MICHAEL     1  Inf   Inf  Inf  Inf
    ## BILL        1  Inf   Inf  Inf  Inf
    ## LEE         5    3     1    1    2
    ## DON         1  Inf   Inf  Inf  Inf
    ## JOHN        3    1     2    2    1
    ## HARRY       0  Inf   Inf  Inf  Inf
    ## GERY        2    0     1    2    1
    ## STEVE       4    2     0    1    1
    ## BERT        4    2     1    0    1
    ## RUSS        3    1     1    1    0

``` r
distances(g,mode="in")
```

    ##         HOLLY BRAZEY CAROL PAM PAT JENNIE PAULINE ANN MICHAEL BILL LEE DON JOHN
    ## HOLLY       0      5     2   3   1      2       2   3       1    2   5   1    3
    ## BRAZEY    Inf      0   Inf Inf Inf    Inf     Inf Inf     Inf  Inf   1 Inf    4
    ## CAROL       2      7     0   2   1      2       1   2       3    4   7   3    2
    ## PAM         1      6     1   0   2      1       1   1       2    3   6   2    2
    ## PAT         1      6     1   2   0      1       1   2       2    3   6   2    2
    ## JENNIE      2      7     2   1   1      0       2   1       3    4   7   3    3
    ## PAULINE     2      7     1   1   2      2       0   1       3    4   7   3    1
    ## ANN         2      7     2   1   2      1       2   0       3    4   7   3    3
    ## MICHAEL     2      4     4   5   3      4       4   5       0    1   4   1    2
    ## BILL      Inf    Inf   Inf Inf Inf    Inf     Inf Inf     Inf    0 Inf Inf  Inf
    ## LEE       Inf      1   Inf Inf Inf    Inf     Inf Inf     Inf  Inf   0 Inf    3
    ## DON         1      5     3   4   2      3       3   4       1    1   5   0    3
    ## JOHN      Inf    Inf   Inf Inf Inf    Inf     Inf Inf     Inf  Inf Inf Inf    0
    ## HARRY       2      5     4   5   3      4       4   5       1    1   5   1    3
    ## GERY      Inf      3   Inf Inf Inf    Inf     Inf Inf     Inf  Inf   3 Inf    1
    ## STEVE     Inf      1   Inf Inf Inf    Inf     Inf Inf     Inf  Inf   1 Inf    2
    ## BERT      Inf      1   Inf Inf Inf    Inf     Inf Inf     Inf  Inf   1 Inf    2
    ## RUSS      Inf      2   Inf Inf Inf    Inf     Inf Inf     Inf  Inf   2 Inf    1
    ##         HARRY GERY STEVE BERT RUSS
    ## HOLLY       1    2     4    4    3
    ## BRAZEY    Inf    3     2    2    3
    ## CAROL       3    4     6    6    5
    ## PAM         2    3     5    5    4
    ## PAT         2    3     5    5    4
    ## JENNIE      3    4     6    6    5
    ## PAULINE     3    4     6    6    5
    ## ANN         3    4     6    6    5
    ## MICHAEL     1    1     3    3    2
    ## BILL      Inf  Inf   Inf  Inf  Inf
    ## LEE       Inf    2     1    1    2
    ## DON         1    2     4    4    3
    ## JOHN      Inf  Inf   Inf  Inf  Inf
    ## HARRY       0    2     4    4    3
    ## GERY      Inf    0     2    2    1
    ## STEVE     Inf    1     0    1    1
    ## BERT      Inf    2     1    0    1
    ## RUSS      Inf    1     1    1    0

``` r
#if you do not define the directionality for a directed graph then it automatically assumes that the graph is undirected.
distances(g)
```

    ##         HOLLY BRAZEY CAROL PAM PAT JENNIE PAULINE ANN MICHAEL BILL LEE DON JOHN
    ## HOLLY       0      4     2   1   1      2       2   2       1    2   4   1    3
    ## BRAZEY      4      0     5   5   5      6       4   5       3    4   1   4    3
    ## CAROL       2      5     0   1   1      2       1   2       3    4   5   3    2
    ## PAM         1      5     1   0   2      1       1   1       2    3   5   2    2
    ## PAT         1      5     1   2   0      1       1   2       2    3   5   2    2
    ## JENNIE      2      6     2   1   1      0       2   1       3    4   6   3    3
    ## PAULINE     2      4     1   1   1      2       0   1       3    4   4   3    1
    ## ANN         2      5     2   1   2      1       1   0       3    4   5   3    2
    ## MICHAEL     1      3     3   2   2      3       3   3       0    1   3   1    2
    ## BILL        2      4     4   3   3      4       4   4       1    0   4   1    3
    ## LEE         4      1     5   5   5      6       4   5       3    4   0   4    3
    ## DON         1      4     3   2   2      3       3   3       1    1   4   0    3
    ## JOHN        3      3     2   2   2      3       1   2       2    3   3   3    0
    ## HARRY       1      4     3   2   2      3       3   3       1    1   4   1    3
    ## GERY        2      2     3   3   3      4       2   3       1    2   2   2    1
    ## STEVE       3      1     4   4   4      5       3   4       2    3   1   3    2
    ## BERT        4      1     4   4   4      5       3   4       3    4   1   4    2
    ## RUSS        3      2     3   3   3      4       2   3       2    3   2   3    1
    ##         HARRY GERY STEVE BERT RUSS
    ## HOLLY       1    2     3    4    3
    ## BRAZEY      4    2     1    1    2
    ## CAROL       3    3     4    4    3
    ## PAM         2    3     4    4    3
    ## PAT         2    3     4    4    3
    ## JENNIE      3    4     5    5    4
    ## PAULINE     3    2     3    3    2
    ## ANN         3    3     4    4    3
    ## MICHAEL     1    1     2    3    2
    ## BILL        1    2     3    4    3
    ## LEE         4    2     1    1    2
    ## DON         1    2     3    4    3
    ## JOHN        3    1     2    2    1
    ## HARRY       0    2     3    4    3
    ## GERY        2    0     1    2    1
    ## STEVE       3    1     0    1    1
    ## BERT        4    2     1    0    1
    ## RUSS        3    1     1    1    0

Components
----------

The concept of paths is important for defining another key concept in describing networks: components.

A component is defined as the maximal set of nodes in which every node can reach every other node by some path.

A strong component takes the directionality into account. Weak components ignore directionality.

``` r
#find the number of strong components.
g.components <- components(g, mode = "strong")
print(g.components)
```

    ## $membership
    ##   HOLLY  BRAZEY   CAROL     PAM     PAT  JENNIE PAULINE     ANN MICHAEL    BILL 
    ##       4       3       4       4       4       4       4       4       4       2 
    ##     LEE     DON    JOHN   HARRY    GERY   STEVE    BERT    RUSS 
    ##       3       4       1       4       3       3       3       3 
    ## 
    ## $csize
    ## [1]  1  1  6 10
    ## 
    ## $no
    ## [1] 4

``` r
#the following line does the same thing but now just returns the number
count_components(g, mode = c("strong"))
```

    ## [1] 4

``` r
#add attributes of component membership
V(g)$components <- g.components$membership

#examine attributes to check if it was added
vertex_attr(g)
```

    ## $name
    ##  [1] "HOLLY"   "BRAZEY"  "CAROL"   "PAM"     "PAT"     "JENNIE"  "PAULINE"
    ##  [8] "ANN"     "MICHAEL" "BILL"    "LEE"     "DON"     "JOHN"    "HARRY"  
    ## [15] "GERY"    "STEVE"   "BERT"    "RUSS"   
    ## 
    ## $Gender
    ##  [1] 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2
    ## 
    ## $Role
    ##  [1] 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2
    ## 
    ## $Betweenness
    ##  [1] 78  0  1 33 40  6 13  1 59  0  5 16  0  2 55 17 14 47
    ## 
    ## $components
    ##  [1] 4 3 4 4 4 4 4 4 4 2 3 4 1 4 3 3 3 3

``` r
#plot the graph with components
plot(g,edge.arrow.size=.4,layout=layout_with_kk,main="campnet dataset with components",vertex.color=vertex_attr(g)$components)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-8-1.png)

Check what happens when you choose "weak" components. How many components do you get?

Whole Network Measures
======================

Whole network measures measure a property of the whole network. Whole network measures thus give one number that characterizes the whole network.

Two basic classes of whole-networks are *cohesion* measures and *shape* measures. Cohesion measures assess the general connectivity of the network. Shape measures see whether the network approximates a certain shape, for example, exhibits "clumpiness", resembles a "star" or "core-periphery" network.

Density
-------

Density is a measure for the "cohesion" of the network. It is simply the number of ties divided by the number of possible ties.

In an undirected graph with no self-loops (i.e. unreflexive) the number of possible ties is n\*(n-1)/2. (So only one side of the matrix, excluding the diagonal).

For a directed network, we would divide the number of ties by n\*(n-1)

``` r
#measure the density. 
graph.density(g_undirected)
#verify:
ecount(g_undirected)/((vcount(g_undirected)*(vcount(g_undirected)-1))/2)

graph.density(g)
#verify:
ecount(g)/(vcount(g)*(vcount(g)-1))
```

Which values for density do you find for the directed and undirected network.

Density can vary across subgroups. To measure the density among men and women in the campnet data, we extract both networks based on this attribute and calculate the density.

``` r
#first extract the womens network
g.women <- induced_subgraph(g, V(g)$Gender == 1)
#we can also plot only the women in the network
plot(g.women,edge.arrow.size=.2,layout=layout_with_kk,main="campnet dataset women",vertex.color=vertex_attr(g.women)$Gender)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-10-1.png)

``` r
#density among women. cf. Table 9.1
graph.density(g.women)
#density among men. cf. Table 9.1
g.men <- induced_subgraph(g, V(g)$Gender == 2)
plot(g.men,edge.arrow.size=.2,layout=layout_with_kk,main="campnet dataset men",vertex.color=vertex_attr(g.men)$Gender)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-10-2.png)

``` r
graph.density(g.men)
```

Which density values do you find for the male and female subgraph? Compare them with table 9.1

Component ratio
---------------

The number of components in a network is also a measure of the whole network structure. If there are many components then this indicates that the network is "disconnected". Just taking the number of components, however, makes it hard to compare different networks on this measure. Therefore we normalize this measure by dividing it by the number of nodes in the network, or more accurately: C-1 / n-1 (where C is the number of components, and n is the number of nodes). It is 1 when all nodes are isolates and 0 when there is just one component.

``` r
component_ratio <- (count_components(g,mode="strong")-1)/(vcount(g)-1)
component_ratio
```

What value do you find?

Connectedness
-------------

This measures the proportion of the pairs of nodes that are located in the same component. So, for a network of size n, connectedness sums up the number of (non-reflexive) pairs that can reach each other, so are within the same component, and divides it by the total number of possible pairs, which is n\*n-1.

``` r
#Connectedness is not available in igraph. But we can calculate it "by hand".
#We assign the size of the different components to a vector csize
csize <- g.components[[2]]
#We then calculate the number of pairs within each component. In this case, we have four components of size 1, 1, 6 and 10. So there are zero pairs in the first two components, 5*6=30 pairs the third component, and 9*10=90 pairs in the fourth component. 
connectedness_value <- sum((csize-1)*csize)/(vcount(g)*(vcount(g)-1))
connectedness_value
```

What value do you find?

Compactness
-----------

Compactness measures the number of paths that exist within a network, weighs them by their path length, and then takes the average by dividing by the number of possible pairs in the network.

``` r
#Compactness
#Compactness is not available in igraph. 
#But we can calculate it by defining a function:
Compactness <- function(x) {
  gra.geo <- distances(x,mode="out") ## get geodesics. for a directed network we have to set the mode. 
  gra.rdist <- 1/gra.geo  ## get reciprocal of geodesics
  diag(gra.rdist) <- NA   ## assign NA to diagonal
  gra.rdist[gra.rdist == Inf] <- 0 ## replace infinity with 0
  # Compactness = mean of reciprocal distances
  comp.igph <- mean(gra.rdist, na.rm=TRUE) 
  return(comp.igph)
}
Compactness(g)
```

What value do you find?

Reciprocity
-----------

In directed networks, we can measure whether ties are reciprocated (or "symmetric"). Note that 0/0 pairs are also reciprocal ties (both nodes not having a tie with each other). If we do not want to include 0/0 pairs as reciprocated then mode is ratio.

``` r
#Reciprocity
reciprocity(g,ignore.loops = TRUE, mode="ratio")
```

What value do you find?

Diameter
--------

A final measure of the cohesiveness of a network is the diameter of the network. This gives the longest shortest path. This thus indicates the "breadth" of the network or how long it takes to get from the two most distant points in the network to each other. You can also find this by looking at the maximal value in the distance matrix.

``` r
#Diameter
diameter(g)
```

What value do you find? And can you find a path that has the largest shortest path in this network?

Transitivity
------------

In mathematics, we call a relation "transitive" when A-&gt;B and B-&gt;C implies A-&gt;C. A "greater then" relation is for example transitive. If B is bigger than A, and C is bigger than B, then logically C is also bigger than A.

In network relations, transitivity refers to the existence of closed triads. If A is friends with B, and B is friends with C, then this relation is transitive when A also is friends with C.

By counting the number of closed triads in a network we can get a sense of how "clumpy" the network is. Duncan Watts and Steven Strogatz have proposed a measure for this "clumpiness" in a network. This measure is for undirected networks.

It starts by measuring the density of ties in each node's ego network (the density of ties among nodes connected to a given node). This is called the individual clustering coefficient.

Below we plot the ego network of Holly. She is connected to 5 nodes.

``` r
ego.holly <- induced.subgraph(g_undirected, neighborhood(g_undirected, 1, 1)[[1]])
plot(ego.holly)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-16-1.png)

To calculate the individual clustering coefficient, we take the number of existing ties among those 5 nodes (in this case 3) and divide it by the number of possible ties between those 5 nodes (5x4/2=10). So the density is 3/10.

For Brazey, we can see that all her "alters" (the nodes she is connected to) are also connected to each other, so her individual clustering coefficient would be 1.

``` r
ego.brazey <- induced.subgraph(g_undirected, neighborhood(g_undirected, 1, 2)[[1]])
plot(ego.brazey)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-17-1.png)

The overall clustering coefficient then just does this for all nodes and takes the average individual clustering coefficient. The overall *weighted* clustering coefficient weighs the individual clustering coefficient by the number of pairs in each node's ego network (so instead of just taking the average across the individual clustering coefficients, we, for example, weigh the value of Holly by 10 and that of Brazey by 3 when calculating the average across all nodes).

In Igraph we can calculate this using the function "transitivity".

``` r
#Transitivity 
#overall clustering coefficient
transitivity(g_undirected,type='average')
#weighted overall clustering coefficient 
transitivity(g_undirected)
```

What values do you find for the overall clustering coefficient and the weighted overall clustering coefficient?

Centralization
--------------

Centralization refers to the extent that a network is dominated by a single node. A maximally centralized network looks like a star: a node in the center has ties to all other nodes, and no other ties exist. (See figure 9.7 in Borgatti).

``` r
#Centralization
#Calculations for the example from figure 9.8 in Borgatti
C=matrix(c(0,1,1,0,0,1,0,1,0,0,1,1,0,1,1,0,0,1,0,1,0,0,1,1,0),nrow=5,ncol=5) 
print(C)
```

    ##      [,1] [,2] [,3] [,4] [,5]
    ## [1,]    0    1    1    0    0
    ## [2,]    1    0    1    0    0
    ## [3,]    1    1    0    1    1
    ## [4,]    0    0    1    0    1
    ## [5,]    0    0    1    1    0

``` r
c <- graph_from_adjacency_matrix(C, mode="undirected")
plot(c)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-19-1.png)

``` r
degree(c)
```

    ## [1] 2 2 4 2 2

``` r
centr_degree(c,mode="all",loops=FALSE,normalized=TRUE)
```

    ## $res
    ## [1] 2 2 4 2 2
    ## 
    ## $centralization
    ## [1] 0.6666667
    ## 
    ## $theoretical_max
    ## [1] 12

``` r
#applied to campnet data
centr_degree(g_undirected,loops=FALSE,normalized = TRUE)
```

    ## $res
    ##  [1] 5 3 3 5 4 3 5 3 5 3 3 4 3 4 4 5 4 4
    ## 
    ## $centralization
    ## [1] 0.07352941
    ## 
    ## $theoretical_max
    ## [1] 272

Centrality Measures
===================

Whereas whole network measures capture a property of the whole network, centrality measures are a variable at the level of the nodes.

We include here centrality measures for undirected, non-valued networks (cf. 10.3 in Borgatti).

Degree centrality
-----------------

Degree centrality takes the number of ties of each node. Nodes with many ties are considered more central to the network.

``` r
# Degree centrality
deg <- degree(g_undirected)
print(deg)
```

    ##   HOLLY  BRAZEY   CAROL     PAM     PAT  JENNIE PAULINE     ANN MICHAEL    BILL 
    ##       5       3       3       5       4       3       5       3       5       3 
    ##     LEE     DON    JOHN   HARRY    GERY   STEVE    BERT    RUSS 
    ##       3       4       3       4       4       5       4       4

``` r
plot(g_undirected,edge.arrow.size=.4,layout=layout_with_kk,main="campnet dataset size=degree",vertex.size=deg*5)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-20-1.png)

Eigenvector centrality
----------------------

Eigenvector centrality has the special quality that it not only takes into account the degree of the node in question, but also the degree of the nodes that that node is connected to, and the degree of the nodes that those nodes are connected to, etc. It thus takes the *global* structure of the whole network into account when calculating the centrality of each node. A node that has relatively low degree could therefore still be highly central globally when that node *is connected* to central nodes.

``` r
#Eigenvector centrality
ev_g_un <- evcent(g_undirected)
#print the vector with eigenvector centrality measure
ev_g_un$vector
#print the largest eigenvalue
ev_g_un$value
#plot
plot(g_undirected,layout=layout_with_kk,main="campnet dataset size=eigenvector centrality",vertex.size=ev_g_un$vector*20)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-21-1.png)

Which node has the largest eigenvector centrality?

We will not discuss the mathematics of this operation now. For an explanation of the intuition behind eigenvector centrality, see <http://djjr-courses.wikidot.com/soc180:eigenvector-centrality> It suffices to say that it is called *eigenvector* centrality because finding centrality scores for nodes based on the centrality scores of the nodes that that node is connected to, is the same problem as finding the eigenvector and eigenvalues of a square matrix. This is illustrated by the code below which manually calculates the eigenvector centrality score by doing an eigenanalysis on the matrix.

``` r
g_matrix <- as_adjacency_matrix(g_undirected)
#get the eigenvectors and eigenvalues of this adjacency matrix
e <- eigen(g_matrix)
#print the eigenvalues
e$values
```

    ##  [1]  4.078841e+00  3.612768e+00  3.323262e+00  1.872646e+00  8.434542e-01
    ##  [6]  4.550483e-01  4.440892e-16 -1.478454e-01 -2.570428e-01 -8.630123e-01
    ## [11] -1.000000e+00 -1.000000e+00 -1.000000e+00 -1.304000e+00 -1.698954e+00
    ## [16] -1.936554e+00 -2.116148e+00 -2.862464e+00

``` r
#we take the largest eigenvalue
max(e$values)
```

    ## [1] 4.078841

``` r
#we take the eigenvectors that correspond to the largest eigenvalue.
eigenvectors <- e$vectors
#these are in the first column. they are all negative so we multiply by -1 to make them positive. that makes no difference to the results. 
eigenvector <- eigenvectors[,1]*-1
#to get the same output above we scale the values to the largest value (in this case for the first node)
eigenvector/max(eigenvector)
```

    ##  [1] 1.0000000 0.2575818 0.5230920 0.7762326 0.6594487 0.4687665 0.6979279
    ##  [8] 0.4763429 0.9533345 0.6480174 0.2575818 0.8449127 0.4116208 0.8449127
    ## [15] 0.5506573 0.4507376 0.3423158 0.4303505

Beta centrality or Bonacich power centrality
--------------------------------------------

Beta centrality encompasses both degree and eigenvector centrality. Using a parameter Beta, we can make the measure closer to degree centrality or eigenvector centrality.

When the beta factor is 0 then you get degree. When it gets close to 1/largest eigenvector then it becomes eigenvector centrality.

The function in igraph normalizes the scores on this measure so that the sum of squared scores equals the number of nodes of the network.

This should be taken into account when comparing the results with the output of degree centrality and eigenvector centrality.

``` r
#Beta centrality or Bonacich power centrality

#when b = 0 then this equals degree centrality
beta_centrality <- power_centrality(g_undirected, exponent=0)
print(beta_centrality)
```

    ##     HOLLY    BRAZEY     CAROL       PAM       PAT    JENNIE   PAULINE       ANN 
    ## 1.2587720 0.7552632 0.7552632 1.2587720 1.0070176 0.7552632 1.2587720 0.7552632 
    ##   MICHAEL      BILL       LEE       DON      JOHN     HARRY      GERY     STEVE 
    ## 1.2587720 0.7552632 0.7552632 1.0070176 0.7552632 1.0070176 1.0070176 1.2587720 
    ##      BERT      RUSS 
    ## 1.0070176 1.0070176

``` r
#check that it has indeed been scaled to the number of nodes
sum(beta_centrality^2)
```

    ## [1] 18

``` r
plot(g_undirected,layout=layout_with_kk,main="campnet dataset size=beta centrality b=0",vertex.size=beta_centrality*20)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-23-1.png)

Verify by hand that the differences between nodes are the same as for degree.

``` r
#we take a value of beta that approximates 1/largest eigenvalue. the largest eigenvalue can be found using the eigenvector centrality function evcent. above it was, for example, given by ev_g_un$value
#you can also calculate it yourself
g_matrix <- as_adjacency_matrix(g_undirected)
e <- eigen(g_matrix)
e$values
```

    ##  [1]  4.078841e+00  3.612768e+00  3.323262e+00  1.872646e+00  8.434542e-01
    ##  [6]  4.550483e-01  4.440892e-16 -1.478454e-01 -2.570428e-01 -8.630123e-01
    ## [11] -1.000000e+00 -1.000000e+00 -1.000000e+00 -1.304000e+00 -1.698954e+00
    ## [16] -1.936554e+00 -2.116148e+00 -2.862464e+00

``` r
max(e$values)
```

    ## [1] 4.078841

``` r
1/max(e$values)
```

    ## [1] 0.2451677

``` r
#with beta at its highest, beta centrality should approximate eigenvector centrality
beta_centrality <- power_centrality(g_undirected, exponent=0.245)
print(beta_centrality)
```

    ##     HOLLY    BRAZEY     CAROL       PAM       PAT    JENNIE   PAULINE       ANN 
    ## 1.5895977 0.4120202 0.8320411 1.2346034 1.0487711 0.7456173 1.1105185 0.7577684 
    ##   MICHAEL      BILL       LEE       DON      JOHN     HARRY      GERY     STEVE 
    ## 1.5155865 1.0298128 0.4120202 1.3427690 0.6558899 1.3427690 0.8774641 0.7201598 
    ##      BERT      RUSS 
    ## 0.5473423 0.6869261

``` r
#check that it has indeed been scaled to the number of nodes
sum(beta_centrality^2)
```

    ## [1] 18

``` r
plot(g_undirected,layout=layout_with_kk,main="campnet dataset size=beta centrality b=0.245",vertex.size=beta_centrality*10)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-24-1.png)

Closeness centrality
--------------------

Closeness centrality measures how many steps are required to access every other node from a given node. It is based on the sum of the geodesic distances from one node to all others. This value can be found by summing across the rows in the distance matrix. It is then normalized by dividing the minimum number when a node would have direct ties to everyone (which would be equal to n-1) by the sum of actual distances. In the campnet data, for Holly, this would for example be 17/38.

``` r
closeness_g_un <- closeness(g_undirected, normalized=TRUE)
print(closeness_g_un)
plot(g_undirected,layout=layout_with_kk,main="campnet dataset size=closeness centrality",vertex.size=closeness_g_un*50)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-25-1.png)

Which node has the highest closeness centrality?

Betweenness centrality
----------------------

Betweenness centrality measures the number of shortest paths going through a specific vertex. It is calculated by computing for all nodes except the focal node, what proportion of all the shortest paths from one to another pass through the focal node. These proportions are summed across all pairs and the result is a single value for each node in the network.

``` r
#Betweenness centrality measures the number of shortest paths going through a specific vertex
betweenness_g_un <- betweenness(g_undirected)
print(betweenness_g_un)
plot(g_undirected,layout=layout_with_kk,main="campnet dataset size=betweenness centrality",vertex.size=betweenness_g_un)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-26-1.png)

Which node has the highest betweenness centrality?

Finding subgroups
=================

One important problem in network analysis is to find cohesive subgroups. Which nodes are more strongly connected to each other, rather than to others?

One important approach to finding cohesive subgroups is the Girvan-Newman algorithm. This approach tries to find structurally important edges which, if they are removed, would fragment the network. The idea is that these edges exist between cohesive groups and not within them. The removal of these edges will just leave just the cohesive groups.

The algorithm starts by calculating the edge betweenness of all edges. Edge betweenness is defined here in a similar way to node betweenness. Edge betweenness is a count of the number of times an edge lies on a geodesic path between a pair of vertices. Hence, as in the vertex case, we take all pairs of vertices and simply count in the same way the number of times each edge is part of a geodesic path. If we delete the edge with the highest score, we will either increase the number of components or increase the fragmentation. If we iteratively repeat this process, the number of components will continue to increase until we are only left with isolates.

As the algorithm iterates, we obtain different partitions. These do not increase necessarily by 1 each time, as there may be more than one edge with the highest score. In addition, the algorithm makes an assessment of how good each partition is in terms of a numerical score called ‘modularity’, denoted by Q. Modularity compares the number of internal links in the groups to how many you would expect to see if they were distributed at random. Higher values mean that the algorithm has found more significant groupings. Negative values are possible, indicating that the groups are less cohesive than a purely random assignment.

Below we use the Girvan-Newman approach to edge\_betweenness on the undirected campnet data.

``` r
community <- cluster_edge_betweenness(g_undirected)
```

The algorithm has calculated the modularity scores and chosen the one with the maximal value. In this case, 0.55. It then gives the, in this case, 3 groups, which can also be plotted.

``` r
community
```

    ## IGRAPH clustering edge betweenness, groups: 3, mod: 0.55
    ## + groups:
    ##   $`1`
    ##   [1] "HOLLY"   "MICHAEL" "BILL"    "DON"     "HARRY"  
    ##   
    ##   $`2`
    ##   [1] "BRAZEY" "LEE"    "JOHN"   "GERY"   "STEVE"  "BERT"   "RUSS"  
    ##   
    ##   $`3`
    ##   [1] "CAROL"   "PAM"     "PAT"     "JENNIE"  "PAULINE" "ANN"    
    ## 

``` r
membership(community)
```

    ##   HOLLY  BRAZEY   CAROL     PAM     PAT  JENNIE PAULINE     ANN MICHAEL    BILL 
    ##       1       2       3       3       3       3       3       3       1       1 
    ##     LEE     DON    JOHN   HARRY    GERY   STEVE    BERT    RUSS 
    ##       2       1       2       1       2       2       2       2

``` r
plot(community,g_undirected)
```

![](Week-2_files/figure-markdown_github/unnamed-chunk-28-1.png)
