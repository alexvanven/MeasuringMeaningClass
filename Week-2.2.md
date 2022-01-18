Measuring Meaning in Mixed Methods - Week 2
================

Week 2 Session 2
================

Structural equivalence
----------------------

As discussed by Mohr (1998: 358) there is a debate within network analysis between those who focus on methods emphasizing "connectivity" (as in finding cohesive subgroups) and those who emphasize "structural equivalence". In this section, we'll discuss the idea behind structural equivalence and two ways of finding structurally equivalent nodes: measuring profile similarity (Borgatti 12.3) and using the Concor algorithm (not discussed in Borgatti but used in Mohr 1994). There are other techniques, such as direct optimization (12.5) and REGE for regular equivalence. We will not discuss those in any detail.

Structural equivalence suggests that nodes are not similar because they are connected to each other (as in cohesive subgroups) but because they have similar relations *to others*. Nodes are similar (or "equivalent") because they have the same "position" or "role" within the network. A teacher, for example, occupies the role of a teacher because she has a characteristic relation to her students, and students occupy the role of students because of their relation to their teacher. The dominant class, in Marxist theory, is a class because of its relation to the dominated class, and vice versa, not necessarily because of their internal cohesiveness. Their relation vis-a-vis each other, in other words, is what makes them occupy a certain role in the social structure. Network analysts have developed various ways to find structurally equivalent roles in social networks. And these also prove useful for measuring "discursive roles" (cf. Mohr 1994).

In an undirected network without self-reflexive ties, the definition of structural equivalence is that actor i and j are structurally equivalent if, excepting each other, they are connected to exactly the same other actors.

This definition is an ideal mathematical model and does not often occur in real data. So we need ways to find structurally equivalent nodes that approximate this model.

Profile similarity
------------------

In a one-mode, adjacency matrix, two nodes are structurally equivalent when they have exactly the same relation to all others in the network. In matrix terms this means that their row (or column) *profiles* are exactly the same.

To see this, look at the matrix of the fictitious data of the first relationship in Figure 12.1. The following steps create the matrix, the plot and output the adjacency matrix.

``` r
g <-  graph.formula(1-1,1-2,1-3,1-4,1-5,2-2,2-3,2-4,2-5,simplify = FALSE)
l <- layout.kamada.kawai(g) 
l[,1] <- c(2,4,1,3,5)
l[,2] <- c(3,3,1,1,1)
plot(g,layout=l)
```

![](Week-2.2_files/figure-markdown_github/unnamed-chunk-1-1.png)

``` r
adj <- as_adjacency_matrix(g,sparse=FALSE)
adj
```

    ##   1 2 3 4 5
    ## 1 1 1 1 1 1
    ## 2 1 1 1 1 1
    ## 3 1 1 0 0 0
    ## 4 1 1 0 0 0
    ## 5 1 1 0 0 0

What you see is that the row profiles (i.e. the complete row of each node) are exactly the same for the structurally equivalent nodes (1,2 and 3,4,5). This is also what the definition of structural equivalence implies. Node 1 and 2 have exactly the same relations with nodes 3,4 and 5. Nodes 3,4 and 5 have the exact same relation to nodes 1 and 2.

To measure whether we have structurally equivalent nodes, we can therefore measure the *profile similarity* between rows (and columns if the network is directed) and thereby assess whether two rows have similar relations to others in the network.

To measure profile similarity we can use various similarity or distance measures. Below we briefly review a few commonly used measures.

Measuring profile similarity using similarity or distance measures in R
-----------------------------------------------------------------------

To illustrate the measures for row profiles, I am using an example from a fictious two-mode matrix, a document-word matrix. The principle is the same for measuring the row profiles of nodes in a one-mode matrix.

### Euclidian distance

Euclidian distance measures the distances between rows using the Pythagoran theorem. For a quick refresher on high school geometry look here: <https://towardsdatascience.com/the-euclidean-distance-is-just-the-pythagorean-theorem-2e672017d875> Measuring the distance between rows gives a row by row distance matrix. If the Euclidian distance is zero, then two row profiles are the same.

<img src="Images/Distance%20measures.png" alt="Euclidian distance" style="width:50.0%" />

We measure the Euclidian distance with the dist function:

