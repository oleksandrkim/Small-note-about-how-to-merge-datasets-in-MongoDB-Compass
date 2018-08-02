# Analyze cinema industry with MongoDB and Python
Small code that explains how to merge 3 datasets into one with MongoDb Compass and save it as a separate table in a database. Datasets are taken from Unesco, World Bank and  Numbeo. Files are available at the repository.<br>

The idea is to analyze the film industry a little bit, compare the state of cinema industry across countries all over the world and analyze the correlation between variables. Hypotheses are:<br /><br />



- Hypothesis 1
1. H1: there is a relationship between population grows and ticket sold
2. H0: there is no relationship between population grows and ticket sold.<br /><br />



- Hypothesis 2
1. H1: there is a positive relationship between GDP and number of cinemas per country
2. H0: there is no relationship between GDP and number of cinemas per country <br /><br />



- Hypothesis 3
1. H1: People who live in developed country buy more tickets in cinema
2. H0: People who live in developed country do not buy more tickets in cinema <br /><br />

## MongoDB

**Start a MongoDB shell**

[Reference on StackOverflow](https://stackoverflow.com/questions/42739166/could-not-connect-to-mongodb-on-the-provided-host-and-port?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

1. Create folder "C:\data\db"
2. access "mongod" in bin folder of MongoDB to set up the MongoDB on a machine and establish connection
3. access "mongo" in bin folder of MongoDB to start shell

**Create a database**

`use project`

>switched to db project

**Add collections (tables) to a database**

```
db.createCollection("unesco")
db.createCollection("numbeo")
db.createCollection("worldbank")
```

>{ "ok" : 1 }

`show collections`

>unesco  
>numbeo  
>worldbank  

**Fill collections with data. Data is stored in csv (through command line)**

```
mongoimport --db georgian --collection unesco --type csv --headerline --file “D:/mongodb/DatasetsFilm/unesco.csv"  
mongoimport --db georgian --collection numbeo --type csv --headerline --file “D:/mongodb/DatasetsFilm/tickets.csv"  
mongoimport --db georgian --collection worldbank --type csv --headerline --file “D:/mongodb/DatasetsFilm/worldbank.csv"  
```

**First merge (unesco with worldbank)**

Mutual field is "country"

```db.createCollection("merged_one")
db.worldbank.aggregate([{$lookup: {from: "unesco",localField: "country",foreignField: "country",as: "cinemas"}},{$out : "merged_one"}])
```

**Second merge (first merge with worldbank)**

Mutual field is "country"

```db.createCollection("merged_two")
db.merged_one.aggregate([{$lookup: {from: "numbeo",localField: "country",foreignField: "country",as: "ticket_price"}},{$out : "merged_two"}])
```

**Export collection as json**

```
mongoexport --db georgian --collection merged_two --out “C:/Users/alexa/Desktop/GC/GC2/Data collection and curation/mongodb/merged_t.json"
```

## Python code and data analysis

**Create dataframe by extracing data from json**

```
import pandas as pd
import json
countries=[]
admissions=[]
ticket_prices=[]
gdp_s=[]
gdp_per_capita=[]
population=[]
cinemas=[]
for line in open("C:/Users/alexa/Desktop/GC/GC2/Data collection and curation/mongodb/merged_t.json", 'r'):
    #lines.append(line)
    js = json.loads(line)
    countries.append(js['country'])
    gdp_s.append(js['gdp'])
    gdp_per_capita.append(js['gdp_capita'])
    population.append(js['population'])
    admissions.append(js['cinemas'][0]['admiss'])
    cinemas.append(js['cinemas'][0]['cinemas'])
    ticket_prices.append(js['ticket_price'][0]['ticket_price'])
    
dict={}

dict["countries"] = countries
dict["admissions"] = admissions
dict["ticket_prices"] = ticket_prices
dict["gdp_s"] = gdp_s
dict["gdp_per_capita"] = gdp_per_capita
dict["population"] = population
dict["cinemas"] = cinemas

df = pd.DataFrame(data=dict)
```

![Alt text](D:/viburnum/mongodb_cinemas/Screenshot_25.png?raw=true "Optional Title")

>countries	admissions	ticket_prices	gdp_s	gdp_per_capita	population	cinemas  
>Austria	15922451.0	12.32	382065930307.98	36117.41	8633169.0	138.0  
>Azerbaijan	559391.0	4.11	53074370486.04	2195.31	9649341.0	10.0  
