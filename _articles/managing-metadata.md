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
While reading in each metadata file with `read_csv`, specify the column types as character with "c". Consistent column types ensure common variables can be joined. This code joins data frames using all variables in common across the individual, biospecimen and assay metadata files. A `right_join` preserves only the individuals and biospecimens that are characterized in each assay type - RNA-seq and SNP array. 

```
individual <- read_csv("MC-CAA_individual_human_metadata.csv", col_types = cols(.default = "c"))
biospecimen <- read_csv("MC-CAA_biospecimen_metadata.csv", col_types = cols(.default = "c"))
rnaseq_assay <- read_csv("MC-CAA_assay_RNAseq_metadata.csv", col_types = cols(.default = "c"))
snparray_assay <- read_csv("MC-CAA_assay_snpArray_metadata.csv", col_types = cols(.default = "c"))

RNASeq <- reduce(
  list(individual, biospecimen, rnaseq_assay),
  right_join
)

snpArray <- reduce(
  list(individual, biospecimen, snparray_assay),
  right_join
)
```
