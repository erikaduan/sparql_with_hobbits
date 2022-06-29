# 1 Write a simple TARQL query :tomato:  

Let's start with [`./data/raw_data/farming.csv`](https://github.com/erikaduan/sparql_with_hobbits/blob/main/data/raw_data/farming.csv). This dataset contains column names in the first row and has a relatively structured format (values in each column are atomic and of the same type with no missing values).  

The dataset, however, does not contain a primary key for tracking different vegetable product counts. A primary key for `Date-Farmer` and foreign key for `Farmer` would need to be created to link this table to other tables within a relational database.  

| Date | Farmer | Product | Count |  
| -----|--------|---------|-------|  
| 2021-09-04 | Bolger | pumpkin | 29 |  
| 2021-09-04 | Bolger | tomato | 62 |  

>**Note**
> The csv file format currently needs to begin with an empty row in order for all columns to be parsed via TARQL.  

Since we are interested in constructing a food supply chain, we might be interested in incoporating data relationships where:  
+ We focus on produce as a subject i.e. who was it produced by, when and where was it produced and in how many units.
+ We would like to link the farmer back to a farm or company and indicate whether this is a local or overseas product.  
<br/>


# 1.1 Create a simple Resource Description Framework (RDF)    

First of all, let's explore how we can transform CSV files into RDF files in preparation for SPAQRL querying. A RDF contains the following interesting properties:  

