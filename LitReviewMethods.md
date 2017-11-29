---
title: "Working Notes"
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
Lit reviews should be as repeatable as any other methodology. 


## R Environment

```r
library(tidyverse)
```

```
## -- Attaching packages -------------------------------------------------------------------------------- tidyverse 1.2.1 --
```

```
## v ggplot2 2.2.1     v purrr   0.2.4
## v tibble  1.3.4     v dplyr   0.7.4
## v tidyr   0.7.2     v stringr 1.2.0
## v readr   1.1.1     v forcats 0.2.0
```

```
## -- Conflicts ----------------------------------------------------------------------------------- tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(stringr)
library(knitr)
library(kableExtra)
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

# Search Protocol
## Web of Science Search
In this example, searching for "lecture recording" OR "lecture capture" in Web of Knowledge (29/11/2017). Save to other formats - tab delimited. 

(Had to save the tab delimited as a csv to read it in properly!)

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




## Exploring the Data
### Publications by Time

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

### Publications by Source

```r
BySource <- ggplot (data = LRecs, aes(x = ISO.Source.Abbrev, fill=..count..)) + 
  geom_bar() + 
  labs (title = "Lecture Recoding Publications by Source", x = "Source Name") + 
  theme_bw() + scale_fill_gradientn(colors = wes_palette("Darjeeling"))  + 
  theme(axis.text.x = element_text(angle = 90), panel.grid = element_blank())
BySource
```

![](LitReviewMethods_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

Need to reorder this somehow


### Wordles
This is not working yet!


```r
Abstract.corpus <- Corpus(VectorSource(LRecs$Abstract)) %>%
  tm_map(removePunctuation) %>%
  tm_map(removeNumbers) %>%
  tm_map(tolower)  %>%
  tm_map(removeWords, stopwords("english")) %>%
  tm_map(stripWhitespace) %>%
  tm_map(PlainTextDocument)
```

Then...

```r
Abstract.dtm <- DocumentTermMatrix(Abstract.corpus)
```
Getting the following error:
`Error in simple_triplet_matrix(i, j, v, nrow = length(terms), ncol = length(corpus),  : 
  'i, j' invalid`
  
Something about the text having too much N/A stuff? Achhhh I can't do this any longer tonight. 
  