``` r
#distance and similarity measures
s1 <- c(5,5,2,1)
s2 <- c(3,3,5,1)
s3 <- c(2,2,6,1)
s4 <- c(4,4,4,1)
my_m <- rbind(s1,s2,s3,s4)
#we can use the function dist which is standard in R. The default is euclidian distance
dist(my_m)
```

    ##          s1       s2       s3
    ## s2 4.123106                  
    ## s3 5.830952 1.732051         
    ## s4 2.449490 1.732051 3.464102

### Correlation

We can also measure the correlation between rows as an indication of their similarity. It varies between -1 and 1 and the more similar two row profiles, the closer to 1.

A useful feature of the correlation coefficient is that it normalizes the frequency of occurrence. To see what that means, let's change the example somewhat and change the difference between document 2 and 3 so that document 3 has 10 more of each word, but the overall pattern is the same. If we then calculate the Euclidian distances, this will be large between document 2 and 3. But correlation will be perfect.

Another similarity measure which has this feature is cosine similarity (which is therefore often used in text analysis). I'll discuss that later in the course.

``` r
#distance and similarity measures
s1 <- c(5,5,2,1)
s2 <- c(3,3,5,1)
s3 <- c(13,13,15,11)
s4 <- c(4,4,4,1)
my_m <- rbind(s1,s2,s3,s4)
#euclidian distance
dist(my_m)
```

    ##           s1        s2        s3
    ## s2  4.123106                    
    ## s3 19.924859 20.000000          
    ## s4  2.449490  1.732051 19.570386

``` r
#cor calculates correlation among columns so transpose matrix first to get correlation among rows
cor(t(my_m))
```

    ##           s1        s2        s3        s4
    ## s1 1.0000000 0.1980295 0.1980295 0.7276069
    ## s2 0.1980295 1.0000000 1.0000000 0.8164966
    ## s3 0.1980295 1.0000000 1.0000000 0.8164966
    ## s4 0.7276069 0.8164966 0.8164966 1.0000000

### Jaccard similarity

The Jaccard measure of similarity is used for binary values. The special characteristic of the Jaccard measure is that it does not take into account values where both rows are zero. If two documents both do not have some word, then this does not add to the similarity of the documents. Dissimilarity only increases when one document has a word that the other one does not have. This is different from a correlation measure which would take these "mutual absences", or 0/0 cases into account. The Jaccard measure of similarity is measured as follows. Document 1 and 2, for example, shared one word (W2) out of the three words that both documents have (W1,W2,W3). Both do not have W4 so we do not include that one in the denominator. The dist function in R gives us 1-this value (so it transforms it into a distance measure rather than a similarity measure. So the smaller the Jaccard similarity, the larger the distance).

<img src="Images/Distance%20measures_jaccard.png" alt="Jaccard" style="width:50.0%" />

``` r
#example for jaccard. dist measures distances so to get the similarity score  you would subtract it from 1. binary=1-jaccard
e <- c(1,1,0,0)
f <- c(0,1,1,0)
g <- c(0,0,0,1)
h <- c(1,1,1,1)
matrix_jaccard <- rbind(e,f,g,h)
dist(matrix_jaccard,method="binary")
```

    ##           e         f         g
    ## f 0.6666667                    
    ## g 1.0000000 1.0000000          
    ## h 0.5000000 0.5000000 0.7500000

When you have measured the distance (or similarity) between row profiles, you can use this information to group together structurally equivalent nodes, i.e. nodes that score low on distance (or high on similarity). Next week, we'll discuss how to do this using cluster analysis and/or MDS. For now it is enough to understand that by measuring profile similarity you have measured (approximately) structurally equivalent nodes.

Blockmodeling
-------------

Once we have identified groups of structurally equivalent nodes, we can produce a simplified or reduced matrix. We rearrange the matrix so that structurally equivalent nodes (so the rows and columns of the adjacency matrix) are grouped together. In the ideal case, all blocks are either zeros or ones, and we can therefore replace each block with a zero or one without losing information. The result is a new and smaller adjacency matrix which we call the *image matrix*. This process of simplifying the matrix based on groups of structurally equivalent nodes is called blockmodeling.

