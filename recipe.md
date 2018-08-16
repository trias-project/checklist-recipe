---
title: "Checklist recipe"
author: 
- Damiano Oldoni
- Peter Desmet
- Lien Reyserhove
date: "`r Sys.Date()`"
output:
  html_document:
    toc: true
    toc_depth: 3
    toc_float: true
    number_sections: true
---

# Ingredients

# Recipe core principle

We want to map our input data to a Darwin Core core or extension file. 
Mapping = assigning content to the Darwin Core terms. This content could be:

a. Static: the value of this Darwin Core is independent of the record, i.e. the content remains unchanged over the whole dataset. This mostly concerns metadata fields in the taxon core; or, wi
b. Unalterd (relative to the input data): Darwin Core terms for which the content is an exact copy of the corresponding field in the input data
c. Altered (relative to the input data): Altered values are used for Darwin Core terms for which the content in `input data` is used as a basis, but it needs to be standardized


The mapping process to Darwin Core Archive is a sequential process. In each step, we map one specific Darwin Core term. For this, we use the following workflow:

1. We always make a copy of the original input dataframe when we generate a core or extension file. This is good practise as you leave the original dataframe untouched and allows you to always return to the orginal data when needed
2. We copy the input data to a new dataframe `taxon` or `distribution`. We do this as `input_data` is the basis for both the  `taxon` core and the `distribution` extension. For each of these files, input_data is the starting point. We never start mapping from the original file as we want to keep this during the whole process. 
2. We add the Darwin Core terms to the input data frame one by one, using the `mutate()` function from the package dplyr. In this way, the generated core or extension file is prolongued with one Darwin Core term in each mapping step. In this stage, the core or extension is a combination of the input data and the newly mapped Darwin Core terms.
3. We remove the input data columns after all Darwin Core terms have been mapped. 


# Data transformation utensils

To add the Darwin Core terms to the input data, we need (a combination of) three basic data transformation utensils provided by the tidyverse package: 

1. mutate()
2. recode()
3. case_when()

## mutate()

For **each of the three** abovementioned types of Darwin Core mapping, we use the `mutate()` function. `mutate()` adds new variables to the dataset and preserves the existing ones. This function is essential in our sequential mapping process, where we add Darwin Core terms to the input data frame, .

The basic code for each of three mappings using `mutate()` looks like this:

```
input_dataframe %<>% mutate(new_column_name = ... )
```

- `input_dataframe`: the dataframe containing the input data
- `%<>% mutate()`: indicating the addition of a new variable to the input data using `mutate()`
- `new_column_name`: name of the new column, in our case a Darwin Core term

In its most basic application here, `mutate()` adds a new Darwin Core terms to the checklist that remains unchanged over the whole checklist, i.e. the terms are **static**. This mostly concerns record-level terms (metadata) in the taxon core. This is how you use the function in our recipe:

```
input_dataframe %<>% mutate(darwin_core_term = "static value" )
```

Some examples:

```
taxon %<>% mutate(language = "en")
```

```
taxon %<>% mutate(rightsHolder = "Research Institute for Nature and Forest")
```

```
taxon %<>% mutate(license = "http://creativecommons.org/publicdomain/zero/1.0/")
```

We can also use this function for other, non-metadata fields:

```
taxon %<>% mutate(kingdom = "Animalia")
```

```
taxon %<>% mutate(nomenclaturalCode = "ICZN")
```

In our mapping recipe, we use this approach for the following Darwin Core terms: `language`, `license`, `rightsHolder`, `datasetID`, `institutionCode` and `datasetName`(see Table x)


Another way to use `mutate()` is to map **unaltered** Darwin Core terms. In this case, the content of the Darwin Core term is an exact copy of the content of one input variable. To code is very similar to the code for mapping static variables, with the only difference that the second part of the expression refers to the column name of the input dataframe you want to map:

```
input_dataframe %<>% mutate(darwin_core_term = input_variable)
```

We 

Some examples

```
taxon %<>% mutate(taxonID = input_taxon_id)
```

