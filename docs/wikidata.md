

Information disponible dans Wikidata concernant la population étudiée.


## Documentation concernant les requêtes SPARQL effectuées dans Wikidata

* SPARQL Endpoint: https://query.wikidata.org
* Query builder: https://query.wikidata.org/querybuilder/?uselang=en
* [SPARQL query service/queries](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/queries): documentation.
* [Structure of statements in Wikidata](https://www.wikidata.org/wiki/Help:Statements)


## Inspection des notices

On choisit quelques personnes et on inspecte leurs notices dans Wikidata afin d'observer quelles propriétés permettent de retrouver la population.

Par exemple:

* [Johannes Kepler](https://m.wikidata.org/wiki/Q8963)
* [Werner Heisenberg](https://m.wikidata.org/wiki/Q40904)


On retient quelques propriétés qui permettent de retrouver toute la population:
* [occupation](https://m.wikidata.org/wiki/Property:P106)
* [field of work](https://m.wikidata.org/wiki/Property:P101)



## On effectue des requêtes pour vérifier quels effectifs sont disponibles et de qui il s'agit


### Effectifs concernant 'occupation' et/ou 'field of work'

        SELECT (COUNT(*) as ?eff)
        WHERE {
            ?item wdt:P31 wd:Q5;  # Any instance of a human.
            wdt:P106 wd:Q11063  # astronomer 10162
            
            # wdt:P101 wd:Q333  # astronomy 2161
            # wdt:P106 wd:Q169470 # physicist 32123
            #  wdt:P101 wd:Q413 # physics ~ 4000
            #  wdt:P106 wd:Q155647  # astrologer 1364
            #  wdt:P101 wd:Q34362 # astrology 241
            #  wdt:P106 wd:Q170790  # mathematician 39562
            #  wdt:P106 wd:Q901 # scientist 36117

        }  


### Combiner 'occupation' avec 'field of work'

12323 le 14 mars 2024


        SELECT (COUNT(*) as ?eff)
        WHERE {
            ?item wdt:P31 wd:Q5;  # Any instance of a human.
            {?item wdt:P106 wd:Q11063}
            UNION
            {?item wdt:P101 wd:Q333}            
        }  
        
### Les individus

    SELECT ?item ?itemLabel ?dateBirth
    WHERE {
        {
          {?item wdt:P106 wd:Q11063}
          UNION
          {?item wdt:P101 wd:Q333}
        }  
        ?item wdt:P31 wd:Q5;  # Any instance of a human.
              wdt:P569 ?dateBirth.
      
        SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
        }  
    LIMIT 100

### Les individus en filtrant sur l'année de naissance

Par exemple ici les astronomes du 18e siècle 

    SELECT DISTINCT ?item ?itemLabel ?year ?birthDate
    WHERE {
        {
          {?item wdt:P106 wd:Q11063}
          UNION
          {?item wdt:P101 wd:Q333}
        }  
        ?item wdt:P31 wd:Q5;  # Any instance of a human.
              wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1700 && xsd:integer(?year) < 1801)

        SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
        }  
    ORDER BY ?year 

NB: il peut y avoir des doublons si les dates de naissance sont multiples. La clause DISTINCT permet d'enlever les doublons il faut toutefois enlever la variable *?birthDate* de la sortie et laisser seulement l'année




## Lister les propriétés disponibles avec effectifs

### Sortantes

Cf. [sur cette page](./Wikidata-liste-proprietes-population.md) les listes de propiétés qui résultent de cette requête

    SELECT ?p ?propLabel ?eff
    WHERE {
    {
    SELECT ?p  (count(*) as ?eff)
    WHERE {
        {?item wdt:P106 wd:Q11063}
        UNION
        {?item wdt:P101 wd:Q333}    
        ?item wdt:P31 wd:Q5; # Any instance of a human.
                wdt:P569 ?birthDate.
        ?item  ?p ?o.

        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1700 && xsd:integer(?year) < 1801)
        }
    GROUP BY ?p 
    
        }
    ?prop wikibase:directClaim ?p .

    SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 

    
    }  
    ORDER BY DESC(?eff)


NB Noter qu'il peut y avoir des problèmes de time-out, la requête est trop longue et on a un message d'erreur.
<br/>
Dans ce cas il faut restreindre la période ou limiter le nombre de clauses UNION et décomposer la requête en différentes parties.

On exporte ensuite cette liste sous forme d'une _table HTML_ afin de documenter la suite des opérations. On ouvre la page HTML avec VS Code, on peut mettre en forme avec la commande (click droit) _format document_, puis on copie seulement la partie 'table' depuis la balise &lt;table&gt; jusqu'à &lt;/table&gt;, balises comprises, et on la colle dans un nouveau document Markdown, cf. [Wikidata-liste-proprietes-population.md](Wikidata-liste-proprietes-population.md)


On pourra prendre des notes concernant les opérations effectuées sur les différentes propriétés directement dans ce document et documenter ainsi les choix effectués.



### Entrantes

    SELECT ?p ?propLabel ?eff ('   ' as ?notes)
    WHERE {
    {
    select ?p  (count(*) as ?eff)
    where {
        {?item wdt:P106 wd:Q11063}
            UNION
            {?item wdt:P101 wd:Q333}    
        ?item wdt:P31 wd:Q5; # Any instance of a human.
                wdt:P569 ?birthDate.
        ?s  ?p ?item.

            BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
            FILTER(xsd:integer(?year) > 1700 && xsd:integer(?year) < 1801)
        }
    GROUP BY ?p 
    
        }
    ?prop wikibase:directClaim ?p .

    SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } 

    
    }  
    ORDER BY DESC(?eff)



## Exemple de requête concernant les appartenances à une organisation, avec dates optionnelles si connues


On doit dans cette requête sortir du cadre classique de la simple propriété 'member of' et passer à travers l'assertion, le *statement*. Un statement de _Wikidata_ apparait en quelques sortes comme une entité temporelle même si elle n'associe que deux entités principales, comme une propriété.


    SELECT DISTINCT ?item ?itemLabel ?birthYear ?statement ?organization ?organizationLabel 
                    ?startYear ?endYear  ?startTime ?endTime
    where {
            
        {?item wdt:P106 wd:Q11063}
                UNION
                {?item wdt:P101 wd:Q333}
            
        ?item wdt:P31 wd:Q5; # Any instance of a human.
                wdt:P569 ?birthDate;
                # member of
                p:P463 ?statement.
            ?statement ps:P463 ?organization.
        OPTIONAL {
                        ?statement pq:P580 ?startTime;
                        pq:P582 ?endTime.
            }
        
        BIND(REPLACE(str(?startTime), "(.*)([0-9]{4})(.*)", "$2") AS ?startYear)
        BIND(REPLACE(str(?endTime), "(.*)([0-9]{4})(.*)", "$2") AS ?endYear)
        
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?birthYear)
        FILTER(xsd:integer(?birthYear) > 1700 && xsd:integer(?birthYear) < 1801)
            
        SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
        }
    ORDER BY ?birthYear ?startYear


### Autre exemple

     SELECT DISTINCT ?item ?itemLabel ?birthYear ?statement ?organization ?organizationLabel 
                    ?startYear ?endYear  ?startTime ?endTime
    where {
            
        {?item wdt:P106 wd:Q11063}
                UNION
                {?item wdt:P101 wd:Q333}
            
        ?item wdt:P31 wd:Q5; # Any instance of a human.
                wdt:P569 ?birthDate;
                # member of
                # p:P463 ?statement.
                # ?statement ps:P463 ?organization.
                # educated at
                p:P69 ?statement.
                ?statement ps:P69 ?organization.
              # employer
                #p:P108 ?statement.
                #?statement ps:P108 ?organization.
      #  OPTIONAL
      {
                        ?statement pq:P580 ?startTime;
                        pq:P582 ?endTime.
            }
        
        BIND(REPLACE(str(?startTime), "(.*)([0-9]{4})(.*)", "$2") AS ?startYear)
        BIND(REPLACE(str(?endTime), "(.*)([0-9]{4})(.*)", "$2") AS ?endYear)
        
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?birthYear)
        FILTER(xsd:integer(?birthYear) > 1800 && xsd:integer(?birthYear) < 1901)
            
        SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
        }
    ORDER BY ?item