We will illustrate this process using the Sampson dataset as used by Borgatti in 12.4.

We are going to use two other R-packages, sna and statnet, because the capabilities for structural equivalence are somewhat limited in igraph. Because of some additional options in Ucinet (the program used by Borgatti) which are not straightforwardly implemented in R, the replication will not be exact, but we can approximate their results. We'll try to replicate figure 12.4 which gives the blockmodel based on dichotomized data and using correlation as a measure of profile similarity.

``` r
#install.packages("sna")
#install.packages("statnet")
detach("package:igraph", unload=TRUE)#this detaches the igraph package as it might interfere with the other packages.
library(sna)
library(statnet)
```

We load in the Sampson data. It has two relations and is split in esteem and disesteem relations.

``` r
#import the sampson esteem data
sampe <- read.csv("Data/sampson_esteem.csv", header=TRUE,row.names = 1)
sampd <- read.csv("Data/sampson_desteem.csv", header=TRUE,row.names = 1)
```

We dichotomize both esteem and disesteem relationships.

``` r
sampe_dichot <- sampe
sampe_dichot[sampe_dichot>0] <- 1
#same for disesteem
sampd_dichot <- sampd
sampd_dichot[sampd_dichot>0] <- 1
```

We use correlation as our similarity measure. The equiv.cluster function in sna does the similarity calculation and clustering in one function.

We cluster the nodes using the average hierarchical cluster algorithm (which we will discuss next week).

The measure of profile similarity is based on the two relations at the same time. So we list both the esteem and disesteem matrix.

``` r
eq <- equiv.clust(list(sampe_dichot,sampd_dichot), diag=TRUE, mode="digraph", method="correlation",cluster.method = "average")
class(eq)
```

    ## [1] "equiv.clust"

``` r
plot(eq)
```

![](Week-2.2_files/figure-markdown_github/unnamed-chunk-8-1.png)

The cluster dendogram gives you the groupings of the nodes based on their profile similarity. Nodes that have high similarity are grouped together more quickly (such as nodes 17 and 18).

The blockmodel function calculates the image matrix and gives a plot of the clustered nodes. The blockmodel does not exactly replicate Borgatti's analysis, as block 2 also contains node 5 (Peter). But it comes fairly close. In any case, the blockmodel now shows a simplified structure, where, for example, none of the groups show any esteem for block 4.

``` r
blk_mod <- sna::blockmodel(sampe_dichot, eq, k=4, eq$cluster, mode="digraph") 
blk_mod
```

    ## 
    ## Network Blockmodel:
    ## 
    ## Block membership:
    ## 
    ##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 
    ##  1  1  1  1  2  2  1  3  3  3  3  3  3  3  2  4  4  4 
    ## 
    ## Reduced form blockmodel:
    ## 
    ##   
    ##           Block 1   Block 2    Block 3 Block 4
    ## Block 1 0.2000000 0.4000000 0.02857143       0
    ## Block 2 0.3333333 0.3333333 0.04761905       0
    ## Block 3 0.1142857 0.0000000 0.47619048       0
    ## Block 4 0.0000000 0.2222222 0.14285714       1

``` r
blk_mod$glabels <- "Esteem"
blk_mod$plabels <- blk_mod$order.vector
plot(blk_mod)
```

![](Week-2.2_files/figure-markdown_github/unnamed-chunk-9-1.png)

``` r
blk_mod <- sna::blockmodel(sampd_dichot, eq, k=4, eq$cluster, mode="digraph") 
blk_mod$glabels <- "Disesteem"
blk_mod$plabels <- blk_mod$order.vector
plot(blk_mod)
```

![](Week-2.2_files/figure-markdown_github/unnamed-chunk-10-1.png)

``` r
blk_mod
```

    ## 
    ## Network Blockmodel:
    ## 
    ## Block membership:
    ## 
    ##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 
    ##  1  1  1  1  2  2  1  3  3  3  3  3  3  3  2  4  4  4 
    ## 
    ## Reduced form blockmodel:
    ## 
    ##   Disesteem 
    ##            Block 1    Block 2    Block 3   Block 4
    ## Block 1 0.05000000 0.06666667 0.05714286 0.6000000
    ## Block 2 0.06666667 0.00000000 0.33333333 0.2222222
    ## Block 3 0.05714286 0.42857143 0.04761905 0.5714286
    ## Block 4 0.40000000 0.33333333 0.04761905 0.0000000

