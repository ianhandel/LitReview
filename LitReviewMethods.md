---
title: "WIP: Quantitative Lit Review Methodology"
author: "Jilly MacKay"
date: "29 November 2017"
output: 
  html_document:
    keep_md: true
    theme: spacelab
    highlight: haddock
    toc: true 
    toc_float: true
    toc_depth: 3
    
---
# About this Methodology
Trying to establish a repeatable (and therefore less time-dependent) methodology for quantitative summaries of literature review searches.

**Please note I am not implying that this is all a literature search is - I just want a repeatable methodology for the data processing part!**

## R Environment

```r
# Remember any packages you don't have can be installed with "install.packages("package.name")"
library(tidyverse)
```

```
## -- Attaching packages --------------------------------------------------------------------------------- tidyverse 1.2.1 --
```

```
## v ggplot2 2.2.1     v purrr   0.2.4
## v tibble  1.3.4     v dplyr   0.7.4
## v tidyr   0.7.2     v stringr 1.2.0
## v readr   1.1.1     v forcats 0.2.0
```

```
## -- Conflicts ------------------------------------------------------------------------------------ tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(stringr)
library(knitr)
library(tibble)
library(wesanderson)
library(tm)
```

```
## Loading required package: NLP
```

```
## 
## Attaching package: 'NLP'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     annotate
```

```r
library(wordcloud)
```

```
## Loading required package: RColorBrewer
```

```r
library(textstem)
```

# Search Protocol
## Web of Science Search
Go to [Web of Science](https://webofknowledge.com/) and perform your search. In this case, we're searching for "lecture recording" OR "lecture capture" across all years. 

Once the search results have loaded go to the option at the top of the search results which, by default, reads `Save to EndNote online` and scroll down to `Save to other File Formats`. 

After clicking this, a pop up will ask you how many records you want to save, add the first and last record number in the `Records` box, and change `Record Content` to `Full Record`.

The `File Format` you want is `Tab-delimited (Windows UTF-8)`
Save this file. 

### Current Unsolved Problem
At some point I want to be able to read this file into R directly and manage it from there, but R is having real trouble reading the format of the file, even when it is UTF-8. 

Therefore, I have to open this file in Excel and save it as a .csv file. The file we're working with is therefore `lrec.csv`



# Data Management Protocol
## Reading in the Data

```r
LRecs <- read.csv("lrec.csv")
LRecs <- 
  LRecs %>%
  rename (Publication.Type = PT, 
          Authors = AU, 
          Book.Authors = BA, 
          Book.Editors = BE, 
          Book.Grp.Authors = GP, 
          Author.Full = AF, 
          Book.Author.Full = BF, 
          Group.Authors = CA, 
          Title = TI, 
          Publication.Name = SO, 
          Book.Series.Title = SE, 
          Book.Series.Subtitle = BS, 
          Language = LA, 
          Doc.Type = DT, 
          Conference.Title = CT, 
          Conference.Date = CY, 
          Conf.Location = CL, 
          Conf.Sponsors = SP, 
          Conf.Host = HO, 
          Keywords.Author = DE, 
          Keywords.Plus = ID, 
          Abstract = AB, 
          Author.Address = C1, 
          Reprint.Address = RP, 
          Contact.Email = EM, 
          ResearchID = RI, 
          OrcID = OI, 
          Funding.Agency = FU, 
          Funding.Text = FX, 
          Cited.Refs = CR, 
          Cited.Refs.Count = NR, 
          Times.Cited.Core = TC, 
          Times.Cited = Z9, 
          Usage.180Days = U1, 
          Usage.Since2013 = U2, 
          Publisher = PU, 
          Publisher.City = PI, 
          Publisher.Address = PA, 
          ISSN = SN, 
          eiSSN = EI, 
          ISBN = BN, 
          Source.Abbrev = J9, 
          ISO.Source.Abbrev = JI, 
          Date.Published = PD, 
          Year.Published = PY, 
          Volume = VL, 
          Issue = IS, 
          Part.Number = PN, 
          Supplement = SU, 
          Special.Issue = SI, 
          Meeting.Abstract = MA, 
          Pg.Start = BP, 
          Pg.End = EP,
          Article.Number = AR, 
          DOI = DI, 
          BkDOI = D2, 
          Page.Count = PG, 
          WoS.Cats = WC, 
          Res.Areas = SC, 
          Doc.Delivery.Number = GA, 
          Accession.Number = UT, 
          PubMedID = PM, 
          Open.Access.Journal = OA, 
          Highly.Cited = HC, 
          Hot.Paper = HP, 
          Date.Exported = DA)
```




# Exploring the Data
## Publications by Time

```r
ByYear <- ggplot (data = LRecs, aes(x = Year.Published, fill=..count..)) + 
  geom_histogram(binwidth = 1) + 
  labs (title = "Lecture Recoding Publications by Year", x = "Publication Year") + 
  theme_bw() + scale_fill_gradientn(colors = wes_palette("Darjeeling"))  + 
  theme(axis.text.x = element_text(angle = 90), panel.grid = element_blank()) + 
  scale_x_continuous(breaks = seq(1999,2017,1))
ByYear
```

![](LitReviewMethods_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

## Publications by Source

```r
BySource <- ggplot (data = LRecs, aes(x = ISO.Source.Abbrev, fill=..count..)) + 
  geom_bar() + 
  labs (title = "Lecture Recoding Publications by Source", x = "Source Name") + 
  theme_bw() + scale_fill_gradientn(colors = wes_palette("Darjeeling"))  + 
  theme(axis.text.x = element_text(angle = 90), panel.grid = element_blank())
BySource
```

![](LitReviewMethods_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

### Current Unsolved Problem
Need a neat way to reorder this


## Visualise Abstracts
Visualise roughly what is being said in the abstracts of these papers.

```r
LRecs$LemAbstracts <- lemmatize_strings(LRecs$Abstract)
Abstract.corpus <- Corpus(VectorSource(LRecs$LemAbstracts)) %>%
     tm_map(removePunctuation) %>%
     tm_map(removeNumbers) %>%
     tm_map(tolower)  %>%
     tm_map(removeWords, stopwords("english")) %>%
     tm_map(removeWords, c("lecture", "record", "capture")) %>%
     tm_map(stripWhitespace)

Abstract.Wordle <- wordcloud(Abstract.corpus, scale = c(5,0.5), max.words = 100, random.order = FALSE, random.color = FALSE, rot.per = 0, use.r.layout = FALSE, colors = wes_palette("Darjeeling"))
```

![](LitReviewMethods_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

