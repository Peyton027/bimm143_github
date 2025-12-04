# Class 6: R Functions
Peyton Ciu (PID:A18145937)

All functions in R have at least 3 things:

- A **name**, we pick this and use it to call the function.

- Input **arguments**, there can be multiple comma separated inputs to
  the function

- The **body**, lines of R code that do the work of the function

Our first function:

``` r
add <- function(x,y=1){
  x + y
}
```

Let’s test our function

``` r
add(c(1,2,3), y=10)
```

    [1] 11 12 13

``` r
add(10,100)
```

    [1] 110

## A second function

Let’s try something more interesting. Make a sequence generation tool.

The ‘sample()’ function could be useful here

``` r
n <- c("A","C","G","T")
sample(n,replace =TRUE,  5)
```

    [1] "A" "T" "A" "C" "T"

Turn this snippet into a function that returns a user specified length
dna sequence. Let’s call it ‘generate_dna’

``` r
generate_dna <-function(x, fasta =FALSE) {
  n <- c("A","C","G","T")
  # Makes single element polynucleotide sequence and stores in a vector
 v<- sample(n,replace =TRUE,  x) 
 
 # Makes gets rid of quotations 
  s<- paste(v, collapse="")
  
  cat("Well done you! \n")
  
  if(fasta == FALSE){
    return(s)
  }
  else {
    return(v)
  }
}
```

``` r
generate_dna(5)
```

    Well done you! 

    [1] "TCCGG"

``` r
s <- generate_dna(15)
```

    Well done you! 

``` r
s
```

    [1] "ATCACCAAGATTTTG"

I want the option to return a single element character vector with my
sequence all together: “GGAGTAC”

``` r
generate_dna(10 , fasta= TRUE)
```

    Well done you! 

     [1] "T" "G" "A" "C" "C" "A" "T" "T" "C" "C"

``` r
generate_dna(10 , fasta= FALSE)
```

    Well done you! 

    [1] "TAGTCTGCAT"

## A more advanced example

Make a third function that generates protein sequence of a user
specified length and format

``` r
generate_protein <- function(x, vector =FALSE){
amino_acids <- c("A", "R", "N", "D", "C", 
                 "Q", "E", "G", "H", "I", 
                 "L", "K", "M", "F", "P", 
                 "S", "T", "W", "Y", "V")
vector_form<-sample(amino_acids, x, replace=TRUE)
protein_seq <-paste(vector_form, collapse = "")
if(vector ==TRUE){
    return(vector_form)
    }else {
  return(protein_seq)}
}
```

``` r
generate_protein(20)
```

    [1] "NENIHNALWCRSWFEMLYPW"

``` r
generate_protein(20, vector =TRUE)
```

     [1] "F" "S" "P" "H" "T" "Y" "A" "R" "L" "I" "K" "E" "V" "D" "A" "T" "T" "V" "A"
    [20] "T"

> Q. Generate random protein sequences between lengths 5 and 12 amino
> acids

``` r
seq_lengths<- 6:12
for (i in seq_lengths) {
  cat(">", i, "\n", sep="")
  cat(generate_protein(i))
  cat("\n")
}
```

    >6
    EFANKH
    >7
    CWDIDEA
    >8
    ECTSPILH
    >9
    ICDHCYDEN
    >10
    ATRFQGGENN
    >11
    VEHPCTTKGML
    >12
    LPCACHSDCMPM

One approach is to do this by brute force calling our function for each
length 5 to 12

Another is to write a ‘for()’ loop to iterate through 5 to 12

A very useful 3rd approach is the ‘sapply()’ function

``` r
sapply(5:12, generate_protein)
```

    [1] "TMYGS"        "WYIWSC"       "NTNCKRG"      "VFRPLINK"     "HYCQRFNIT"   
    [6] "QAHVCAYECE"   "DWTYTQLHSGP"  "FTQGYQKVWMMY"

> **Keypoint** writing functions in R is doable but not the easiest
> thing in the world. Start with a working snippet of code and then
> using LLM tools to improve and generalize your code is a good
> approach.