```
taxon %<>% mutate(scientificName = input_scientific_name)
```

In our mapping recipe, we use this approach for the following Darwin Core terms:`taxonID`, `scientificName`, `kingdom`, `countryCode` and `occurrenceStatus`.

Next to mapping static and unaltered fields, we here show you how to map **altered** Darwin COre fields. Altered values are used for Darwin Core terms for which the content in the input dataset is used as a basis, but it needs to be standardized. This applies to Darwin Core terms for which we use a vocabulary or where we want to transform for clarity or to correct obvious mistakes. FOr the mapping of these fields, we need to introduce two new functions, both used in combination with `mutate()`: `recode()` and `case_when`.

## recode()

This function replaces an originial value of the input variable with a values in the Darwin Core field. This is a one-to-one mapping, i.e. one specified value of the input variable is replaced by one new value in the newly generated Darwin Core field. This is especially interesting for the following cases:

- Correction of typo's in the original content
- Change abbreviations to full names or vice versa.

In the mapping process, we always combine this function with mutate(), as we want to keep the original variables and add the altered Darwin Core field to the input dataset. 

The code for this function looks like this:

```
input_dataframe %<>% mutate(darwin_core_term = recode(input_variable,
    "inutput_value_1" = "dwc_value_1",
    "old_value_2" = "dwc_value_2"))
```

In this case, you specify the specific value of the input variable between quotes in the left-hand side of the equation, and the specific value of the mapped Darwin Core (dwc) value in the right-hand side of the equation. We use this approach for 

An example using our checklist recipe:

```
taxon %<>% mutate(taxonRank = recode(input_rankmarker,
  "infrasp."  = "infraspecificname",
  "sp."       = "species",
  "var."      = "variety",
  .missing    = ""
))
```

In this example, the abbreviations in the input variable `input_rankmarker` are recoded to their full names in the Darwin Core field `taxonRank`. In the statement `.missing`, we specify that all missing values from `input_rankmarker`(in this case the abbreviation `agg.`) will be mapped to empty values in the Darwin Core field `taxonRank`. This specifciation is optional and could be omitted. 

SOme other examples:

```
taxon %<>%  mutate(order = recode(input_order, "Veneroidea" = "Venerida")) # Correct typo: Veneroida --> Venerida
```

```
taxon %<>% mutate (phylum = recode(input_phylum, "Crustacea" = "Arthropoda")) # Crustacea is not a phylum; recoded to Arthropoda
```

```
distribution %<>% mutate(description = recode(input_description,
  "Ext.?"     = "Ext.",
  "Ext./Cas." = "Cas.",
  "Cas.?"     = "Cas.",
  "Nat.?"     = "Nat.",
  .missing = ""))
```

## case_when()

This function allows you to map a Darwin Core terms which values are based on (multipele) conditional statements using the input variable(s). This is the case when multipele values in the input variable are mapped to one single value in the Darwin COre field (many-to-one mapping), or when the mapped value depends on the values of multipele input variables. In these cases, the one-to-one mapping using recode() can not be applied. `case_when()` can thus be applied to more complex cases.

The code for this function looks like this:

```
input_dataframe %<>% mutate(darwin_core_term = case_when(
    conditional_statement_1 ~ "dwc_value_1",
    conditional_statement_2 ~ "dwc_value_2"))
```

You can read this as following: if `conditional_statement_1` is true, then map the darwin core term to `dwc_value_1`. If `conditional_statement_2` is true, then map the darwin core term to `dwc_value_2`.
These conditional statement can be very diverse. Some examples:

1. The mapped value depends on the value of multipele input variables:

```
input_dataframe %<>% mutate(darwin_core_term = case_when(
`input_variable_1 == "a" & `input_variable_2 == "b  ~"dwc_value_1"))   # IF the value of input_variable_1 equals "a" AND `input_variable_2` equals "b" THEN map dwc_value_1
```

2. The multipele values in the input variable are mapped to one single value in the Darwin COre field (many-to-one mapping):