**Internationalized Resource Identifiers (IRIs)**  
+ An IRI is designed to reference a universally consistent naming convention, so that all nodes and edges are described in an unambiguous way.  
+ An IRI can be a unique URL describing each subject  value i.e.`http://hobbiton.com/schema/Frodo` or a unique project-value tag i.e. `urn:isbn:12345X`. IRIs are also referred to as Universal Resource Identifiers (URIs), although URIs are actually a subset of IRIs.  
+ The IRI naming convention is declared via the `PREFIX` section at the start of our TARQL script. For example, we use `PREFIX xsd:` from the [XML schema](https://www.w3.org/TR/rdf11-concepts/#xsd-datatypes) to denote data type and `xsd:` is a shorthand for `http://www.w3.org/2001/XMLSchema#` in the rest of the script.  
+ A script can reference multiple IRIs i.e. multiple naming conventions (see example below).  
+ In `./data/raw_data/farming.csv`, we would need to generate IRIs for at least products and farmers via the TARQL functions `UUID()` or `IRI()`, similar to the concept of generating a primary key in a relational database.  

```
# A user-defined default namespace for the majority of nodes and the edges in the graph
PREFIX ex: <http://hobbiton.com/schema/> 

# A built-in RDF namespace for describing instances of a class i.e. rdf:type 
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

# A built-in RDF namespace for describing classes i.e. rdfs:subClassOf 
PREFIX rdfs: <https://www.w3.org/2000/01/rdf-schema#>

# The RDF namespace for describing data types is based on the XML Schema i.e. xsd:integer() 
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
```

**Subject-predicate-object triples**  
![](https://github.com/erikaduan/sparql_with_hobbits/blob/main/figures/rdf_properties.svg)

+ RDFs are stored in the form of `?subject ?predicate ?object` triples, as represented above. Subjects can also be thought of as resource identifiers, predicates as property names and objects as property values or resource facts.  
+ Subjects and predicates must therefore exist as IRIs. This is why column names, which are predicates, must always reference a PREFIX in TARQL.  
+ Objects can exist as an IRI, string or other data type i.e. date, integer, BigDecimal or boolean data type. TARQL treats all values as strings by default. To access special arithmetic, boolean or date operations, we cast strings using `BIND(xsd:type(?column_name) AS ?new_name)` or `BIND(STRDT(?column_name, xsd:integer) AS ?new_name)`.  
<br/>


## Exercise 1.1  

We can convert `farming.csv` into a simple RDF with:  
1. `Product` and `Farmers` as subjects and both assigned an IRI.    
2. `Count` converted into an integer type and `Date` converted into a date type.  

The script execution sequence should be read in the order of `FROM <...> WHERE {...} CONSTRUCT {...}`.   

```
PREFIX ex: <http://www.hobbiton.com/schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

CONSTRUCT {
  # Generate IRI for produce and list associated columns
  ?product_iri ex:product ?Product ;
    ex:farmer ?Farmer ;
    ex:date ?date ;
    ex:count ?count .
  
  # Generate IRI for farmer
  ?farmer_iri ex:farmer ?Farmer .
}

FROM <file:farming.csv>

WHERE {
  BIND(REPLACE(STR(?Product),"[ ]","_") AS ?product)
  BIND(IRI(CONCAT('http://www.hobbiton.com/schema/product#', ?product)) AS ?product_iri)
  BIND(IRI(CONCAT('http://www.hobbiton.com/schema/farmer#', ?Farmer)) AS ?farmer_iri)
  BIND(xsd:integer(?Count) AS ?count)
  BIND(xsd:date(?Date) AS ?date)
}
```

To convert `./data/raw_data/farming.csv` into an RDF, we need to use the command line to navigate to `./scripts` and run the TARQL script using the following code below. The use of `>` enables us to save the output rdf as `./data/clean_data/farming_unstructured.rdf`.   

```
tarql --write-base convert_farming_csv_unstructured.rq ../data/raw_data/farming.csv > ../data/clean_data/farming_unstructured.rdf
```

The output TURTLE file can then be viewed as [`./data/clean_data/farming_unstructured.rdf`](https://github.com/erikaduan/sparql_with_hobbits/blob/main/data/clean_data/farming_unstructured.rdf).  

>**Note**
> The output option `--write-base` needs to be used to convert CSV files into Turtle RDFs i.e. the most commonly used RDF serialisation format.  
<br/>


# 1.2 Create a structured RDF data model

Let's examine our TURTLE RDF [output](https://github.com/erikaduan/sparql_with_hobbits/blob/main/data/clean_data/farming_unstructured.rdf) from exercise 1.1. Our simple TARQL script has produced some undesirable RDF artefacts.  

Take the following original rows from `./data/raw_data/farming.csv`.   
| Date | Farmer | Product | Count |  
| -----|--------|---------|-------|  
| 2021-09-12 | Bolger | potato | 83 |  
| 2021-09-20 | Bolger | potato | 65 |  

The RDF corresponding to these two rows is captured below. Undesirable artefacts include:  
+ Data duplication for `Farmer` IRI, as each value for `Farmer` is separately captured.  
+ Potential data duplication for observations where `Farmer` and `Product` are the same, but a different `Date` and `Count` is observed.  

```
<http://www.hobbiton.com/schema/farmer#Bolger>
        ex:farmer  "Bolger" .

<http://www.hobbiton.com/schema/product#potato>
        ex:product  "potato" ;
        ex:farmer   "Bolger" ;
        ex:date     "2021-09-12"^^xsd:date ;
        ex:count    83 .

<http://www.hobbiton.com/schema/farmer#Bolger>
        ex:farmer  "Bolger" .

<http://www.hobbiton.com/schema/product#potato>
        ex:product  "potato" ;
        ex:farmer   "Bolger" ;
        ex:date     "2021-09-20"^^xsd:date ;
        ex:count    65 .
``` 
These problems can be addressed if we explicitly specify relationships between different columns in a CSV.  
 
1. Assign columns as `rdf:type rdfs:Class` to designate a node category i.e. `potato` and `tomato` belong to the same `Product` class. Class hierarchies can also be created with `rdfs:subClassOf`.  
2. Assign column relationships using `rdf:type rdf:Property` to denote a subject-object relationship i.e. `Count` is a property of the `Product` class. Properties are defined in terms of what subject i.e. `rdfs:domain` and object value i.e. `rdfs:range` they link together. 
3. Optional: Store metadata for properties using `rdfs:label`.  
<br/> 


## Exercise 1.2  

We can convert `farming.csv` into a structured RDF with:  
1. `Product` and `Farmers` defined as `rdfs:Class`.    
2. `Count` and `Date` defined as properties of `Product`.    

```
PREFIX ex: <http://www.hobbiton.com/schema>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <https://www.w3.org/2000/01/rdf-schema#>

CONSTRUCT {
  # Specify product_iri as a Class and specify subject-object relationships
  ?product_iri rdf:type rdfs:Class ;
    ex:isProduct ?Product ;
    ex:producedBy ?farmer_iri ;
    ex:producedOn ?date ;
    ex:hasCount ?count .

  # Duplicate information
  # ex:isProduct a rdf:Property ;
  #   rdfs:domain ?product_iri ;
  #   rdfs:range ?product .

  # Specify farmer_iri as a Class and specify subject-object relationships
  ?farmer_iri rdf:type rdfs:Class ;
    ex:isFarmer ?Farmer .

  # Duplicate information  
  # ex:isFarmer a rdf:Property ;
  #   rdfs:domain ?farmer_iri ;
  #   rdfs:range ?Farmer .
}

FROM <file:farming.csv>

WHERE {
  BIND(REPLACE(STR(?Product),"[ ]","_") AS ?product)
  BIND(IRI(CONCAT('http://www.hobbiton.com/schema/product#', ?product)) AS ?product_iri)
  BIND(IRI(CONCAT('http://www.hobbiton.com/schema/farmer#', ?Farmer)) AS ?farmer_iri)
  BIND(xsd:integer(?Count) AS ?count)
  BIND(xsd:date(?Date) AS ?date)
}
```

To convert `farming.csv` into an RDF, we need to use the command line to navigate to `./scripts` and run the TARQL script using the following code below.   

```
tarql --write-base convert_farming_csv_structured.rq ../data/raw_data/farming.csv > ../data/clean_data/farming_structured.rdf
```

The output TURTLE file can then be viewed as [`./data/clean_data/farming_structured.rdf`](https://github.com/erikaduan/sparql_with_hobbits/blob/main/data/clean_data/farming_structured.rdf).  
<br/>


# Resources 
+ Chapter 2 of [Learning SPAQRL Version 2](http://www.learningsparql.com/) by Bob DuCharme.  
+ [Stardog tutorial](https://docs.stardog.com/tutorials/rdf-graph-data-model#rdf-schema) describing the properties of an RDF schema.  
+ [LinkedIn post](https://www.linkedin.com/pulse/using-tarql-convert-excel-spreadsheets-rdf-kurt-cagle/) on installing and using the `tarql` package.    
+ Blog posts [here](https://www.bobdc.com/blog/tarql/) and [here](https://www.semanticarts.com/how-to-sparql-with-tarql/) on converting `csv` files to `rdf` files using the `tarql` package.