---
name: california-doj-download-openjustice-dataset
description: >-
  Find a California DOJ OpenJustice criminal-justice dataset (arrests, crimes and clearances,
  homicide, hate crime, use of force, RIPA stop data, deaths in custody...) and download its
  current CSV/XLSX/ZIP distribution and context PDF from the live JSON:API.
api: California DOJ OpenJustice Open Data Portal JSON:API
base_url: https://data-openjustice.doj.ca.gov/jsonapi
auth: none
operations:
  - getResourceIndex
  - listNodeDataset
  - getNodeDataset
  - listFileFile
  - listTaxonomyTermDataType
generated: '2026-09-17'
method: generated
source: openapi/california-attorney-general-openjustice-jsonapi-openapi.yml
---

# Download an OpenJustice dataset

## 1. List the datasets with their files in one call

`listNodeDataset` —

```
GET https://data-openjustice.doj.ca.gov/jsonapi/node/dataset?include=field_source,field_data_type&fields[file--file]=uri,filesize,filemime,filename&fields[taxonomy_term--data_type]=name&sort=title
Accept: application/vnd.api+json
```

- 26 datasets come back in one page (do NOT add `page[limit]=1` — that page is empty with
  `meta.omitted` because the lowest-id node is unpublished).
- Each dataset's `relationships.field_source.data[]` lists file ids; resolve them against
  `included[]` (type `file--file`).
- To narrow by name: `filter[title][operator]=CONTAINS&filter[title][value]=Hate`.
- To narrow by category use `listTaxonomyTermDataType` (Criminal Justice Data, Firearms,
  Other Data, Agency Information) and filter on the relationship:
  `filter[field_data_type.name]=Criminal Justice Data` (verified 2026-09-17: 23 datasets; the
  Firearms term currently has none).

## 2. Pick the distribution

Files carry `filemime` and `filesize`. Data is `text/csv`, `application/zip` (RIPA) or the
OOXML spreadsheet type; the documentation is the `application/pdf` "Context" file that ships
with every dataset. File names end in the year range (`Hate Crimes_2001-2025.csv`) and may carry
a `_0` revision suffix — always read the current name from the API rather than hard-coding it.

## 3. Download

`uri.url` is site-relative. Prefix the data host:

```
GET https://data-openjustice.doj.ca.gov/sites/default/files/dataset/2026-07/Adult_Probation_2003-2025_0.csv
```

Verified 2026-09-17: HTTP 200, `Content-Type: text/csv`, `Content-Length: 1749288`. No auth, no
rate-limit headers, no licence text — cite the DOJ conditions page https://oag.ca.gov/conditions.

## 4. One dataset by id

`getNodeDataset` — `GET /node/dataset/{uuid}?include=field_source`. The id is the JSON:API
UUID, never `drupal_internal__nid`. Unknown UUID → 404 JSON:API error document.

## Rules

- Reads only. Never call `createNodeSignup`, `createNodeSuggestion` or `createNodeBug` without
  a human's explicit instruction: they create records DOJ staff read, are not idempotent and
  cannot be reversed.
- Errors are JSON:API `errors[]`; `status` is a string. A 400 with "Invalid nested filtering"
  means the filter field name is wrong — read `attributes` from one record first.
- Datasets refresh annually (all 26 changed June–July 2026); re-read the file list rather than
  caching URLs across years.