CONCOR
------

The most prominent examples of the use of blockmodeling (at least in sociology) make use of the CONCOR algorithm (as first developed by Harrison White and Ron Breiger). The name CONCOR refers to the CONvergence of iterated CORrelations. As the name suggests, it uses a simple trick to find structurally equivalent groups. Above we measured the similarity between rows using correlation. The result is a similarity matrix with the correlation coefficients. Now what CONCOR does is to take the correlations of those correlations, and repeating that process a number of times. If you repeat it long enough, it will (in most cases) converge and have only 1s and -1s in the matrix. The cells with 1s are then one group, and the cells with -1 another. This process can be repeated for the two blocks, which will split it into 4 blocks, and so on. CONCOR has therefore many similarities with the steps that we took above when first measuring similarities among row profiles with correlations, and then using a hierarchical cluster analysis to select the number of blocks. CONCOR has some limitations, though, as it can only work with correlations as a measure of similarity. It also always splits up the blocks into divisions of 2, which is an artifact of the algorithm, and does not necessarily have to be descriptive of the network structure. That said, a basic understanding of the algorithm is necessary for following the Mohr 1994 article.

John Mohr's Soldiers, Mothers, Tramps and Others
------------------------------------------------

The 1994 article is the first of several within his project on the cultural logic of the field of welfare organizations around the turn of the century.

It coded the directory descriptions for the occurrence of status identities (soldiers, strangers, the blind, etc.) and the categories of relief and social welfare practices. Table 1 and 2 list both of them and provide examples from the actual textual material. Table 3 is the raw two-mode data matrix of practices by identities.

The next step is measuring the profile similarities among the status identities using the correlation coefficient. This thus gives a status x status matrix of similarities.

Then he measures how similar those status identities are in relation to other status identities. Again by taking the correlation we find which status identities are similar in their relation vis-a-vis other status identities.

Mohr gives the example of seamen and widows. These have, for example, relatively low correlation in terms of their practice profiles. But they have quite similar positions in relation to *other* status identities. In other words, in the overall relations among status identities, these two identities seemed to have held a quite similar position.

The Concor algorithm executes this logic of correlations of correlations of correlations, etc. so it is useful to find the overall block structure of how status identities occupied similar discursive role positions.

Here we will replicate the analysis done by John Mohr in Soldiers, Mothers, Tramps and Others.

We first import the matrix of table 3.

``` r
#we import the mohr soldiers data
soldiers <- read.csv2("Data/mohr_matrix_soldiers.txt", header=TRUE,row.names = 1)
```

We take the correlation among the columns. This will measure the column similarity of status identies in how they are treated.

``` r
soldiers_cor <- cor(soldiers)
```

Just by looking at the raw matrix (Table 3), we can already see that, for example, the two identities TRAMP\_F and TRAMP\_M have quite unique and similar treatment profiles. Both are characterized by GIVEWOR\_S and JAIL\_S and TRAMP\_F is given JOBTRAN\_S. So they will correlate strongly with each other.

``` r
cor(soldiers$TRAMP_F,soldiers$TRAMP_M)
```

    ## [1] 0.8091736

The correlation is indeed 0.81. You can also see that they are quite unique in their treatment profile, as the three treatments are not present for any other category. We can check by looking at the full correlation profiles. Correlations with other status identities are indeed all negative.

``` r
soldiers_cor["TRAMP_F",]
```

    ##     BLIND_M    BLIND_NG     BLIND_F   CONSUM_NG  DISABLE_NG    EXCON_NG 
    ## -0.07173421 -0.13794015 -0.08652532 -0.06356417 -0.04413674 -0.07173421 
    ##    HISTAT_M   HISTAT_NG    HISTAT_F   IMMIGN_NG    IMMIGN_F     MOTHERS 
    ## -0.04413674 -0.04413674 -0.07173421 -0.15666989 -0.08652532 -0.11298654 
    ##      SEAMEN    SOLDIERS   STRANG_NG     TRAMP_M     TRAMP_F    UNEMPL_M 
    ## -0.10660036 -0.08652532 -0.04413674  0.80917359  1.00000000 -0.07173421 
    ##   UNEMPL_NG    UNEMPL_F   UNWEDMOTH      WIDOWS    WORKBOYS   WORKGIRLS 
    ## -0.06356417 -0.03093441 -0.05454545 -0.09341987 -0.08652532 -0.10009272 
    ##     WORKMEN   WORKWOMEN 
    ## -0.03093441 -0.08652532