In the checklist recipe, we use the `case_when()` function in several examples:

```
distribution %<>% mutate(locality = case_when(
  !is.na(input_locality) ~ input_locality,   # !is.na selects all rows for which `input_locality` is not empty
  input_country_code == "BE" ~ "Belgium",
  input_country_code == "GB" ~ "United Kingdom",
  input_country_code == "MK" ~ "Macedonia",
  input_country_code == "NL" ~ "The Netherlands",
  TRUE ~ "" # In other cases leave empty
))
```

In this case, we specify that `locality` can be mapped to:

- values from the variable `input_locality`, _only when_ this a value is given for this variable (i.e. `input_locality` is not empty)
- Countries `Belgium`, `United Kingdom`, `Macedonia` OR `The Netherlands` when `input_locality` is empty and `input_country_code` equals (respectively) `BE`, `GB`, `MK` OR `NL`.

# Data preparation

The checklist template is an Excel file (`./data/raw/checklist.xlsx`). The import specifications for Excel files are more limited than those for delimited files (`csv`, `tsv`, `txt`). However, we use Excel here as this format is often used to manage datasets. To import Excel files, you can use the `read_excel()` function from the package readxl(), where you specify the path to the xlsx file. **maybe some more information about how to define the path here**. The raw data file is imported as the dataframe `input_data`.

```
input_data <- read_excel(path)
```

The very first step is to inspect whether the checklist has been imported correctly. The function `head()` returns the first lines of the dataframe. In this example the code returns the first five lines:

```
input_data %>% head(n = 5)
```

# Process source data

Befor we start mapping to Darwin Core, `input_data` needs to be cleaned and transformed somewhat. This recipe includes the following steps:

