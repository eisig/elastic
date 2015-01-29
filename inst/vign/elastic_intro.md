<!--
%\VignetteEngine{knitr::knitr}
%\VignetteIndexEntry{elastic introduction}
-->



elastic introduction
======

`elastic` is an R client for [Elasticsearch](http://www.elasticsearch.org/). This vignette is an introduction to the package, while other vignettes dive into the details of various topics.

## Installation

You can install from CRAN (once the package is up there)


```r
install.packages("elastic")
```

Or the development version from GitHub


```r
install.packages("devtools")
devtools::install_github("ropensci/elastic")
```

Then load the package


```r
library("elastic")
```

## Elasticsearch info

+ [Elasticsearch home page](http://elasticsearch.org)
+ [API docs](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/index.html)

## Install Elasticsearch

* [Elasticsearch installation help](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/_installation.html)

__Unix (linux/osx)__

Replace `1.4.1` with the version you are working with.

+ Download zip or tar file from Elasticsearch [see here for download](http://www.elasticsearch.org/overview/elkdownloads/), e.g., `curl -L -O https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.1.tar.gz`
+ Uncompress it: `tar -xvf elasticsearch-1.4.1.tar.gz`
+ Move it: `sudo mv /path/to/elasticsearch-1.4.1 /usr/local`
+ Navigate to /usr/local: `cd /usr/local`
+ Add shortcut: `sudo ln -s elasticsearch-1.4.1 elasticsearch`

On OSX, you can install via Homebrew: `brew install elasticsearch`

__Windows__

Windows users can follow the above, but unzip the zip file instead of uncompressing the tar file.

## Start Elasticsearch

* Navigate to elasticsearch: `cd /usr/local/elasticsearch`
* Start elasticsearch: `bin/elasticsearch`

I create a little bash shortcut called `es` that does both of the above commands in one step (`cd /usr/local/elasticsearch && bin/elasticsearch`).

__Note:__ Windows users should run the `elasticsearch.bat` file

## Initialize connection

The function `connect()` is used before doing anything else to set the connection details to your remote or local elasticsearch store. The details created by `connect()` are written to your options for the current session, and are used by `elastic` functions.


```r
connect()
```

```
#> uri:       http://127.0.0.1 
#> port:      9200 
#> username:  NULL 
#> password:  NULL 
#> api key:   NULL 
#> elasticsearch details:   
#>       status:                  200 
#>       name:                    Taj Nital 
#>       Elasticsearch version:   1.4.2 
#>       ES version timestamp:    2014-12-16T14:11:12Z 
#>       lucene version:          4.10.2
```

On package load, your base url and port are set to `http://127.0.0.1` and `9200`, respectively. You can of course override these settings per session or for all sessions. 

## Get some data

Elasticsearch has a bulk load API to load data in fast. The format is pretty weird though. It's sort of JSON, but would pass no JSON linter. I include a few data sets in `elastic` so it's easy to get up and running, and so when you run examples in this package they'll actually run the same way (hopefully).

I have prepared a non-exported function useful for preparing the weird format that Elasticsearch wants for bulk data loads (see below). See `elastic:::make_bulk_plos` and `elastic:::make_bulk_gbif`.

### Shakespeare data

Elasticsearch provides some data on Shakespeare plays. I've provided a subset of this data in this package. Get the path for the file specific to your machine:


```r
shakespeare <- system.file("examples", "shakespeare_data.json", package = "elastic")
```

Then load the data into Elasticsearch:


```r
docs_bulk(shakespeare)
```

If you need some big data to play with, the shakespeare dataset is a good one to start with. You can get the whole thing and pop it into Elasticsearch (beware, may take up to 10 minutes or so.):

```sh
curl -XGET http://www.elasticsearch.org/guide/en/kibana/current/snippets/shakespeare.json > shakespeare.json
curl -XPUT localhost:9200/_bulk --data-binary @shakespeare.json
```

### Public Library of Science (PLOS) data

A dataset inluded in the `elastic` package is metadata for PLOS scholarly articles. Get the file path, then load:


```r
plosdat <- system.file("examples", "plos_data.json", package = "elastic")
docs_bulk(plosdat)
```

### Global Biodiversity Information Facility (GBIF) data

A dataset inluded in the `elastic` package is data for GBIF species occurrence records. Get the file path, then load:


```r
gbifdat <- system.file("examples", "gbif_data.json", package = "elastic")
docs_bulk(gbifdat)
```

GBIF geo data with a coordinates element to allow `geo_shape` queries


```r
gbifgeo <- system.file("examples", "gbif_geo.json", package = "elastic")
docs_bulk(gbifgeo)
```

### More data sets

There are more datasets formatted for bulk loading in the `ropensci/elastic_data` GitHub repository. Find it at [https://github.com/ropensci/elastic_data](https://github.com/ropensci/elastic_data)

## Search

Search the `plos` index and only return 1 result


```r
Search(index="plos", size=1)$hits$hits
```

```
#> [[1]]
#> [[1]]$`_index`
#> [1] "plos"
#> 
#> [[1]]$`_type`
#> [1] "article"
#> 
#> [[1]]$`_id`
#> [1] "4"
#> 
#> [[1]]$`_version`
#> [1] 1
#> 
#> [[1]]$`_score`
#> [1] 1
#> 
#> [[1]]$`_source`
#> [[1]]$`_source`$id
#> [1] "10.1371/journal.pone.0107758"
#> 
#> [[1]]$`_source`$title
#> [1] "Lactobacilli Inactivate Chlamydia trachomatis through Lactic Acid but Not H2O2"
```

Search the `plos` index, and the `article` document type, sort by title, and query for _antibody_, limit to 1 result


```r
Search(index="plos", type="article", sort="title", q="antibody", size=1)$hits$hits
```

```
#> [[1]]
#> [[1]]$`_index`
#> [1] "plos"
#> 
#> [[1]]$`_type`
#> [1] "article"
#> 
#> [[1]]$`_id`
#> [1] "568"
#> 
#> [[1]]$`_version`
#> [1] 1
#> 
#> [[1]]$`_score`
#> NULL
#> 
#> [[1]]$`_source`
#> [[1]]$`_source`$id
#> [1] "10.1371/journal.pone.0085002"
#> 
#> [[1]]$`_source`$title
#> [1] "Evaluation of 131I-Anti-Angiotensin II Type 1 Receptor Monoclonal Antibody as a Reporter for Hepatocellular Carcinoma"
#> 
#> 
#> [[1]]$sort
#> [[1]]$sort[[1]]
#> [1] "1"
```

## Get documents

Get document with `id=1`


```r
docs_get(index='plos', type='article', id=1)
```

```
#> $`_index`
#> [1] "plos"
#> 
#> $`_type`
#> [1] "article"
#> 
#> $`_id`
#> [1] "1"
#> 
#> $`_version`
#> [1] 1
#> 
#> $found
#> [1] TRUE
#> 
#> $`_source`
#> $`_source`$id
#> [1] "10.1371/journal.pone.0098602"
#> 
#> $`_source`$title
#> [1] "Population Genetic Structure of a Sandstone Specialist and a Generalist Heath Species at Two Levels of Sandstone Patchiness across the Strait of Gibraltar"
```

Get certain fields


```r
docs_get(index='plos', type='article', id=1, fields='id')
```

```
#> $`_index`
#> [1] "plos"
#> 
#> $`_type`
#> [1] "article"
#> 
#> $`_id`
#> [1] "1"
#> 
#> $`_version`
#> [1] 1
#> 
#> $found
#> [1] TRUE
#> 
#> $fields
#> $fields$id
#> $fields$id[[1]]
#> [1] "10.1371/journal.pone.0098602"
```

## Get multiple documents at once

Same index and type, different document ids


```r
docs_mget(index="plos", type="article", id=3:4)
```

```
#> $docs
#> $docs[[1]]
#> $docs[[1]]$`_index`
#> [1] "plos"
#> 
#> $docs[[1]]$`_type`
#> [1] "article"
#> 
#> $docs[[1]]$`_id`
#> [1] "3"
#> 
#> $docs[[1]]$`_version`
#> [1] 1
#> 
#> $docs[[1]]$found
#> [1] TRUE
#> 
#> $docs[[1]]$`_source`
#> $docs[[1]]$`_source`$id
#> [1] "10.1371/journal.pone.0107756"
#> 
#> $docs[[1]]$`_source`$title
#> [1] "The Effect of S-Adenosylmethionine on Cognitive Performance in Mice: An Animal Model Meta-Analysis"
#> 
#> 
#> 
#> $docs[[2]]
#> $docs[[2]]$`_index`
#> [1] "plos"
#> 
#> $docs[[2]]$`_type`
#> [1] "article"
#> 
#> $docs[[2]]$`_id`
#> [1] "4"
#> 
#> $docs[[2]]$`_version`
#> [1] 1
#> 
#> $docs[[2]]$found
#> [1] TRUE
#> 
#> $docs[[2]]$`_source`
#> $docs[[2]]$`_source`$id
#> [1] "10.1371/journal.pone.0107758"
#> 
#> $docs[[2]]$`_source`$title
#> [1] "Lactobacilli Inactivate Chlamydia trachomatis through Lactic Acid but Not H2O2"
```

Different indeces, types, and ids


```r
docs_mget(index_type_id=list(c("plos","article",1), c("gbif","record",1)))$docs[[1]]
```

```
#> $`_index`
#> [1] "plos"
#> 
#> $`_type`
#> [1] "article"
#> 
#> $`_id`
#> [1] "1"
#> 
#> $`_version`
#> [1] 1
#> 
#> $found
#> [1] TRUE
#> 
#> $`_source`
#> $`_source`$id
#> [1] "10.1371/journal.pone.0098602"
#> 
#> $`_source`$title
#> [1] "Population Genetic Structure of a Sandstone Specialist and a Generalist Heath Species at Two Levels of Sandstone Patchiness across the Strait of Gibraltar"
```

## Raw JSON data

You can optionally get back raw `json` from `Search()`, `docs_get()`, and `docs_mget()` setting parameter `raw=TRUE`.

For example:


```r
(out <- docs_mget(index="plos", type="article", id=5:6, raw=TRUE))
```

```
#> [1] "{\"docs\":[{\"_index\":\"plos\",\"_type\":\"article\",\"_id\":\"5\",\"_version\":1,\"found\":true,\"_source\":{\"id\":\"10.1371/journal.pone.0085123\",\"title\":\"MiR-21 Is under Control of STAT5 but Is Dispensable for Mammary Development and Lactation\"}},{\"_index\":\"plos\",\"_type\":\"article\",\"_id\":\"6\",\"_version\":1,\"found\":true,\"_source\":{\"id\":\"10.1371/journal.pone.0098600\",\"title\":\"Correction: Designing Mixed Species Tree Plantations for the Tropics: Balancing Ecological Attributes of Species with Landholder Preferences in the Philippines\"}}]}"
#> attr(,"class")
#> [1] "elastic_mget"
```

Then parse


```r
jsonlite::fromJSON(out)
```

```
#> $docs
#>   _index   _type _id _version found                   _source.id
#> 1   plos article   5        1  TRUE 10.1371/journal.pone.0085123
#> 2   plos article   6        1  TRUE 10.1371/journal.pone.0098600
#>                                                                                                                                                     _source.title
#> 1                                                                       MiR-21 Is under Control of STAT5 but Is Dispensable for Mammary Development and Lactation
#> 2 Correction: Designing Mixed Species Tree Plantations for the Tropics: Balancing Ecological Attributes of Species with Landholder Preferences in the Philippines
```