``` r
soldiers_cor["TRAMP_M",]
```

    ##     BLIND_M    BLIND_NG     BLIND_F   CONSUM_NG  DISABLE_NG    EXCON_NG 
    ## -0.05804543 -0.11161752 -0.07001400 -0.05143445 -0.03571429 -0.05804543 
    ##    HISTAT_M   HISTAT_NG    HISTAT_F   IMMIGN_NG    IMMIGN_F     MOTHERS 
    ## -0.03571429 -0.03571429 -0.05804543 -0.12677314 -0.07001400 -0.09142572 
    ##      SEAMEN    SOLDIERS   STRANG_NG     TRAMP_M     TRAMP_F    UNEMPL_M 
    ## -0.08625819 -0.07001400 -0.03571429  1.00000000  0.80917359 -0.05804543 
    ##   UNEMPL_NG    UNEMPL_F   UNWEDMOTH      WIDOWS    WORKBOYS   WORKGIRLS 
    ## -0.05143445 -0.02503131 -0.04413674 -0.07559289 -0.07001400 -0.08099239 
    ##     WORKMEN   WORKWOMEN 
    ## -0.02503131 -0.07001400

Two status identities do not have to have high pairwise correlation to be structurally equivalent. The example is given of SEAMEN and WIDOWS.

``` r
cor(soldiers$SEAMEN,soldiers$WIDOWS)
```

    ## [1] 0.2145247

The correlation is relatively low with 0.21. But as we'll see, they will be classified as similar based on having similar patterns of correlation with *other* status identities.

We can check by correlating their correlations. The correlation of their correlation profiles is already higher at 0.32. And after a few iterations, the correlation becomes almost 1.

``` r
seamen <- soldiers_cor[,"SEAMEN"]
widows <- soldiers_cor[,"WIDOWS"]
cor(seamen,widows)
```

    ## [1] 0.3175255

``` r
#and only after a few iterations it is already 0.91
cor(cor(cor(cor(soldiers_cor))))["SEAMEN",]
```

    ##     BLIND_M    BLIND_NG     BLIND_F   CONSUM_NG  DISABLE_NG    EXCON_NG 
    ## -0.79397213  0.96067929 -0.66895985  0.56848937  0.45687934  0.43867279 
    ##    HISTAT_M   HISTAT_NG    HISTAT_F   IMMIGN_NG    IMMIGN_F     MOTHERS 
    ##  0.14747206  0.14747206  0.26765427  0.86603362  0.16659298  0.14558174 
    ##      SEAMEN    SOLDIERS   STRANG_NG     TRAMP_M     TRAMP_F    UNEMPL_M 
    ##  1.00000000  0.86583712  0.30748619 -0.81156503 -0.81047909  0.38201337 
    ##   UNEMPL_NG    UNEMPL_F   UNWEDMOTH      WIDOWS    WORKBOYS   WORKGIRLS 
    ##  0.20997535  0.24707829  0.36404491  0.90899806 -0.08140380 -0.20286812 
    ##     WORKMEN   WORKWOMEN 
    ##  0.27963983 -0.08814707

The Concor algorithm implements this logic of correlations of correlations.