- Clean rows and columns
- Screen all scientific names for potential errors using the [GBIF nameparser](https://www.gbif.org/tools/name-parser)
- Retrieve and add nomenclatural information
- Add taxonRank information
- Generate taxonID

## Tidy data

Your checklist should be tidy. Concretely, this implies that:

1. Each variable forms a column.
2. Each observation forms a row.
3. Each type of observational unit forms a table.

The provided checklist template is a good example of a tidy dataset. For more information on tidy datasets, click [here](http://vita.had.co.nz/papers/tidy-data.html). Starting with untidy data will make the mapping script a lot more complex, with many preparatory steps before you can even start mapping. In a tidy dataset, you should be able to start the mapping immediately. However, in some cases, some small cleaning steps could be required, such as removing empty rows. For this, we use the function `remove_empty()` from the package janitor. It removes all rows from the dataframe that are composed entirely of empty values.

```
input_data %<>% remove_empty("rows")
```

More cleaning steps could be necessary, but are out of scope for this checklist recipe. 
LR: refer to some good tutorials/websites here?

During the mapping, we will sequentially add new Darwin Core terms (see further). To avoid name clashes between the original columns in raw_data and the added Darwin Core columns, we add the prefix `input_` to the column names of raw_data (LR: integrate link)

### Scientific names

The full scientific name of a species could be lengthy (e.g. `Bassia laniflora (S.G. Gmel.) A.J. Scott`). Mistakes could easily be made when entering the names in the template. One way to screen for potential errors is by using the [GBIF nameparser](https://www.gbif.org/tools/name-parser). It disects the scientific name in its different components and checks them against the taxonomic backbone used by GBIF. 

![Output nameparser](src/static/images/output_nameparser.png)

The following information returned by the nameparser function indicates that the scientific name could be correct:

- type = "SCIENTIFIC" AND
- parsed = "TRUE" AND
- parsedpartially = "FALSE"

Information deviating from these criteria could imply that the scientific name is incorrect. 

The "type" field indicates whether or not the scientific name is truly scientific (type = "SCIENTIFIC") or e.g. whether it is not a scientificname of any kind (type = `NO_NAME`). Important to note is that the nameparser not necessarily detects all incorrect scientific names, it just gives you a good idea about which ones could be wrong. 

The `parsed` and `parsedpartially` field indicates whether or not the name parser has parsed the full scientific name, which is not alwas the case. This could be due to spelling errors or when taxonomic, nomenclatural or identification notes are added to the end of the name. In these cases the name will only be parsed partially (parsedpartially = "TRUE") or not at all (parsed = "FALSE").
When a scientific name was not parsed at all

In the checklist recipe, we apply the nameparser function to `input_scientific_names` and screen for possbile errors. We specifically select the scientific names deviating from the abovementioned criteria by using the following filtering function:

```
filter(!(type == "SCIENTIFIC" & parsed == "TRUE" & parsedpartially == "FALSE"))
```

This renders the following output:

![Output nameparser exercise](src/static/images/output_nameparser_exercise.png)

Here, the output indicates that `Acmella agg.` is a scientific name with some informal addition (type = "INFORMAL"). The decision whether or not to implement changes for this name is up to the author of the checklist. In this example, we decided to leave the scientific name unchanged.
In the case of the species `AseroÙ rubra`, the nameparser indicates that it was parsed only partially. This is due to a spelling error, i.e. the species name should be `Asero rubra`. There are two options to correct the scientific name in this case: 

- COrrect the scientific name in the raw data file (recommended)
- apply a cleaning function in R

Although we recommend to clean the scientific name in the [raw data file](), we here provide an [example] on how to clean the scientifc name by applying the recode() function provided by the package `dplyr`.

```
input_data %<>% mutate(variable = recode(variable,
  "scientific_name_after_cleaning" = "scientific_name_before_cleaning"
))
```

In this example, the variable is `input_scientific_name`, the scientific_name_before_cleaning is `"AseroÙ rubra"`, the scientific_name_before_cleaning is `"Asero rubra"`.

### Taxon ranks

Another advantage of the nameparser function is that it provides the [taxonRank](http://rs.tdwg.org/dwc/terms/#taxonRank) information for the taxa listed in the checklist (link). We can retrieve this information and attach it to `input_data` (link to code). 
This field needs some recoding though, which will be specified under the taxonRank mapping in the taxon core.

### Taxon ID

taxonID is a required field in both the taxon core and the distribution extension. Because we need this in both processed files, we add it here to `input_data`. TaxonID is defined as an _identifier for the set of taxon information (data associated with the Taxon class)_. We strongly advise to use a global unique identifier or an identifier specific to the data set. SUch a unique identifier can be generated in R by applying a hash function (using the digest() function provided by the digest package). Essentially, this function generates a randomized code of fixed size, linked to the scientific name. When the scientific names remains the same, the taxonID will remain unchanged as well. 

## Mapping

Even though `input_data` contains all necessary information in a single data frame, a Darwin Core Archive might consist of multiple files, e.g. a core and extensions. We recommend to create the core file first and then the extensions. The mapping process is **sequential**: we add the Darwin Core terms step by step. The Darwin Core terms for each core/extension file can be found on the [GBIF Resources page](http://rs.gbif.org/):

[![rs.gbif.org](src/static/images/darwin_core_taxon.png)](http://rs.gbif.org)

It is good practice to inspect the Darwin Core terms on this webpage one by one to see whether a particular term can be used in your checklist. It's good practice to respect the order of the terms as they listed on the GBIF resource page.

### Create taxon core 

The Darwin Core terms for the taxon core can be found on the [GBIF Resources Taxon Core page](http://rs.gbif.org/core/dwc_taxon_2015-04-24.xml). 
The following terms are required for publication on GBIF: `scientificName`, `taxonID` and `taxonRank`.

Mapping of the taxon core generally starts with the addition of the record level terms which contain metadata about the dataset (which is generally the same for all records). For this, we use the dplyr function mutate() as specified under the section xxxx.

We suggest to use the following record-level terms:

record-level term | defenition | example
--- | --- | ---
[language](http://rs.tdwg.org/dwc/terms/#dcterms:language) | The language of the resource | English 
[license](http://rs.tdwg.org/dwc/terms/#dcterms:license) | A legal document giving official permission to do something with the resource | http://creativecommons.org/publicdomain/zero/1.0/ 
[rightsHolder](http://rs.tdwg.org/dwc/terms/#dcterms:rightsHolder) | A person or organization owning or managing rights over the resource | INBO
[datasetID](http://rs.tdwg.org/dwc/terms/#datasetID) | A (unique) identifier for the set of data | https://doi.org/10.15468/wtda1m
[institutionCode](http://rs.tdwg.org/dwc/terms/#institutionCode) | The name of the institution having ownership of the object(s) or information | INBO
[datasetName](http://rs.tdwg.org/dwc/terms/#datasetName) | The name identifying the data set |  Checklist of non-native freshwater fishes in Flanders, Belgium

After this, we add the specific taxonomic information to the checklist. Depending on the content of the raw data, We suggest to use the following terms:
 
taxon core term | defenition | example
--- | --- | ---
[taxonID](http://rs.tdwg.org/dwc/terms/index.htm#taxonID) | An identifier for the set of taxon information (data associated with the Taxon class)
[scientificName](http://rs.tdwg.org/dwc/terms/index.htm#scientificName) | An identifier for the set of taxon information (data associated with the Taxon class)
[kingdom](http://rs.tdwg.org/dwc/terms/index.htm#kingdom) | The full scientific name of the kingdom in which the taxon is classified
[taxonRank](http://rs.tdwg.org/dwc/terms/index.htm#taxonRank) | The taxonomic rank of the most specific name in the scientificName
[nomenclaturalCode](http://rs.tdwg.org/dwc/terms/index.htm#nomenclaturalCode) | The nomenclatural code under which the scientificName is constructed.

For the mapping of the taxon core, we use **define specific approach**


## Create distribution extension

Many checklists contain information related to species distribution. The Darwin Core terms for the distribution extension can be foud on the [GBIF Resources Species Distribution page](http://rs.gbif.org/extension/gbif/1.0/distribution.xml). Species distribution contains typically:

- Geographical information, `locality` (e.g. _Bariloche, 25 km NNE via Ruta Nacional 40 (=Ruta 237)_), `countryCode` (e.g. _US_, _BE_, _FR_)
- Information about how frequent the species occurs, `occurrenceStatus` (e.g. _present_, _rare_, _absent_)
- Threat status as defined by IUCN, `threatStatus` (e.g. _EX_, _EW_, _CR_)
- Description whether the organism occurs natively, is introduced or cultivated, `esatblishmentMeans` (e.g. _introduced_)

# Push changes to GitHub

You would eventually like to _save_ your mapping not only locally on your computer, but remotely on GitHub too. You have then to _commit_ your work and  _push_ it to GitHub. Doing so, you will sync the GitHub repository with the most updated version on your local machine.

You can do it within RStudio: [hands-on session within RStudio](https://inbo-tutorials.netlify.com/git/rstudio/#/), or you can alternatively use GitHub Desktop: [hands-on session with GitHub Desktop](https://inbo-tutorials.netlify.com/git/desktop/#/).

# Using IPT to publish

GBIF provides an Integrated Publishing Toolkit (IPT) in order to check your checklist and eventualy publish it on GBIF. Follow the guideline as provided by GBIF: [How To Prepare Your IPT Server](https://github.com/gbif/ipt/wiki/IPTServerPreparation.wiki). You will likely have to install a servlet as Tomcat and setup a virtual host name. If you are member of an organization, contact first your IT team.

# Examples

Based on this recipe we were able to publish several checklists on GBIF. Some examples:

1. [Checklist of non-native freshwater fishes in Flanders](https://trias-project.github.io/alien-fishes-checklist/)
2. [Inventory of alien macroinvertebrates in Flanders, Belgium](https://github.com/trias-project/alien-macroinvertebrates)
3. [Catalogue of the Rust Fungi of Belgium](https://github.com/trias-project/uredinales-belgium-checklist)

Feel free to use the provided documentation as additional learning material.
