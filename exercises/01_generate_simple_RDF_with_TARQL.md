# 01 Write a simple TARQL query  

Let's start with `farming.csv` from `./data/raw_data`. This dataset contains column names in the first row and a very structured format (no missing values, values in each column are atomic and the same type).     

| Date | Farmer | Product | Count | 
| -----|--------|---------|-------|
| 4/09/2021 | Bolger | pumpkin | 29 | 
| 4/09/2021	| Bolger | tomato | 62 | 

Since we are interested in food supply chains, we might be interested in restructuring relationship where: 
+ We care about the product as the main subject i.e. who it is produced by, when it is produced and how many.
+ We also care about linking the farmer back to a farm or company.  

We can first read the dataset with type encoded but minimal information about column relationships. 

```
PREFIX ex: <http://www.hobbiton.com/schema#>
prefix xsd: <http://www.w3.org/2001/XMLSchema#>

CONSTRUCT { 
   ?uuid ex:product ?Product ;
       ex:date ?d ;
       ex:farmer ?Farmer ;
       ex:count ?c . 
}
FROM <file:farming.csv>
WHERE { 
  BIND (UUID() AS ?uuid) 
  BIND (xsd:date(?Date) AS ?d)
  BIND (xsd:integer(?Count) AS ?c)
}
```

We then navigate to `sparql_with_hobbits\scripts` and type the following into the command line. 

```
tarql --write-base convert_farming_csv_unstructured.rq ..\data\raw_data\farming.csv > ..\data\clean_data\farming_unstructured.rdf
```





# Resources 
+ [LinkedIn post](https://www.linkedin.com/pulse/using-tarql-convert-excel-spreadsheets-rdf-kurt-cagle/) on installing and using the `tarql` package.    
+ Blog posts [here](https://www.bobdc.com/blog/tarql/) and [here](https://www.semanticarts.com/how-to-sparql-with-tarql/) on converting `csv` files to `rdf` files using the `tarql` package.  