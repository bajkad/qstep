Preprocessing text and analyzing word frequencies
================
Kasper Welbers & Wouter van Atteveldt
2020-03

-   [Introduction](#introduction)
-   [Preprocessing](#preprocessing)
    -   [Introduction with toy data](#introduction-with-toy-data)
    -   [Using real data](#using-real-data)
    -   [Filtering the DTM](#filtering-the-dtm)
-   [Analysis](#analysis)
    -   [Word frequencies and wordclouds](#word-frequencies-and-wordclouds)
    -   [Compare corpora](#compare-corpora)
    -   [Keyword-in-context](#keyword-in-context)
-   [Advanced preprocessing](#advanced-preprocessing)
    -   [Introduction with toy data](#introduction-with-toy-data-1)
    -   [Using real data](#using-real-data-1)

Introduction
============

In this tutorial you will learn how to preprocess texts in R, and to perform some basic text analysis techniques.

We will be using the [quanteda package](https://quanteda.io/). This is an extensive text analysis suite for R. It covers everything you need to perform a variety of automatic text analysis techniques, and features clear and extensive documentation.

``` r
library(quanteda)
```

For this tutorial we will mainly be relying on basic preprocessing. At the end of the tutorial (if you still have time) we include a small section on using advanced preprocessing.

Preprocessing
=============

Introduction with toy data
--------------------------

Many text analysis techniques only use the frequencies of words in documents. This is also called the bag-of-words assumption, because texts are then treated as bags of individual words. Despite ignoring much relevant information in the order of words and syntax, this approach has proven to be very powerfull and efficient.

The standard format for representing a bag-of-words is as a `document-term matrix` (DTM). This is a matrix in which rows are documents, columns are terms, and cells indicate how often each term occured in each document. We'll first create a small example DTM from a few lines of text. Here we use quanteda's `dfm()` function, which stands for `document-feature matrix` (DFM). This is essentially a document term matrix (DTM), but quanteda prefers 'features' over 'terms', because it's more general (sometimes the columns are higher order n-grams or wordembeddings, which are not technically terms)

``` r
text <-  c(d1 = "Cats are awesome!",
           d2 = "We need more cats!",
           d3 = "This is a soliloquy about a cat.")

dtm <- dfm(text, tolower=F)
dtm
```

Here you see, for instance, that the word `soliloquy` only occurs in the third document. In this matrix format, we can perform calculations with texts, like analyzing different sentiments of frames regarding cats, or the computing the similarity betwee the third sentence and the first two sentences.

However, directly converting a text to a DTM is a bit crude. Note, for instance, that the words `Cats`, `cats`, and `cat` are given different columns. In this DTM, "Cats" and "awesome" are as different as "Cats" and "cats", but for many types of analysis we would be more interested in the fact that both texts are about felines, and not about the specific word that is used. Also, for performance it can be useful (or even necessary) to use fewer columns, and to ignore less interesting words such as `is` or very rare words such as `soliloquy`.

This can be achieved by using additional `preprocessing` steps. In the next example, we'll again create the DTM, but this time we make all text lowercase, ignore stopwords and punctuation, and perform `stemming`. Simply put, stemming removes some parts at the ends of words to ignore different forms of the same word, such as singular versus plural ("gun" or "gun-s") and different verb forms ("walk","walk-ing","walk-s")

``` r
dtm = dfm(text, tolower=T, remove = stopwords('en'), stem = T, remove_punct=T)
dtm
```

The `tolower` argument determines whether texts are (TRUE) or aren't (FALSE) converted to lowercase. `stem` determines whether stemming is (TRUE) or isn't (FALSE) used. The remove argument is a bit more tricky. If you look at the documentation for the dfm function (`?dfm`) you'll see that `remove` can be used to give "a pattern of user-supplied features to ignore". In this case, we actually used another function, `stopwords()`, to get a list of english stopwords. You can see for yourself.

``` r
stopwords('en')
```

This list of words is thus passed to the `remove` argument in the `dfm()` to ignore these words. If you are using texts in another language, make sure to specify the language, such as stopwords('nl') for Dutch or stopwords('de') for German.

And that's basically all you need to know to prepare a DTM (or DFM, if you prefer that terminology) in R. As you'll see later on today, many text analysis functions in R use this DTM as the data input format.

Using real data
---------------

Now let's see how this works with more and more realistic data. For this tutorial we will be importing text from a csv file. For convenience, we're using a csv that's available online, but the process is the same for a csv file on your own computer. The data consists of the State of the Union speeches of US presidents, with each document (i.e. row in the csv) being a paragraph. The data will be imported as a data.frame.

``` r
library(readr)
url = 'https://bit.ly/2QoqUQS'
d = read_csv(url)
```

We now have a data.frame in which each row contains a text (here stored in the text column). The other column can be considered as meta data of the text (which president, which party, etc). This is a convenient format that can readily be used by quanteda and corpustools to create a corpus of tokenized texts and meta data.

``` r
d
```

We could directly pass the text to the `dfm` function, just like above. However, here we take one additional step: we first make a quanteda `corpus`, and then pass this corpus to the `dfm` function. The advantage of this middle step is that all the other columns in `d` will then be stored as docvars (document variables, or meta data) in the DFM. Another advantage is that the corpus provides us with some additional tools for analyzing the texts. As shown below, we can query the data for multi-word phrases (this information is lost in the DTM) and look for keywords in their actual context.

We can create a quanteda corpus with the `corpus()` function. If you want to learn more about this function, recall that you can use the question mark to look at the documentation.

``` r
?corpus
```

Here you see that for a data.frame, we need to specify which column contains the text field. Also, the text column must be a character vector.

``` r
corp = corpus(d, text_field = 'text')  ## create the corpus
corp
```

We can now pass this corpus to the `dfm()` function and set the preprocessing parameters.

``` r
dtm = dfm(corp, tolower=T, stem=T, remove=stopwords('en'), remove_punct=T)
dtm
```

This dtm has 23,469 documents and 20,429 features (i.e. terms). In addition, there are 4 `docvars`. You can look at these docvars with the `docvars()` function. (here we use `head` to only show the top rows)

``` r
head(docvars(dtm))
```

Filtering the DTM
-----------------

Depending on the type of analysis that you want to conduct, we might not need this many words, and with larger datasets and certain types of analysis we might actually run into computational limitations.

Luckily, many of these 20K features are not that informative. The distribution of term frequencies tends to have a very long tail, with many words occuring only once or a few times in our corpus. For many types of bag-of-words analysis it would not harm to remove these words, and it might actually improve results.

We can use the `dfm_trim` function to remove columns based on criteria specified in the arguments. Here we say that we want to remove all terms for which the frequency (i.e. the sum value of the column in the DTM) is below 10.

``` r
dtm  = dfm_trim(dtm, min_termfreq = 10)
dtm
```

Now we have about 5000 features left. See `?dfm_trim` for more options.

Analysis
========

Here we perform some simple word frequency based analyses.

Word frequencies and wordclouds
-------------------------------

Get most frequent words in corpus.

``` r
textplot_wordcloud(dtm, max_words = 50)     ## top 50 (most frequent) words
textplot_wordcloud(dtm, max_words = 50, color = c('blue','red')) ## change colors
textstat_frequency(dtm, n = 10)             ## view the frequencies 
```

You can also inspect a subcorpus. For example, looking only at Obama speeches. To subset the DTM we can use quanteda's `dtm_subset()`. This lets us directly use the document variables to take a subset of the corpus

``` r
obama_dtm = dfm_subset(dtm, President == "Barack Obama")
textplot_wordcloud(obama_dtm, max_words = 25)
```

Compare corpora
---------------

Instead of looking just at frequencies, we can also compare frequencies between two subcorpora using the `textstat_keyness` function. If you look at the documentation, you see that this function requires a `target` argument, which is a selection of documents that you want to compare (to the rest of the documents).

Here we compare Obama's word use to that of the other presidents. Our `target` should therefore be the selection of all docvars where the president is Obama. WE first make a logical vector with this selection, and then use this as the target in `textstat_keyness`.

``` r
is_obama = docvars(dtm)$President == 'Barack Obama' 
ts = textstat_keyness(dtm, target = is_obama)
head(ts, 20)    ## view first 20 results
```

The results show how often each feature/word occured in the target data (speeches from Obama) and the other (reference) data. We also get a chi2 score and p-value. A higher chi2 value means that Obama is relatively more likely to use this word compared to other presidents, and the p value indicates whether the difference in usage is significant.

We can visualize these results, stored under the name `ts`, by using the textplot\_keyness function

``` r
textplot_keyness(ts)
```

Here the words on the right hand side (chi2 &gt; 0) are the words that Obama used relatively more often, and the words on the left hand side are the words that Obama used relatively less often.

Keyword-in-context
------------------

As seen in the first tutorial, a keyword-in-context listing shows a given keyword in the context of its use. This is a good help for interpreting words from a wordcloud or keyness plot.

Since a DTM only knows word frequencies, the `kwic()` function requires the corpus object as input.

``` r
k = kwic(corp, 'freedom', window = 7)
head(k, 10)    ## only view first 10 results
```

The `kwic()` function can also be used to focus an analysis on a specific search term. You can use the output of the kwic function to create a new DTM, in which only the words within the shown window are included in the DTM. With the following code, a DTM is created that only contains words that occur within 10 words from `terror*` (terrorism, terrorist, terror, etc.).

``` r
terror = kwic(corp, 'terror*')
terror_corp = corpus(terror)
terror_dtm = dfm(terror_corp, tolower=T, remove=stopwords('en'), stem=T, remove_punct=T)
```

Now you can focus an analysis on whether and how Presidents talk about `terror*`.

``` r
textplot_wordcloud(terror_dtm, max_words = 50)     ## top 50 (most frequent) words
```

Advanced preprocessing
======================

Introduction with toy data
--------------------------

For many types of analysis the preprocessing support in Quanteda will be sufficient. However, for some types of analysis, and for some languages, it will be worthwhile to use a more advanced Natural Language Processing pipeline.

To perform advanced preprocessing in R, we need other packages. For this tutorial we will use the [udpipe package](https://cran.r-project.org/web/packages/udpipe/index.html). Or actually, we will be using the [corpustools package](https://cran.r-project.org/web/packages/corpustools/index.html) which has a nice wrapper for udpipe, and makes it easy to create a DFM from the output.

``` r
library(corpustools)
```

Let's again start with a simple example.

``` r
text <-  c(d1 = "Cats are awesome!",
           d2 = "We need more cats!",
           d3 = "This is a soliloquy about a cat.")
```

In `corpustools`, we can use the `create_tcorpus` function to create a corpus. By default this only tokenizes the text, but we can add the `udpipe_model` argument to specify a model that we want to use (each language requires a different model, and some languages have multiple models). Here we use the 'english-ewt' model. The first time you run this code, R will automatically download the model (by default to your current working directory).

``` r
tc = create_tcorpus(text, udpipe_model = 'english-ewt')
```

The object `tc` (short for tcorpus) is a special type of corpus in which the texts are already preprocessed. The preprocessed texts are stored in a format that is common for the output of a natural language parser: a data.frame in which each row is a token, and columns contain information about the token. Its best to just show it.

``` r
head(tc$tokens)
```

For the first word "Cats" we see that the lemma is "cat", that this is a noun (POS = part-of-speech), and furthermore that this is plural (Number=Plur). For the second word "are", we see that the lemma is "be", so here you clearly see the difference between stemming and lemmatization.

The `feats` column is a bit inconvenient, as it contains nested data. The tcorpus therefor has a method for expanding this column to multiple columns. Here we use this to extract the Number (singular or plural) and Person (first, second or third) features.

``` r
tc$feats_to_columns(keep = c('Number','Person'))
tc$tokens
```

There are many things you can do with this enriched textual data, but for now we'll focus on two usefull features:

-   We can use the lemma instead of the stemmed and lowercased tokens. This is more accurate, especially for highly inflected languages such as Dutch and German.
-   We can filter the tokens based on part-of-speech tags (POS). For instance, to focus an analysis on the nouns, verbs and/or names used.

`corpustools` has the `get_dfm` function, which makes it easy to cast these tokens into a quanteda DFM. This also has the subset\_tokens argument, in which you can specify a condition for selecting which tokens to include. Here we make a DFM with 'lemma' as features, and where we only include nouns, proper names and verbs.

``` r
dfm2 = get_dfm(tc, feature = 'lemma', subset_tokens = POS %in% c('NOUN','PROPN','VERB'))
dfm2
```

Using real data
---------------

Now let's do this with real data. Beware, however, that using these NLP parsers is quite heavy lifting, and can take quite a while. For this example, we'll only use Obamas speeches.

``` r
obama = d[d$President == "Barack Obama",]
```

This time we're passing a data.frame to `create_tcorpus`. We therefore need to specify which column contains the `text`. (If there is a unique ID in the data, you can use the doc\_column argument to use it. Otherwise, corpustools with simply use the row indices as IDs).

``` r
tc_obama = create_tcorpus(obama, text_columns = 'text', udpipe_model='english-ewt')
```

As an example, we can now make a DFM with only the NOUNS spoken by Obama, and create a wordcloud.

``` r
dfm_obama = get_dfm(tc_obama, feature = 'lemma', subset_tokens = POS %in% c('NOUN'))
textplot_wordcloud(dfm_obama, max_words = 25)
```
