# 1 Write a simple TARQL query :tomato:  

Let's start with [`./data/raw_data/farming.csv`](https://github.com/erikaduan/sparql_with_hobbits/blob/main/data/raw_data/farming.csv). This dataset contains column names in the first row and has a relatively structured format (values in each column are atomic and of the same type with no missing values).

The dataset, however, does not contain a primary key for tracking individual product harvests. A primary key for `Date-Farmer` and foreign key for `Farmer` would need to be created to link this table to other tables within a relational database.

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

We can convert `./data/raw_data/farming.csv` into a simple RDF with:  
1. `Product` as subject and assigned an IRI.    
2. `Count` converted into an integer type and `Date` converted into a date type.  

The script execution sequence should be read in the order of `FROM <...> WHERE {...} CONSTRUCT {...}`.   

```
PREFIX ex: <http://www.hobbiton.com/schema>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

CONSTRUCT {
  # Generate IRI for produce and add associated columns
  # The shorthand ; is used for triples which share the same subject 
  ?product_iri ex:product ?Product ;
    ex:farmer ?Farmer ;
    ex:date ?date ;
    ex:count ?count .
}

FROM <file:farming.csv>

WHERE {
  BIND(REPLACE(STR(?Product),"[ ]","_") AS ?product)
  BIND(IRI(CONCAT('http://www.hobbiton.com/schema/product#', ?product)) AS ?product_iri)
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

Let's examine our TURTLE RDF [output](https://github.com/erikaduan/sparql_with_hobbits/blob/main/data/clean_data/farming_unstructured.rdf) from exercise 1.1.   

Take the following original rows from `./data/raw_data/farming.csv`.   
| Date | Farmer | Product | Count |  
| -----|--------|---------|-------|  
| 2021-09-12 | Fredegar Bolger | potato | 83 |  
| 2021-09-20 | Fredegar Bolger | potato | 65 |  
| 2021-09-12 | Berylla Baggins | potato | 48 |

The RDF corresponding to these three rows is captured below. Notice how multiple rows of data can be stored under the same subject IRI i.e. `product_iri`. #TODO ask what happens when there is no unique key in the csv to convert.  

```
<http://www.hobbiton.com/schema/product#potato>
        ex:product  "potato" ;
        ex:farmer   "Fredegar Bolger" ;
        ex:date     "2021-09-12"^^xsd:date ;
        ex:count    83 ;
        ex:product  "potato" ;
        ex:farmer   "Fredegar Bolger" ;
        ex:date     "2021-09-20"^^xsd:date ;
        ex:count    65 ;
        ex:product  "potato" ;
        ex:farmer   "Berylla Baggins" ;
        ex:date     "2021-09-12"^^xsd:date ;
        ex:count    48 .