``` r
#Adam Slez has rewritten the Concor program in R. This is not available through CRAN but we can download it directly from github. We need the devtools package to do this. You need to uncomment the following two lines to install the package. 
#library(devtools)
#devtools::install_github("aslez/concoR") 
library(concoR)
#p=3 means that we look at a 3-level split, so 8 blocks. Which is what Mohr reports in his figure 1.
blks3 <- concor_hca(list(soldiers_cor),p=3) 
#the blocks were reverse coded so we their index numbers to match the article for easier interpretation
blks3$block <- 9-blks3$block
blks3
```

    ##    block     vertex
    ## 1      8    BLIND_M
    ## 12     4   BLIND_NG
    ## 2      8    BLIND_F
    ## 7      6  CONSUM_NG
    ## 22     2 DISABLE_NG
    ## 17     3   EXCON_NG
    ## 9      5   HISTAT_M
    ## 10     5  HISTAT_NG
    ## 11     5   HISTAT_F
    ## 13     4  IMMIGN_NG
    ## 18     3   IMMIGN_F
    ## 3      7    MOTHERS
    ## 14     4     SEAMEN
    ## 15     4   SOLDIERS
    ## 23     2  STRANG_NG
    ## 25     1    TRAMP_M
    ## 26     1    TRAMP_F
    ## 19     3   UNEMPL_M
    ## 20     3  UNEMPL_NG
    ## 21     3   UNEMPL_F
    ## 24     2  UNWEDMOTH
    ## 16     4     WIDOWS
    ## 4      7   WORKBOYS
    ## 5      7  WORKGIRLS
    ## 8      6    WORKMEN
    ## 6      7  WORKWOMEN

SEAMEN and WIDOWS are now grouped together in block 4 (together with BLIND\_NG IMMIGN\_NG SOLDIERS). This means that these status identities are structurally equivalent, i.e. have similar positions vis-a-vis other status identities. In other words, both relate in similar manners to other status identities. So even if they do not strongly correlate with each other, they do have a similar position in the overall discourse structure.

To further investigate the blockmodel solution we can also look at the groupings at lower splits. This should show the same divisions as in figure 1.

``` r
blks2 <- concor_hca(list(soldiers_cor),p=2) 
blks2
```

    ##    block     vertex
    ## 1      1    BLIND_M
    ## 12     3   BLIND_NG
    ## 2      1    BLIND_F
    ## 7      2  CONSUM_NG
    ## 22     4 DISABLE_NG
    ## 13     3   EXCON_NG
    ## 8      2   HISTAT_M
    ## 9      2  HISTAT_NG
    ## 10     2   HISTAT_F
    ## 14     3  IMMIGN_NG
    ## 15     3   IMMIGN_F
    ## 3      1    MOTHERS
    ## 16     3     SEAMEN
    ## 17     3   SOLDIERS
    ## 23     4  STRANG_NG
    ## 24     4    TRAMP_M
    ## 25     4    TRAMP_F
    ## 18     3   UNEMPL_M
    ## 19     3  UNEMPL_NG
    ## 20     3   UNEMPL_F
    ## 26     4  UNWEDMOTH
    ## 21     3     WIDOWS
    ## 4      1   WORKBOYS
    ## 5      1  WORKGIRLS
    ## 11     2    WORKMEN
    ## 6      1  WORKWOMEN

``` r
blks1 <- concor_hca(list(soldiers_cor),p=1) 
blks1
```

    ##    block     vertex
    ## 1      1    BLIND_M
    ## 12     2   BLIND_NG
    ## 2      1    BLIND_F
    ## 3      1  CONSUM_NG
    ## 13     2 DISABLE_NG
    ## 14     2   EXCON_NG
    ## 4      1   HISTAT_M
    ## 5      1  HISTAT_NG
    ## 6      1   HISTAT_F
    ## 15     2  IMMIGN_NG
    ## 16     2   IMMIGN_F
    ## 7      1    MOTHERS
    ## 17     2     SEAMEN
    ## 18     2   SOLDIERS
    ## 19     2  STRANG_NG
    ## 20     2    TRAMP_M
    ## 21     2    TRAMP_F
    ## 22     2   UNEMPL_M
    ## 23     2  UNEMPL_NG
    ## 24     2   UNEMPL_F
    ## 25     2  UNWEDMOTH
    ## 26     2     WIDOWS
    ## 8      1   WORKBOYS
    ## 9      1  WORKGIRLS
    ## 10     1    WORKMEN
    ## 11     1  WORKWOMEN

We can also show the block structure in figure 2.

