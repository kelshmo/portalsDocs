---
title: Managing Metadata
layout: article
excerpt: Merge individual, biospecimen and assay-specific metadata files.
category: tutorials
---

File annotations and metadata provide critical information about the data in the AD Knowledge portal. Annotations describe content within the files, including: type of data and assay, tissue of origin, source study, ID of the source individual and type of data. Annotations conform to a structured schema to make data highly searchable and standardized across studies and projects. The standards for defining, managing and implementing controlled vocabularies for content in Synapse is available in the synapseAnnotations Github repository. 

Some types of annotations contain private health information, and cannot be represented as file annotations. For this reason, some of the metadata is stored within a set of csv files. There are typically several metadata csv files required to fully represent details of a single file or individual. 

The individual metadata file describes each individual in the study. Each row corresponds to a unique individual identifiable by the key *individualID*. The biospecimen metadata file describes the specimens that were collected and includes metadata like tissue type, cell type or nucleic acid source. While each row corresponds to a unique specimen identified by the key *specimenID*, there may be multiple specimens per individual. The biospecimen file also contains the key *individualID*, thus the individual metadata file can be joined on this key. The assay metadata file contains experimental processing details such as technology used, batch processing, quality control metrics and the key *specimenID*. There may be more than one assay metadata file if specimens or individuals were characterized across several assay types.

In short, metadata files of type individual, biospecimen and assay can be joined together on the keys *individualID* and *specimenID*.

# Merge Metadata Files on *individualID* and *specimenID*
 
The following example merges the metadata files from the [*MC-CAA* study in the Alzheimer's Disease (AD) Knowledge Portal](https://adknowledgeportal.synapse.org/Explore/Data?QueryWrapper0=%7B%22sql%22%3A%22SELECT%20*%20FROM%20syn11346063%22%2C%22limit%22%3A25%2C%22offset%22%3A0%2C%22selectedFacets%22%3A%5B%7B%22concreteType%22%3A%22org.sagebionetworks.repo.model.table.FacetColumnValuesRequest%22%2C%22columnName%22%3A%22study%22%2C%22facetValues%22%3A%5B%22MC-CAA%22%5D%7D%2C%7B%22concreteType%22%3A%22org.sagebionetworks.repo.model.table.FacetColumnValuesRequest%22%2C%22columnName%22%3A%22dataSubtype%22%2C%22facetValues%22%3A%5B%22metadata%22%5D%7D%5D%7D). See the [Downloading Data Programmatically article]({{ site.baseurl }}{% link _articles/download-data.md %}#R) for steps to download data from the portal in R. In this example, we join the same individual and biospecimen file to both the RNAseq and SNP array assay metadata files. You can think of the assay files as containing a subset of the individuals or biospecimens.

Install the `tidyverse` package to perform data frame manipulations.

```
library(tidyverse)
```
We will create a data frame to store the file names in your working directory that contain the phrase "metadata" under the variable *name*. Included in this data frame is the *.csv* file itself read in with `readr::read_csv` under the variable *files* and the metadata file type, parsed from the file name, under the variable *type*.
```
files <- tibble::tibble(name = list.files(pattern = "metadata")) %>% 
  dplyr::mutate(files = purrr::map(name,
                                   ~ readr::read_csv(., 
                                                     col_types = readr::cols(.default = "c")))) %>% 
  dplyr::mutate(type = purrr::map_chr(name, 
                                     ~stringr::str_split_fixed(., "[_\\.]",5)[2]))
```
The `merge()` helper function subsets the metadata files relevant to the analysis type and joins the metadata files. A `right_join` preserves only the individuals and biospecimens that are characterized in each assay type - RNA-seq and SNP array. 

```
merge <- function(files, assay) {
  subset <- files[grepl(glue::glue("individual|{assay}|biospecimen"), files$name),]
  order <- c(which(subset$type == "individual"), 
             which(subset$type == "biospecimen"),
             which(subset$type == "assay"))
  order <- subset[order,]
  metadata <- order$files %>% 
    reduce(., dplyr::right_join)
  metadata
}
```
The result is two data frames named `RNAseq` and `snpArray` that contain all of the relevant metadata for each analysis.

```
RNAseq <- merge(files, "RNAseq")
snpArray <- merge(files, "snpArray")
```