```

Knowledge graphs make use of different universally defined ontologies to describe relationship types between different nodes. The RDF and RDFS ontologies are deliberately limited to defining class membership and object properties to support flexibility. Relational structure is built using more expressive ontologies, like [OWL2](https://www3.cs.stonybrook.edu/~pfodor/courses/CSE595/L05_Web_Ontology_Language_OWL2.pdf).

The OWL2 ontology applies to classes, properties and instances of classes (individuals). We can use the OWL2 ontology to define more precise relationships between our columns (subjects or classes and predicates or properties) in `./data/raw_data/farming.csv`.  

>**Note**
> When creating knowledge graphs, there is a compromise between enabling computationally efficient reasoning support and being sufficiently expressive.
<br/>

**OWL2 class axioms**  
+ Subjects must be assigned as `rdf:type owl:Class` to use OWL2 class axioms.  
+ `owl:Thing` is the most general class i.e. every OWL individual is a member of this class and every object of `owl:Class` is a subclass of `owl:Thing`. 
+ `LuxuryApartment rdf:type owl:Class ; rdfs:subClassOf :Apartment`. A subclass relation is defined using RDFS.  
+ `Apartment rdf:type owl:Class ; owl:equivalentClass dbpedia:Apartment`. An equivalence between two class defines that every instance of a specified class is also an instance of its equivalent class. Equivalence is useful for mapping subjects from different ontologies to each other.  
+ `LuxuryApartment rdf:type owl:Class ; owl:disjointWith RunDownApartment`. A disjoint between two classes defines the logic that instances of class `LuxuryApartment` cannot also belong to class `RunDownApartment`. 
+ `UnFurnishedApartment rdfs:subClassOf Apartment; owl:complementOf FurnishedApartment`. The complement of class `UnFurnishedApartment` comprises all instances which do not belong to this class. The union of a class and its complement is therefore `owl:Thing`. This means that we also imply that `Apartment rdf:type owl:Class` and `owl:Thing` are equivalent, which may not be our original intention.  
+ `Apartment rdf:type owl:Class ; owl:unionOf ( LuxuryApartment RunDownApartment )`. The union of a class defines the logic that `Apartment` is equivalent to two or more separate classes i.e. `LuxuryApartment` and `RunDownApartment`. An extension of `owl:unionOf` is `owl:disjointUnionOf`, which defines the logic that two classes are subclasses of `Apartment` and disjoint with respect to each other.  
+ `LuxuryApartment rdf:type owl:Class ; owl:intersectionOf ( GoodLocationApartment LargeApartment )`. The intersection defines the logic that every instance of class `LuxuryApartment` is the instance of the intersection of classes `GoodLocationApartment` and `LargeApartment`.   

**OWL2 property axioms** 
+ Predicates must be assigned as `rdf:type owl:ObjectProperty` to use OWL2 property axioms. 
+ `isPartOf rdf:type owl:ObjectProperty ; rdf:type owl:TransitiveProperty`. A transitive property defines the logic that if `apartment_a isPartof complex_a` and `bathroom_a isPartof apartment_a`, then `bathroom_a isPartof complex_a`.  
+ `isAdjacentTo rdf:type owl:ObjectProperty ; rdf:type owl:SymmetricProperty`. A symmetric property defines the logic that if `a isAdjacentTo b`, then `b isAdjacentTo a`. An extension of this logic is that `owl:AsymmetricProperty` also exists, i.e. `isCheaperThan rdf:type owl:ObjectProperty ; rdf:type owl:AsymmetricProperty ; rdf:type owl:TransitiveProperty .`  
+ `isRentedBy rdf:type owl:ObjectProperty ; owl:inverseOf rents`. An inverse property defines the logic that if `apartment_a isRentedBy person_b`, then `person_b rents apartment_a`. More importantly, domain and range are inherited from the inverse property i.e. `IsRentedby` has `rdfs:domain ?Apartment` and `rdfs:range ?Person`. In OWL2, only object properties have an inverse.   
+ `isPartOf rdf:type owl:ObjectProperty ; owl:equivalentProperty dbpedia:partOf`. An equivalence between two properties defines the logic that if `a isPartOf c`, then `a partOf c` always is true. Equivalence is useful for mapping objects from different ontologies to each other.   
+ `rents rdf:type owl:ObjectProperty ; owl:disjointProperty owns`. A disjoint between two properties defines the logic that if `person_a rents apartment_a`, `person_a owns apartment_a` cannot also be true.  
+ `hasNumberOfRooms rdf:type owl:DatatypeProperty ; rdf:type owl:FunctionalProperty`. A functional property defines the logic that `hasNumberOfRooms` can only have one value at most associated with it.  

## Exercise 1.2  

We can convert `farming.csv` into a more structured RDF:   
1. Assign `product_iri`, `product` and `farmer` as classes.  
2. Rename `date` and `count` as the properties `producedOn` and `hasCount` and create a new `producedBy` property which links `product` to `farmer`.  
3. Introduce extra class and property logic using the OWL2 ontology i.e. `hasCount rdf:type owl:FunctionalProperty` to define that `hasCount` can always only have one value.  

```
PREFIX ex: <http://www.hobbiton.com/schema>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <https://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#> 

CONSTRUCT {
  # Specify product_iri, product and farmer as classes
  # Specify product_iri and product class equivalence 
  ?product_iri rdf:type owl:Class ;
    ex:producedBy ?Farmer ;
    ex:producedOn ?date ;
    ex:hasCount ?count .
  
  ?Product rdf:type owl:Class ;
    owl:equivalentClass ?product_iri .  

  ?Farmer rdf:type owl:Class . 

  ex:producedBy rdf:type owl:ObjectProperty ; 
    rdf:type owl:TransitiveProperty ; 
    owl:inverseOf ex:produces ; 
    rdf:type owl:FunctionalProperty . 
  
  ex:producedOn rdf:type owl:ObjectProperty ; 
    rdf:type owl:FunctionalProperty . 

  ex:hasCount rdf:type owl:ObjectProperty ; 
    rdf:type owl:FunctionalProperty . 
}

FROM <file:farming.csv>

WHERE {
  BIND(REPLACE(STR(?Product),"[ ]","_") AS ?product)
  BIND(IRI(CONCAT('http://www.hobbiton.com/schema/product#', ?product)) AS ?product_iri)
  BIND(xsd:integer(?Count) AS ?count)
  BIND(xsd:date(?Date) AS ?date)
}
```

Use the command line to navigate to `./scripts` and run the TARQL script using the following code below.   

```
tarql --write-base convert_farming_csv_structured.rq ../data/raw_data/farming.csv > ../data/clean_data/farming_structured.rdf
```

The output TURTLE file can then be viewed as [`./data/clean_data/farming_structured.rdf`](https://github.com/erikaduan/sparql_with_hobbits/blob/main/data/clean_data/farming_structured.rdf).  
<br/>


# Link multiple datasets to create a structured RDF  
<br/>

## Exercise 1.3  
<br/>

# Resources 
+ Chapter 2 of [Learning SPAQRL Version 2](http://www.learningsparql.com/) by Bob DuCharme.  
+ [Stardog tutorial](https://docs.stardog.com/tutorials/rdf-graph-data-model#rdf-schema) describing the properties of an RDF schema.  
+ [LinkedIn post](https://www.linkedin.com/pulse/using-tarql-convert-excel-spreadsheets-rdf-kurt-cagle/) on installing and using the `tarql` package.    
+ Blog posts [here](https://www.bobdc.com/blog/tarql/) and [here](https://www.semanticarts.com/how-to-sparql-with-tarql/) on converting `csv` files to `rdf` files using the `tarql` package.