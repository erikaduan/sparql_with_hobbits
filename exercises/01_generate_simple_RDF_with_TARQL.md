# 01 Write a simple TARQL query  

Let's start with `farming.csv` from `./data/raw_data`. This dataset contains column names in the first row and has a structured format (values in each column are atomic and of the same type with no missing values). The dataset, however, lacks a primary key for tracking different vegetable product counts.          

| Date | Farmer | Product | Count | 
| -----|--------|---------|-------|
| 4/09/2021 | Bolger | pumpkin | 29 | 
| 4/09/2021	| Bolger | tomato | 62 | 

Since we are interested in constructing a food supply chain, we might be interested in restructuring relationship where:  
+ We focus on the product as a subject i.e. who it is produced by, when and where it is produced and how many units.
+ We also care about linking the farmer back to a farm or company and whether this is a local or overseas product.    

## Create a simple Resource Description Framework (RDF) data model  

We can convert `farming.csv` into a simple RDF where only data type is encoded. To do so, we need to think about the following steps:  

1. Chose appropriate Internationalized Resource Identifiers (IRIs). An IRI is really just a consistent naming convention (using unicode strings) to identify nodes and edges in an unambiguous way. We declare the IRI naming convention via the `PREFIX` section of our TARQL code. We can use naming conventions from multiple IRIs in a single script i.e. use `xsd:` from the [XML schema](https://www.w3.org/TR/rdf11-concepts/#xsd-datatypes) to denote data type.    

  ```
  # A user-defined default namespace for the majority of nodes and the edges in the graph
  # http://hobbiton.com/food_supply_chain/ would host all term definitions used in nodes and edges
  PREFIX : <http://hobbiton.com/food_supply_chain/> 

  # A built-in RDF namespace for describing instances of a class i.e. rdf:type 
  PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

  # A built-in RDF namespace for describing classes i.e. rdfs:subClassOf 
  PREFIX rdfs:<https://www.w3.org/2000/01/rdf-schema#>

  # The RDF namespace for describing data types is based on the XML Schema i.e. xsd:integer() 
  PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
  ```
2. Casting**: TARQL treats all values as strings. To apply business rules and operation constraints, we can cast values into defined data types using `BIND(xsd:type(...) AS ?new_entity)`.   

The TARQL script is shown below. The execution sequence should be read as `FROM ... WHERE ... CONSTRUCT` columns.   

```
PREFIX : <http://www.hobbiton.com/schema#>
prefix xsd: <http://www.w3.org/2001/XMLSchema#>

CONSTRUCT { 
   ?uuid :product ?Product ;
       :date ?d ;
       :farmer ?Farmer ;
       :count ?c . 
}
FROM <file:farming.csv>
WHERE { 
  BIND (UUID() AS ?uuid) 
  BIND (xsd:date(?Date) AS ?d)
  BIND (xsd:integer(?Count) AS ?c)
}
```

To convert `farming.csv` into an RDF, we use the command line to navigate to `.\scripts` and run the TARQL script using the following line of code. The use of `>` enables us to save the output rdf as `farming_unstructured.rdf` in `.\data\clean_data\`.  

```
tarql --write-base convert_farming_csv_unstructured.rq ..\data\raw_data\farming.csv > ..\data\clean_data\farming_unstructured.rdf
```





# Resources 

+ https://docs.stardog.com/tutorials/rdf-graph-data-model#rdf-schema

+ [LinkedIn post](https://www.linkedin.com/pulse/using-tarql-convert-excel-spreadsheets-rdf-kurt-cagle/) on installing and using the `tarql` package.    
+ Blog posts [here](https://www.bobdc.com/blog/tarql/) and [here](https://www.semanticarts.com/how-to-sparql-with-tarql/) on converting `csv` files to `rdf` files using the `tarql` package.  