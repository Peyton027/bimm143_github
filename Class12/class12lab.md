# Class 12 Lab
Peyton Chiu (PID:18145937)

- [Background](#background)
- [Toy differential gene expression](#toy-differential-gene-expression)
- [Volcano plot](#volcano-plot)
- [Save our results](#save-our-results)

## Background

Today will analyze some RNASeq data from Himes et al. on the effects of
a common steroid (dexamethsaone) on airway smooth muscles cells

Our starting poin the “counts” data and “metadata” that contain the
count values for each gene in their different expiriments (cells lines
with or w/o the drug treatment)

``` r
counts <- read.csv("airway_scaledcounts.csv", row.names=1)
metadata <-  read.csv("airway_metadata.csv")
```

``` r
head(counts)
```

                    SRR1039508 SRR1039509 SRR1039512 SRR1039513 SRR1039516
    ENSG00000000003        723        486        904        445       1170
    ENSG00000000005          0          0          0          0          0
    ENSG00000000419        467        523        616        371        582
    ENSG00000000457        347        258        364        237        318
    ENSG00000000460         96         81         73         66        118
    ENSG00000000938          0          0          1          0          2
                    SRR1039517 SRR1039520 SRR1039521
    ENSG00000000003       1097        806        604
    ENSG00000000005          0          0          0
    ENSG00000000419        781        417        509
    ENSG00000000457        447        330        324
    ENSG00000000460         94        102         74
    ENSG00000000938          0          0          0

``` r
dim(counts)
```

    [1] 38694     8

``` r
nrow(counts)
```

    [1] 38694

> Q1. How many genes are in this dataset?

There are 38694 genes in the dataset

``` r
head(metadata)
```

              id     dex celltype     geo_id
    1 SRR1039508 control   N61311 GSM1275862
    2 SRR1039509 treated   N61311 GSM1275863
    3 SRR1039512 control  N052611 GSM1275866
    4 SRR1039513 treated  N052611 GSM1275867
    5 SRR1039516 control  N080611 GSM1275870
    6 SRR1039517 treated  N080611 GSM1275871

``` r
sum(metadata$dex =="control")
```

    [1] 4

> Q2. How many ‘control’ cell lines do we have?

There are 4 control lines

## Toy differential gene expression

To start ur analysis let’s caluclate the mean count countrs for all
genes in the “countrol” expiriments

1.Extract all “control” columns for the `counts` object 2. Calculate the
mean for all rows (genes) of thesse “control” columns

3-4: Repeat 1 and 2 for treatment 5. compare these mean values

``` r
control.inds <- metadata$dex == "control"
control.counts <- counts[,control.inds]
control.mean <-rowMeans(control.counts)
```

> Q3:How would you make the above code in either approach more robust?
> Is there a function that could help here?How would you make the above
> code in either approach more robust? Is there a function that could
> help here?

Yes, you can make it more robust by utilizing the rowMeans function
which reducess the total number of steps in your code while also
reducing the error if say you add another control or treated line.

> Q4. Follow the same procedure for the treated samples (i.e. calculate
> the mean per gene across drug treated samples and assign to a labeled
> vector called treated.mean)

``` r
treated.ids <- metadata$dex == "treated"
treated.counts <-counts[,treated.ids]
treated.mean <- rowMeans(treated.counts)
```

Store the mean values together for bookkeeping

``` r
meancounts <- data.frame(control.mean, treated.mean)
head(meancounts)
```

                    control.mean treated.mean
    ENSG00000000003       900.75       658.00
    ENSG00000000005         0.00         0.00
    ENSG00000000419       520.50       546.00
    ENSG00000000457       339.75       316.50
    ENSG00000000460        97.25        78.75
    ENSG00000000938         0.75         0.00

> Q5 (a). Create a scatter plot showing the mean of the treated samples
> against the mean of the control samples. Your plot should look
> something like the following.

``` r
meancounts <- data.frame(control.mean, treated.mean)
plot(meancounts)
```

![](class12lab_files/figure-commonmark/unnamed-chunk-9-1.png)

> Q5 (b).You could also use the ggplot2 package to make this figure
> producing the plot below. What geom\_?() function would you use for
> this plot?

``` r
library(ggplot2)
```

    Warning: package 'ggplot2' was built under R version 4.4.3

``` r
ggplot(meancounts, aes(control.mean, treated.mean))+
  geom_point() 
```

![](class12lab_files/figure-commonmark/unnamed-chunk-10-1.png)

Making this a log plot

``` r
library(ggplot2)
ggplot(meancounts, aes(control.mean, treated.mean))+
  geom_point() +
  scale_x_log10()+
  scale_y_log10()
```

    Warning in scale_x_log10(): log-10 transformation introduced infinite values.

    Warning in scale_y_log10(): log-10 transformation introduced infinite values.

![](class12lab_files/figure-commonmark/unnamed-chunk-11-1.png)

We often talk about metrics like log2fold change

``` r
#treated/control
log2(10/10)
```

    [1] 0

``` r
log2(10/20)
```

    [1] -1

``` r
log2(1/4)
```

    [1] -2

Let’s calculate the log2 fold change for our treated over control mean
counts

``` r
meancounts$log2fc <- log2(meancounts$treated.mean/meancounts$control.mean)
```

``` r
head(meancounts)
```

                    control.mean treated.mean      log2fc
    ENSG00000000003       900.75       658.00 -0.45303916
    ENSG00000000005         0.00         0.00         NaN
    ENSG00000000419       520.50       546.00  0.06900279
    ENSG00000000457       339.75       316.50 -0.10226805
    ENSG00000000460        97.25        78.75 -0.30441833
    ENSG00000000938         0.75         0.00        -Inf

A common rule of thumb is log2 fold change cutoff +2 or -2 to call genes
either up regulated or down regulated

``` r
# the number of upregulated genes 
sum(meancounts$log2fc > +2, na.rm =T)
```

    [1] 1846

``` r
# number of downregulated genes 
sum(meancounts$log2fc > -2, na.rm =T)
```

    [1] 22928

> Q8. Using the up.ind vector above can you determine how many up
> regulated genes we have at the greater than 2 fc level?

There are 1846 upregulated genes

> Q9. Using the down.ind vector above can you determine how many down
> regulated genes we have at the greater than 2 fc level?

There are 22928 down regulated genes

> Q10. Do you trust these results? Why or why not?

I do not trust these results as they were formed from calculating the
mean difference between the treat and control numbers. However the mean
is susceptible to outliers which can skew the overall value in one
direction. This can make mean differences much lower or higher than in
reality.

\##DeSeq2 Analysis

``` r
library(DESeq2)
```

    Warning: package 'matrixStats' was built under R version 4.4.3

For DESeq analysis we need three things -count values(`contData`)  
- metadata telling us about the columns in `contData`(`colData`) -design
of the expiriment (what do you want to compare)

Our first function from DESeq2 will set up the input required for
analysis by storing all these 3 things together

``` r
dds <-DESeqDataSetFromMatrix(countData = counts,colData = metadata,design = ~dex)
```

    converting counts to integer mode

    Warning in DESeqDataSet(se, design = design, ignoreRank): some variables in
    design formula are characters, converting to factors

The main function in DeSeq2 that runs the analysis is called DESeq()

``` r
dds <-DESeq(dds)
```

    estimating size factors

    estimating dispersions

    gene-wise dispersion estimates

    mean-dispersion relationship

    final dispersion estimates

    fitting model and testing

``` r
res <-results(dds)
head(res)
```

    log2 fold change (MLE): dex treated vs control 
    Wald test p-value: dex treated vs control 
    DataFrame with 6 rows and 6 columns
                      baseMean log2FoldChange     lfcSE      stat    pvalue
                     <numeric>      <numeric> <numeric> <numeric> <numeric>
    ENSG00000000003 747.194195     -0.3507030  0.168246 -2.084470 0.0371175
    ENSG00000000005   0.000000             NA        NA        NA        NA
    ENSG00000000419 520.134160      0.2061078  0.101059  2.039475 0.0414026
    ENSG00000000457 322.664844      0.0245269  0.145145  0.168982 0.8658106
    ENSG00000000460  87.682625     -0.1471420  0.257007 -0.572521 0.5669691
    ENSG00000000938   0.319167     -1.7322890  3.493601 -0.495846 0.6200029
                         padj
                    <numeric>
    ENSG00000000003  0.163035
    ENSG00000000005        NA
    ENSG00000000419  0.176032
    ENSG00000000457  0.961694
    ENSG00000000460  0.815849
    ENSG00000000938        NA

## Volcano plot

This a common summary figure from these types of expiriments and plot
the log2fold change vs the adjusted p value

``` r
plot(res$log2FoldChange, -log(res$padj))
abline(v=c(-2,2),col="red")
abline(h=-log(0.04), col="red")
```

![](class12lab_files/figure-commonmark/unnamed-chunk-20-1.png)

## Save our results

``` r
write.csv(res,file="my_results_csv")
```
