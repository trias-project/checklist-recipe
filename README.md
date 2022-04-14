# Checklist recipe<!-- Replace this with the title of your checklist dataset -->

<!-- Delete the following text -->
> 👩🏻‍🍳 This is a template repository for **standardizing thematic species checklist data to Darwin Core using R**. As a result, the rest of the README is a template as well. To use this repository for your own checklist data, [read the recipe](https://github.com/trias-project/checklist-recipe/wiki). Happy cooking!

## Rationale

<!-- This section gives a quick description of what this repository is for. At least update the "... the data of (blank) ..." or edit as you see fit. -->

This repository contains the functionality to standardize the data of <!-- Title of checklist data or reference to source, e.g. "[Zieritz et al. (2014)](https://doi.org/10.3897/neobiota.23.5665)" --> to a [Darwin Core Archive](https://www.gbif.org/darwin-core) that can be harvested by [GBIF](https://www.gbif.org/).

## Workflow

<!-- This section describes how we go from raw data to standardized Darwin Core data -->

[source data](data/raw) <!-- Additionally, you can write here where that raw data came from, e.g. "(downloaded as [Supplementary Material 1](http://neobiota.pensoft.net//lib/ajax_srv/article_elements_srv.php?action=download_suppl_file&instance_id=31&article_id=4007))" --> → Darwin Core [mapping script](src/dwc_mapping.Rmd) → generated [Darwin Core files](data/processed)

## Published dataset

<!-- This section provides links to the published dataset. Obviously, you'll only be able to add those links once you have published your dataset. 😋 -->

* [Dataset on the IPT](<!-- Add the URL of the dataset on the IPT here -->)
* [Dataset on GBIF](<!-- Add the DOI of the dataset on GBIF here -->)

## Repo structure

<!-- This section helps users (and probably you!) to find their way around this repository. You can leave it as is, unless you're starting to adapt the structure a lot. -->

The repository structure is based on [Cookiecutter Data Science](http://drivendata.github.io/cookiecutter-data-science/) and the [Checklist recipe](https://github.com/trias-project/checklist-recipe). Files and directories indicated with `GENERATED` should not be edited manually.

```
├── README.md              : Description of this repository
├── LICENSE                : Repository license
├── checklist-recipe.Rproj : RStudio project file
├── .gitignore             : Files and directories to be ignored by git
│
├── src
│   ├── dwc_mapping.Rmd    : Darwin Core mapping script
│   ├── _site.yml          : Settings to build website in docs/
│   └── index.Rmd          : Template for website homepage
│
├── docs                   : Repository website GENERATED
│
└── data
    ├── raw                : Source data, input for mapping script
    └── processed          : Darwin Core output of mapping script GENERATED
```

## Installation

<!-- This section is for users who want to download/adapt your checklist repository. You can leave it as is. -->

1. Click on `Use this template` to create a new repository on your account
2. Open the RStudio project file
3. Open the `dwc_mapping.Rmd` [R Markdown file](https://rmarkdown.rstudio.com/) in RStudio
4. Install any required packages
5. Click `Run > Run All` to generate the processed data
6. Alternatively, click `Build > Build website` to generate the processed data and build the website in `docs/` (advanced)

## Contributors

<!-- This section lists everyone who contributed to this repository. You can maintain a manual list here or reference the contributors on GitHub. -->

[List of contributors](<!-- Add the URL to the GitHub contributors of your repository here, e.g. https://github.com/trias-project/checklist-recipe/contributors -->)

## License

<!-- The license is the open source license for the code and documentation in this repository, not the checklist data (that you can define in dwc_mapping.Rmd). As your repository is based on https://github.com/trias-project/checklist-recipe, we'd like it if you kept the open and permissive MIT license. You're welcome to add your name as a copyright holder (because your are for your own code contributions), which you can do in the LICENSE file. If you want to release your repository under a different license, please indicate somehow that it was based on https://github.com/trias-project/checklist-recipe. We know, licenses are complicated. See https://choosealicense.com/ for more information. -->

[MIT License](LICENSE) for the code and documentation in this repository. The included data is released under another license.