``` r
library(sna)
blk_mod <- blockmodel(soldiers_cor, blks3$block, diag=TRUE,
                      glabels = names(soldiers_cor),
                      plabels = rownames(soldiers_cor[[1]]))
blk_mod
```

    ## 
    ## Network Blockmodel:
    ## 
    ## Block membership:
    ## 
    ##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 
    ##  8  4  8  6  2  3  5  5  5  4  3  7  4  4  2  1  1  3  3  3  2  4  7  7  6  7 
    ## 
    ## Reduced form blockmodel:
    ## 
    ##   1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 
    ##             Block 1      Block 2      Block 3     Block 4     Block 5
    ## Block 1  0.90458680 -0.043064042 -0.058706296 -0.10514113 -0.04824695
    ## Block 2 -0.04306404  0.610484380  0.006737715  0.15250511  0.07908077
    ## Block 3 -0.05870630  0.006737715  0.492418600  0.18590600  0.05105989
    ## Block 4 -0.10514113  0.152505114  0.185905999  0.50310553  0.20581618
    ## Block 5 -0.04824695  0.079080770  0.051059889  0.20581618  0.82901404
    ## Block 6 -0.04274109 -0.041238350 -0.011088347  0.33565808  0.43912514
    ## Block 7 -0.08732200 -0.046248058  0.036379215  0.12918545  0.21626965
    ## Block 8 -0.07157974 -0.029233952 -0.013916476 -0.02688315  0.05073917
    ##             Block 6     Block 7     Block 8
    ## Block 1 -0.04274109 -0.08732200 -0.07157974
    ## Block 2 -0.04123835 -0.04624806 -0.02923395
    ## Block 3 -0.01108835  0.03637921 -0.01391648
    ## Block 4  0.33565808  0.12918545 -0.02688315
    ## Block 5  0.43912514  0.21626965  0.05073917
    ## Block 6  0.74333213  0.16028070  0.04427764
    ## Block 7  0.16028070  0.57623420  0.08581134
    ## Block 8  0.04427764  0.08581134  0.91452730

``` r
densitymatrix <- blk_mod[["block.model"]]
```

``` r
#we cut off the density within each block at the average which is 0.1375
densitymatrix[densitymatrix>0.1375] <- 1
densitymatrix[densitymatrix<0.1375] <- 0
diag(densitymatrix) <- 0
densitymatrix
```

    ##         Block 1 Block 2 Block 3 Block 4 Block 5 Block 6 Block 7 Block 8
    ## Block 1       0       0       0       0       0       0       0       0
    ## Block 2       0       0       0       1       0       0       0       0
    ## Block 3       0       0       0       1       0       0       0       0
    ## Block 4       0       1       1       0       1       1       0       0
    ## Block 5       0       0       0       1       0       1       1       0
    ## Block 6       0       0       0       1       1       0       1       0
    ## Block 7       0       0       0       0       1       1       0       0
    ## Block 8       0       0       0       0       0       0       0       0

We now have a network of relations between the blocks. And we can use Igraph to plot this network.

``` r
library(igraph)
```

    ## 
    ## Attaching package: 'igraph'

    ## The following objects are masked from 'package:sna':
    ## 
    ##     betweenness, bonpow, closeness, components, degree, dyad.census,
    ##     evcent, hierarchy, is.connected, neighborhood, triad.census

    ## The following objects are masked from 'package:network':
    ## 
    ##     %c%, %s%, add.edges, add.vertices, delete.edges, delete.vertices,
    ##     get.edge.attribute, get.edges, get.vertex.attribute, is.bipartite,
    ##     is.directed, list.edge.attributes, list.vertex.attributes,
    ##     set.edge.attribute, set.vertex.attribute

    ## The following objects are masked from 'package:stats':
    ## 
    ##     decompose, spectrum

    ## The following object is masked from 'package:base':
    ## 
    ##     union

``` r
mohrnetwork <- graph_from_adjacency_matrix(densitymatrix)
plot(mohrnetwork,edge.arrow.size=.2,layout=layout_with_kk)
```

![](Week-2.2_files/figure-markdown_github/unnamed-chunk-21-1.png)

We now get an overall sense of the structure of relations among status identities. Block 4, which contains SOLDIERS, occupies the core position and links to 4 other blocks. Block 1 and Block 8 are isolates. Block 1 contain the two unique TRAMP categories. Block 7 contains MOTHERS.
