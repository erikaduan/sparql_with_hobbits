# SPARQL with hobbits :woman_farmer::farmer::tomato:	
This repository aims to equip users with enough TARQL and SPARQL knowledge so they can avoid being coerced into taking an unexpected (data) journey.    	


# Windows project setup  
This project requires access to:  
+ [TARQL](https://github.com/tarql/tarql/releases), which is a command line tool for converting CSV files to RDF using SPARQL 1.1 syntax. A guide to installing TARQL for Windows is found [here](https://www.linkedin.com/pulse/using-tarql-convert-excel-spreadsheets-rdf-kurt-cagle/) and includes a recommendation to use the [`chocolatey`](https://community.chocolatey.org/) package manager for Windows users.   
    
    + Install `chocolatey` using the command prompt as administrator, following instructions on https://docs.chocolatey.org/en-us/choco/setup.  
    + Install `openjdk11` as administrator. This version is still used by most applications although long term service has moved to Java version 17.    
    + Install `maven` as administrator.    
    + Clone `tarql` package from Github and navigate to the tarql directory as administrator.   
    + Assemble `tarql` maven package as administrator.  
    + Add the path of the tarql bin file `.tarql/target/appassembler/bin/` to your environment's PATH variable as administrator, so your system can locate `tarql` and make it available through the command line. Note that `setx` sets the variable in the local user environment.  
    + Installation success can be tested by inputting `tarql --help` in the command line.  

    ```
    choco install openjdk11
    choco install maven  
    git clone http://github.com/tarql/tarql
    cd tarql 
    mvn package appassembler:assemble
    setx PATH "C:\Windows\System32\tarql\target\appassembler\bin" 
    ```

+ A Python virtual environment, ideally managed using `venv` (for managing Python virtual environments) and `poetry` (for managing Python project package repositories). A guide to setting up virtual Python environments for Windows can be found [here](https://realpython.com/python-virtual-environments-a-primer/#how-can-you-work-with-a-python-virtual-environment).   
    
    + Install `pyenv` as administrator for simple Python version management. For Windows, install [`pyenv-win`](https://github.com/pyenv-win/pyenv-win) using `choco install pyenv-win`.   
    + Use `pyenv` to install Python 3.9 by checking which Python versions are available using `pyenv install --list` and then installing a library i.e. using `pyenv install 3.9.6`. Set your global python version i.e. using `pyenv global 3.9.6`.   
    + Navigate to your local `sparql_with_hobbits` repository and set your local python version i.e. using `pyenv local 3.9.6`. Doing so creates a `.python-version` file in your repository to track your project Python version.   
    + Install `poetry` as administrator using `curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -`. This is the recommended installation method and prevents the need to install `poetry` for each Python version you upgrade to and use.    

    ```
    choco install pyenv-win
    pyenv install 3.9.6
    pyenv global 3.9.6
    cd .../sparql_with_hobbits
    pyenv local 3.9.6
    curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -
    ```
    
    + The `poetry` package can be used to manage Python virtual environments and package dependencies. Navigate to your local `sparql_with_hobbits` and initialise `poetry` inside a pre-existing project using `poetry init`. Doing so creates a `pyproject.toml` file in your repository, which `poetry` uses to define your project metadata, dependencies and scripts.  
    + Install `rdflib` to work with RDF files in Python using `poetry add rdflib`. This installs `rdflib` in your poetry-managed Python virtual environment and lists it as a dependency in `pyproject.toml` and a new `poetry.lock` file.    

    ```
    cd .../sparql_with_hobbits
    poetry init
    poetry add rdflib
    ```
    + Use `poetry shell` create and activate your `poetry` managed virtual environment. To run your Python script, you can also use `poetry run python .\scripts\example.py`.  
    + **Note:** Make sure you have the following environment path variables `C:\Windows\System32\tarql\target\appassembler\bin`, `%USERPROFILE%\.pyenv\pyenv-win\bin`, `%USERPROFILE%\.pyenv\pyenv-win\shims` and `%USERPROFILE%\.poetry\bin` to access `tarql` and `python` from the command line.  

+ The Visual Studio [`Stardog RDF Grammars`](https://marketplace.visualstudio.com/items?itemName=stardog-union.stardog-rdf-grammars) IDE support extension, which is also hosted on [GitHub](https://github.com/stardog-union/stardog-vsc/tree/master/stardog-rdf-grammars).  
+ A [Stardog cloud](https://www.stardog.com/stardog-cloud/) free tier account, which provides alternative access to a GUI interface for running SPARQL queries.  

Where possible, we will work in a code editor or directly in the command line to develop reproducible coding practices.   


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
2. Write a simple SPARQL query  


# Package versions  
+ Chocolatey v1.1.0
+ Python 3.9.6
+ Poetry 1.1.13 
+ Access [`./poetry.lock`](https://github.com/erikaduan/sparql_with_hobbits/blob/main/poetry.lock) for Python package versions   


# Acknowledgements  
+ [Learning SPAQRL Version 2](http://www.learningsparql.com/) by Bob DuCharme.  
+ [Official website](https://tarql.github.io/) containing documentation and tutorials for using the `tarql` package.   
+ [YouTube series](https://www.youtube.com/watch?v=Q5DrZV5wWzo&list=PLoOmvuyo5UAeihlKcWpzVzB51rr014TwD) on semantic web technologies created by OpenHPI Tutorials.   
+ [Analysis of Australia's food supply chain](https://www.awe.gov.au/agriculture-land/farm-food-drought/food/publications/foodmap-a-comparative-analysis) conducted by the Department of Agriculture, Water and Energy (2012).   
+ [Website](https://www.ranker.com/list/hobbit-names/book-keeper) containing list of Hobbit names.   
+ [Blog post](https://blog.jayway.com/2019/12/28/pyenv-poetry-saviours-in-the-python-chaos/) on managing Python versions and packages for projects.