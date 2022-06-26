# SPARQL with hobbits :woman_farmer::farmer::tomato:	
This repository aims to equip users with enough TARQL and SPARQL knowledge so they can avoid being coerced into taking an unexpected (data) journey.    	

# Project setup  
This project requires access to:  
+ [TARQL](https://github.com/tarql/tarql/releases), which is a command line tool for converting CSV files to RDF using SPARQL 1.1 syntax. A guide to installing TARQL for Windows environments is found [here](https://www.linkedin.com/pulse/using-tarql-convert-excel-spreadsheets-rdf-kurt-cagle/) and includes a recommendation to use the [`chocolatey`](https://community.chocolatey.org/) package manager for Windows users.   
    + Install `chocolatey` as administrator following instructions on https://docs.chocolatey.org/en-us/choco/setup 
    + Install `openjdk11` as administrator. This version is still used by most applications although long term service has moved to Java version 17.    
    + Install `maven` as administrator.    
    + Clone `tarql` package from Github and navigate to the tarql directory.    
    + Assemble `tarql` maven package.  
    + Add the path of the tarql bin file `.tarql/target/appassembler/bin/` to your environment's PATH variable so your system can locate the tarql application and make `tarql` available as a direct command from the command line. Note that `setx` sets the variable in the local user environment.  
    + Installation success can be tested by inputting `tarql --help` in the command line.  

    ```
    choco install openjdk11
    choco install maven  
    git clone http://github.com/tarql/tarql
    cd tarql 
    mvn package appassembler:assemble
    setx PATH "C:\Windows\System32\tarql\target\appassembler\bin" 
    ```

+ The Visual Studio [`Stardog RDF Grammars`](https://marketplace.visualstudio.com/items?itemName=stardog-union.stardog-rdf-grammars) IDE support extension, which is also hosted on [GitHub](https://github.com/stardog-union/stardog-vsc/tree/master/stardog-rdf-grammars).  
+ A [Stardog cloud](https://www.stardog.com/stardog-cloud/) free tier account, which provides access to a GUI interface for running SPARQL queries.  

Where possible, we will work in a code editor or directly in the command line. 

# Scenario  
## Mapping Hobbiton's food supply chain    
Food is inarguably the most important resource in Hobbiton. Goods are arguably assigned to either the agriculture, retail trade, wholesale trade, food services, transport or recreation industry, depending on which hobbit economist is doing the arguing.  

To incorporate these complexities and resolve subject matter in-fighting, we can represent Hobbiton food supply chains using a knowledge graph. 

## Datasets   
The following datasets are collected in Hobbiton and stored in `./data/raw_data`:  
+ `farming.csv` - contains the date, farmer, product and count of vegetable produce.  
+ `shops_register.csv` - contains the date, Shire business ID, trading name, shop owner and trading status of all businesses within the Shire (the region that Hobbiton belongs to).   
+ `general_store.csv` - contains the date, product and number of items sold at the Hobbiton general store.    


# Exercises  
1. Generate a simple RDF with TARQL    
2. Generate more complex RDFs with TARQL   
3. Write a simple SPARQL query  


# Acknowledgements  
+ [Learning SPAQRL Version 2](http://www.learningsparql.com/) by Bob DuCharme.  
+ [Official website](https://tarql.github.io/) containing documentation and tutorials for using the `tarql` package.   
+ [YouTube series](https://www.youtube.com/watch?v=Q5DrZV5wWzo&list=PLoOmvuyo5UAeihlKcWpzVzB51rr014TwD) on semantic web technologies created by OpenHPI Tutorials. 
+ [Analysis of Australia's food supply chain](https://www.awe.gov.au/agriculture-land/farm-food-drought/food/publications/foodmap-a-comparative-analysis) conducted by the Department of Agriculture, Water and Energy (2012).  
+ [Website](https://www.ranker.com/list/hobbit-names/book-keeper) containing list of Hobbit names. 