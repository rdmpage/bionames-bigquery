# bionames-bigquery

## Export data from MySQL

Export data from MySQL in CSV format, note that we need to handle NULL values properly otherwise BigQuery will complain.

```
SELECT 
IFNULL(id, ""), 
IFNULL(cluster_id, ""), 
IFNULL(nameComplete, ""), 
IFNULL(taxonAuthor, ""), 
IFNULL(journal, ""), 
IFNULL(volume, ""),
IFNULL(spage, ""),
IFNULL(epage, ""),
IFNULL(year, ""),
IFNULL(issn, ""),
IFNULL(isbn, ""),
IFNULL(doi, ""),
IFNULL(pmid, ""),
IFNULL(biostor, ""),
IFNULL(jstor, ""),
IFNULL(cinii, ""),
IFNULL(handle, ""),
IFNULL(pdf, ""),
IFNULL(url, ""),
IFNULL(oclc, "")
INTO OUTFILE "/tmp/names3.csv"
FIELDS TERMINATED BY ',' ENCLOSED BY '"' ESCAPED BY '"'
LINES TERMINATED BY '\n'
FROM names
WHERE publication IS NOT NULL;
```

## Get data into BigQuery

Upload CSV file to Google Cloud Storage https://cloud.google.com/products/cloud-storage/ In this case I created a folder "ion-names" and uploaded "names3.csv".

In BigQuery https://bigquery.cloud.google.com/ add uploaded data (in this case gs://ion-names/names3.csv)

BigQuery needs a schema, in this case:

```
id:string,cluster_id:string,nameComplete:string,taxonAuthor:string,journal:string,volume:string,spage:string,epage:string,year:string,issn:string,isbn:string,doi:string,pmid:string,biostor:string,jstor:string,cinii:string,handle:string,pdf:string,url:string,oclc:string
```

Once imported there are 1,549,152 rows in the database.

## Query


### How many names have a publication with a DOI?

```
SELECT COUNT(id) from [names.names] WHERE doi != "";

196915
```
### How many names have a publication in BioStor?

```
SELECT COUNT(id) from [names.names] WHERE biostor != "";

130792
```

### How many names have any sort of article-level identifier?

```
SELECT COUNT(id) from [names.names] WHERE doi != "" OR biostor != "" OR jstor != "" OR cinii != "" OR pmid != "" OR handle != "" OR pdf != "" OR url != "";

489029

```


### Numbers of names with an identifier, by year

This query builds a table where the columns are counts for each identifier for each year.

```
SELECT year, 
COUNT(id),
COUNT(CASE WHEN doi != "" THEN 1 END) doi,
COUNT(CASE WHEN biostor != "" THEN 1 END) biostor, 
COUNT(CASE WHEN jstor != "" THEN 1 END) jstor,
COUNT(CASE WHEN cinii != "" THEN 1 END) cinii,
COUNT(CASE WHEN pmid != "" THEN 1 END) pmid,
COUNT(CASE WHEN pdf != "" THEN 1 END) pdf ,
COUNT(CASE WHEN url != "" THEN 1 END) url
from [names.names] group by year order by year;
```

Download this as CSV file (need to be using Google Chrome to do this) and generate graphs.

## Digitisation coverage by journal (ISSN)

```
SELECT issn, journal,
COUNT(id) as n,
COUNT(CASE WHEN doi != "" OR biostor != "" OR jstor != "" OR cinii != "" OR pmid != "" OR handle != "" OR pdf != "" OR url != "" THEN 1 END) identifier
from [names.names] 
where issn != ""
group by issn,journal order by n desc;
```

Output used to create Google Sheet https://docs.google.com/spreadsheets/d/1gYf4AAqKJx_nFFriCFgwnSGL2m9cRpxKtd1hauWq8Jw/edit?usp=sharing

## Digitisation coverage by journal (OCLC)

```
SELECT oclc, journal,
COUNT(id) as n,
COUNT(CASE WHEN doi != "" OR biostor != "" OR jstor != "" OR cinii != "" OR pmid != "" OR handle != "" OR pdf != "" OR url != "" THEN 1 END) identifier
from [names.names] 
where oclc != ""
group by oclc,journal order by n desc;
```

Output used to create Google Sheet https://docs.google.com/spreadsheets/d/1gYf4AAqKJx_nFFriCFgwnSGL2m9cRpxKtd1hauWq8Jw/edit?usp=sharing




