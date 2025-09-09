# OpenMRS PATH DRC Content Packages

This project contains the various content packages for the PATH DRC project. There is one "main" content package, called the "Base Content Package" which contains all of the metadata shared across different sites. This includes the form defintions, concept dictionary, drug formulary, etc. In addition to this, there is a per-site content package which includes site-specific metadata. For the most part, this will be site-specific factors, like the locations available in the site, the list of queues configured for that site, cashpoints, and other facility-specific configurations required.

## Organization

Each content package is a full content package as explained [on the OpenMRS Wiki](https://openmrs.atlassian.net/wiki/spaces/docs/pages/183795733/OpenMRS+Content+Packages+Templates) and can contain any supported metadata described there. However, metadata that should aplpy across all sites should generally be managed via the "Base Content Package". The content packages will be expanded into a single folder, so that any content included in the facility-specific metadata will override metadata from the "Base Content Package", assuming that the files have the same name. It is recommended that for metadata domains consisting of CSV files, each domain has only a single CSV file to facilitate overriding where necessary.

## Adding a new facility

Adding a new facility should be relatively straight-forward. Each content package is a folder containing certain metadata for producing the content-package, which consists of a file called `pom.xml` (the Maven project descriptor), `content.properties`, a content-package specific descriptor which describes the content package and any backend modules it depends on, and `assembly.xml` which describes how to put together the zip file that is published and distributed.

To create a new content package, first create a new top-level folder named after the facility. Then copy the `pom.xml`, `assembly.xml` and `content.properties` files from an existing facility content package, then in a text editor, open the `pom.xml` file. You will need to adjust at least the following properties there:

* `<name>...</name>` This is the human-readable name for the content package. We recommend using a string like "Path DRC Content Package - Akram" (where "Akram" is the name of the facility)
* `<artifactId>...</artifactId>` This is the computer-readable name for the content package. This must be a string like `path.drc-akram` (where `akram` is the name of the facility in all lower-case; spaces should be replaced by `-`s)
* Optionally `<description>...</description>` This is a human-readable longer description of what the package contains. We recommend a string like "Content Package containing customized metadata for the PATH DRC instance".

Once that is done, create a folder called `configuration` with two subfolders, `backend_configuration` and `frontend_configuration`. Within the `backend_configuration` you can define whatever domains make sense for this facility, though you may want to copy files from an existing setup and change them. **If you copy files from an existing setup and modify them, please remember to change the UUID value for every line in a CSV that has a UUID. It is important that these UUIDs be unique across implementations.**. Within the `frontend_configuration` folder, you can supply any facility-specific configuration JSONs or assets (in a sub-folder called `assets`). If either of these folders is empty, please create an empty file called `.gitkeep` in the folder.

Once the files are arrange, open the `.github/workflows/main.yml` and find the YAML block labelled `id: filter-paths`. Inside this block there is a `with:` section that contains a section called `filters`, which defines a filter for each facility. You must add your new facility name to this block at the end. For example "Akram" yields a entry like this:

```yaml
Akram: ./Akram/**
```

A facility name with spaces should be represented like this:

```yaml
"My Facility": ./My\ Facility/**
```
It is important that the entry name exactly match the folder name.

Once that is done, open the `.github/workflows/build-all.yml` file. This file is used for manually-triggered CI publication. In this file, find the `working-directory:` entry that comes under the `matrix:` entry and add your facility to this list, separated by other with commas and quoted in quotation marks if it has any spaces.

## Distribution

By default the CI jobs in this repo will publish updated versions of the packages as new changes are made to the GitHub Packages repository. The content packages will then be downloaded from there and included in the appropriate distribution.
