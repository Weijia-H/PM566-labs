Lab 06 - Text Mining
================

# Learning goals

  - Use `unnest_tokens()` and `unnest_ngrams()` to extract tokens and
    ngrams from text.
  - Use dplyr and ggplot2 to analyze text data

# Lab description

For this lab we will be working with a new dataset. The dataset contains
transcription samples from <https://www.mtsamples.com/>. And is loaded
and “fairly” cleaned at
<https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv>.

This markdown document should be rendered using `github_document`
document.

# Setup the Git project and the GitHub repository

1.  Go to your documents (or wherever you are planning to store the
    data) in your computer, and create a folder for this project, for
    example, “PM566-labs”

2.  In that folder, save [this
    template](https://raw.githubusercontent.com/USCbiostats/PM566/master/content/assignment/06-lab.Rmd)
    as “README.Rmd”. This will be the markdown file where all the magic
    will happen.

3.  Go to your GitHub account and create a new repository, hopefully of
    the same name that this folder has, i.e., “PM566-labs”.

4.  Initialize the Git project, add the “README.Rmd” file, and make your
    first commit.

5.  Add the repo you just created on GitHub.com to the list of remotes,
    and push your commit to origin while setting the upstream.

### Setup packages

You should load in `dplyr`, (or `data.table` if you want to work that
way), `ggplot2` and `tidytext`. If you don’t already have `tidytext`
then you can install with

``` r
install.packages("tidytext")
```

### read in Medical Transcriptions

Loading in reference transcription samples from
<https://www.mtsamples.com/>

``` r
library(readr)
library(dplyr)
library(tidytext)
library(ggplot2)
mt_samples <- read_csv("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv")
mt_samples <- mt_samples %>%
  select(description, medical_specialty, transcription)

head(mt_samples)
```

    ## # A tibble: 6 x 3
    ##   description                  medical_specialty   transcription                
    ##   <chr>                        <chr>               <chr>                        
    ## 1 A 23-year-old white female ~ Allergy / Immunolo~ "SUBJECTIVE:,  This 23-year-~
    ## 2 Consult for laparoscopic ga~ Bariatrics          "PAST MEDICAL HISTORY:, He h~
    ## 3 Consult for laparoscopic ga~ Bariatrics          "HISTORY OF PRESENT ILLNESS:~
    ## 4 2-D M-Mode. Doppler.         Cardiovascular / P~ "2-D M-MODE: , ,1.  Left atr~
    ## 5 2-D Echocardiogram           Cardiovascular / P~ "1.  The left ventricular ca~
    ## 6 Morbid obesity.  Laparoscop~ Bariatrics          "PREOPERATIVE DIAGNOSIS: , M~

-----

## Question 1: What specialties do we have?

We can use `count()` from `dplyr` to figure out how many different
catagories do we have? Are these catagories related? overlapping? evenly
distributed?

``` r
mt_samples %>%
  count(medical_specialty, sort = TRUE)
```

    ## # A tibble: 40 x 2
    ##    medical_specialty                 n
    ##    <chr>                         <int>
    ##  1 Surgery                        1103
    ##  2 Consult - History and Phy.      516
    ##  3 Cardiovascular / Pulmonary      372
    ##  4 Orthopedic                      355
    ##  5 Radiology                       273
    ##  6 General Medicine                259
    ##  7 Gastroenterology                230
    ##  8 Neurology                       223
    ##  9 SOAP / Chart / Progress Notes   166
    ## 10 Obstetrics / Gynecology         160
    ## # ... with 30 more rows

-----

## Question 2

  - Tokenize the the words in the `transcription` column
  - Count the number of times each token appears
  - Visualize the top 20 most frequent words

Explain what we see from this result. Does it makes sense? What insights
(if any) do we get?

``` r
library(forcats)
mt_samples %>%
  unnest_tokens(output =  token, input = transcription) %>%
  count(token, sort=T)%>%
  top_n(n=20, wt=n)%>%
  ggplot(aes(x=n, y=fct_reorder(token,n)))+
  geom_col()
```

![](README_06_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

-----

## Question 3

  - Redo visualization but remove stopwords before
  - Bonus points if you remove numbers as well

What do we see know that we have removed stop words? Does it give us a
better idea of what the text is about?

``` r
library(forcats)
mt_samples %>%
  unnest_tokens(word, transcription)%>%
  anti_join(tidytext::stop_words)%>%
  filter(!(word %in% as.character(seq(0,100))))%>%
  count(word, sort=T)%>%
  top_n(20,n)%>%
  ggplot(aes(x=n, y=fct_reorder(word, n)))+
  geom_col()
```

    ## Joining, by = "word"

![](README_06_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
mt_samples %>%
  unnest_tokens(word, transcription)%>%
  filter(word %in% tidytext::stop_words$word)%>%
  count(word)%>%
  top_n(20,n)%>%
  ggplot(aes(x=n, y=fct_reorder(word, n)))+
  geom_col()
```

![](README_06_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

-----

# Question 4

repeat question 2, but this time tokenize into bi-grams. how does the
result change if you look at tri-grams?

``` r
mt_samples %>%
  unnest_ngrams(output =  token, input = transcription ,n=2) %>%
  count(token, sort=T)%>%
  top_n(n=20, wt=n)%>%
  ggplot(aes(x=n, y=fct_reorder(token,n)))+
  geom_col()
```

![](README_06_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

-----

# Question 5

Using the results you got from questions 4. Pick a word and count the
words that appears after and before it.

``` r
library(tidyr)
mt_bigrams <- mt_samples %>%
  unnest_ngrams(output=token, input=transcription,n=2)%>%
  separate (col = token, into = c("word1", "word2"),sep = " ")%>%
  select(word1, word2)

mt_bigrams%>%
  filter(word1 == "blood")%>%
  count(word2, sort =T)
```

    ## # A tibble: 161 x 2
    ##    word2        n
    ##    <chr>    <int>
    ##  1 pressure  1265
    ##  2 loss       965
    ##  3 cell       130
    ##  4 in         114
    ##  5 cells      112
    ##  6 sugar       91
    ##  7 and         84
    ##  8 sugars      79
    ##  9 was         65
    ## 10 cultures    53
    ## # ... with 151 more rows

``` r
mt_bigrams%>%
  filter(word2 == "blood")%>%
  count(word1, sort =T)
```

    ## # A tibble: 439 x 2
    ##    word1         n
    ##    <chr>     <int>
    ##  1 estimated   754
    ##  2 white       180
    ##  3 signs       170
    ##  4 and         154
    ##  5 of          149
    ##  6 red         123
    ##  7 her         116
    ##  8 his          99
    ##  9 the          96
    ## 10 no           72
    ## # ... with 429 more rows

``` r
mt_bigrams%>%
  count(word1, word2, sort=T)
```

    ## # A tibble: 301,399 x 3
    ##    word1   word2       n
    ##    <chr>   <chr>   <int>
    ##  1 the     patient 20301
    ##  2 of      the     19050
    ##  3 in      the     12784
    ##  4 to      the     12372
    ##  5 was     then     6952
    ##  6 and     the      6346
    ##  7 patient was      6291
    ##  8 the     right    5509
    ##  9 on      the      5241
    ## 10 the     left     4858
    ## # ... with 301,389 more rows

``` r
mt_bigrams%>%
  anti_join(
    tidytext::stop_words%>% select(word), by =c("word1" = "word")
  ) %>%
  anti_join(
    tidytext::stop_words%>% select(word), by =c("word2" = "word")
  ) %>%
  count(word1, word2, sort=T)
```

    ## # A tibble: 128,013 x 3
    ##    word1         word2           n
    ##    <chr>         <chr>       <int>
    ##  1 0             vicryl       1802
    ##  2 blood         pressure     1265
    ##  3 medical       history      1223
    ##  4 diagnoses     1            1192
    ##  5 preoperative  diagnosis    1176
    ##  6 physical      examination  1156
    ##  7 4             0            1123
    ##  8 vital         signs        1117
    ##  9 past          medical      1113
    ## 10 postoperative diagnosis    1092
    ## # ... with 128,003 more rows

-----

# Question 6

Which words are most used in each of the specialties. you can use
`group_by()` and `top_n()` from `dplyr` to have the calculations be done
within each specialty. Remember to remove stopwords. How about the most
5 used words?

``` r
mt_samples %>%
  unnest_tokens(token, transcription)%>%
  anti_join(tidytext::stop_words, by =c("token"="word"))%>%
  group_by(medical_specialty)%>%
  count(token)%>%
  top_n(5,n)
```

    ## # A tibble: 209 x 3
    ## # Groups:   medical_specialty [40]
    ##    medical_specialty    token         n
    ##    <chr>                <chr>     <int>
    ##  1 Allergy / Immunology allergies    21
    ##  2 Allergy / Immunology history      38
    ##  3 Allergy / Immunology nasal        13
    ##  4 Allergy / Immunology noted        23
    ##  5 Allergy / Immunology past         13
    ##  6 Allergy / Immunology patient      22
    ##  7 Autopsy              1            66
    ##  8 Autopsy              anterior     47
    ##  9 Autopsy              inch         59
    ## 10 Autopsy              left         83
    ## # ... with 199 more rows

# Question 7 - extra

Find your own insight in the data:

Ideas:

  - Interesting ngrams
  - See if certain words are used more in some specialties then others
