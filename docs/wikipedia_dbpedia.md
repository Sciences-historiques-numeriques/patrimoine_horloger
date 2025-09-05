



    <p><a href="https://www.wikipedia.org/">Wikipedia. The Free Encyclopedia</a></p>
    <p>Un moyen de communication et d'information universel et ouvert qui permet d'avoir des informations plus ou moins
        étendues</p>


Écrire dans Wikipedia


## Le point de départ: Wikpedia


### Population d'astronomes

<https://en.wikipedia.org/wiki/List_of_astronomers>

&nbsp;


### Une fiche d'astronome dans Wikipedia (données non-structurées et semi-structurées)

<https://en.wikipedia.org/wiki/Giovanni_Domenico_Cassini>

* Observer l'information que contient le texte
* Observer les propriétés de l'objet présentes dans l'infobox
  * [Infobox](https://en.wikipedia.org/wiki/Infobox)
  * [HelpInfobox](https://en.wikipedia.org/wiki/Help:Infobox)
* Procédure 'analogique': extraire 'manuellement' l'information du texte et de l'infobox et la mettre dans la base de données SQLite

&nbsp;

### Une fiche d'astronome dans DBPedia (données structurées)

<https://dbpedia.org/page/Giovanni_Domenico_Cassini>

Les informations qui figurent sur cette page sont extraites de Wikipedia et produites sous forme de données structurées selon le protocole RDF. En d'autres termes, si la page apparaît comme étant du texte, en réalité elle présente, et rend lisibles, les données structurées du graphe concernant cette personne.

On peut donc les interroger et les récupérer grâce à des requêtes formulées dans le langage SPARQL.

Il s'agit d'abord d'inspecter la page et relever les  informations intéressantes qui sont disponibles. RDF fonctionne sur un modèle "sujet -> prédicat-> objet" qui constitue les triplets.

Le sujet de la page est le sujet de tous les triplets (dans ce cas la personne). Les prédicats sont exprimés sous forme de propriétés (<u>_properties_</u> en anglais) 

Exemples de propriétés:

* dbp:birthDate
  * URI complète de la propriété: <http://dbpedia.org/property/birthDate>
  * préfixe de l'espace de noms: 'dbp' (pour <http://dbpedia.org/property/>)
  * dbp:birthDate est un qualified name, ou QName
  * noter que ces propriétés ont généralement comme cible une valeur (chaîne de caractères ou nombre) même si elles pointent sur un objet
* dbo:influencedBy
  * URI complète de la propriété: https://dbpedia.org/ontology/influenced
  * préfixe de l'espace de noms 'dbo': <http://dbpedia.org/ontology>
  * ce type de proriétés associée généralement un objet à un autre objet (et non à une valeur) et donc en sobjet du triplet on trouve une URI
  * essayez de naviguer d'un _influencer_ vers autre: c'est là qu'on voit des données structurées, toutes définies par des URI
  
&nbsp;

À noter (parmi d'autres entités):

* dbr:List_of_astrologers
* dbr:List_of_astronomers
* [dbr:Astronomer]()
* dbr:Astrology
* dbr:Astronomy


dbr: est le préfixe qui remplace http://dbpedia.org/resource/. En entier la première entrée donne http://dbpedia.org/resource/List_of_astrologers.

dbr:List_of_astrologers est donc un QName

&nbsp;


## DBPedia  

Présentation de DBPedia dans Wikipedia:
<https://en.wikipedia.org/wiki/DBpedia>

Documentation:

* [Linked Data Access](https://www.dbpedia.org/resources/linked-data/). Entre autres: spécification des URI, Content Negotiation
* [SPARQL over Online Databases](https://www.dbpedia.org/resources/sparql/) (documentation)
* __Interfaces__ pour requêtes __SPARQL__:
  * <https://dbpedia.org/sparql>
  * <https://dbpedia.org/snorql/>  (navigateur)  

### DBPedia Live

Modifier Wikipedia et voir les résultats en temps réel

* <https://www.dbpedia.org/resources/live/>
* <https://live.dbpedia.org/sparql>
* NB : Default Data Set Name :  <http://live.dbpedia.org>

### SPARQLIS

* [SPARQLIS sur DBPedia](http://www.irisa.fr/LIS/ferre/sparklis/?title=Core%20English%20DBpedia&endpoint=http%3A//servolis.irisa.fr/dbpedia/sparql)
* [Example queries for Sparklis](http://www.irisa.fr/LIS/ferre/sparklis/examples.html)

SPARQLIS permet de construire des requêtes SPARQL grâce à une interface graphique

&nbsp;

## Requêtes d'exploration

* Interface pour requêtes: __<https://dbpedia.org/sparql>__
* La syntaxe des requêtes: <https://www.w3.org/TR/sparql11-query>
* Tutoriels:
  * [Le tutoriel SPARQL](https://web-semantique.developpez.com/tutoriels/jena/arq/introduction-sparql/) 
  * Tutoriel vidéo: [Partie I](https://www.youtube.com/watch?v=Z8_rTxy67-8) , [Partie II](https://www.youtube.com/watch?v=HlOJSsu3_to)

&nbsp;

### Occupation: astronomer

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    
    SELECT DISTINCT ?thing_1 ?birthDate
    WHERE { ?thing_1 dbo:occupation dbr:Astronomer .
        OPTIONAL { 
                   ?thing_1 dbo:birthYear ?birthDate .
                   }
          }


* Noter que changer les espaces de noms dbp et dbo donne des résultats différents
* Cette requête liste 160 astronomes (3 décembre 2022), le maximum des effectifs avec cette approche

      PREFIX dbr: <http://dbpedia.org/resource/>
      PREFIX dbo: <http://dbpedia.org/ontology/>
      PREFIX dbp: <http://dbpedia.org/property/>
      SELECT (COUNT(*) as ?effectif)
      WHERE { ?thing_1 dbo:occupation dbr:Astronomer .
            }

### Astronomes nés entre 1371 et nous jours / 1770

    PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    PREFIX dbp: <http://dbpedia.org/property/>
    SELECT DISTINCT ?thing_1 ?intYear
    WHERE { ?thing_1 dbo:occupation dbr:Astronomer .
         ?thing_1 dbo:birthYear ?birthYear .
         BIND(xsd:integer(str(?birthYear)) AS ?intYear)
    FILTER ( (?intYear >= 1371
    ###  La clause de filtre ci-dessous est commentée, i.e. non active. Décommenter pour l'activer
    #              && ?intYear < 1771 
                  ) ) }
    ORDER BY ?intYear

Leur effectif (3 décembre 2022): 23 / 124

La requête ci-dessous ne change rien, en espace de noms dbr seulement 13 astronomes

    PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    PREFIX dbp: <http://dbpedia.org/property/>
    SELECT (COUNT(*) as ?effectif)
    WHERE {
      {SELECT DISTINCT ?thing_1
           WHERE {
        { ?thing_1 dbo:occupation dbr:Astronomer }
        UNION
        {?thing_1 dbp:occupation dbr:Astronomer}
              }
        }
     ?thing_1 dbo:birthYear ?birthYear .
     BIND(xsd:integer(str(?birthYear)) AS ?intYear)
    FILTER ( (?intYear >= 1371 
      #        && ?intYear < 1771 
              ) ) }
    ORDER BY ?intYear

&nbsp;

### Liste: dbr:List_of_astronomers

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT DISTINCT ?p ?o1 
    WHERE { 
      dbr:List_of_astronomers ?p ?o1.
      ?o1 a dbo:Person.
      }
    LIMIT 10

### Effectif de la population

Effectifs au 3 décembre 2022 : 762

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT (COUNT(*) as ?eff)
    WHERE { 
    dbr:List_of_astronomers ?p ?o1.
    ?o1 a dbo:Person.
      }

###  Propriétés sortantes et effectifs

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT ?p1 (COUNT(*) as ?eff)
    WHERE { 
    dbr:List_of_astronomers ?p ?o1.
    ?o1 a dbo:Person;
        ?p1 ?o2.
      }
    GROUP BY ?p1
    ORDER BY DESC(?eff)

Noter ces propriétés:

* <http://dbpedia.org/property/birthDate>: 443
* <http://dbpedia.org/ontology/birthDate>: 396

&nbsp;

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbp: <http://dbpedia.org/property/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT (COUNT(*) as ?eff)
    WHERE { 
    dbr:List_of_astronomers ?p ?o1.
    ?o1 a dbo:Person;
      dbp:birthDate ?birthDate.
    BIND(xsd:integer(SUBSTR(STR(?birthDate), 1, 4)) AS ?birthYear)
    FILTER ( (?birthYear >= 1371
    #            && ?birthYear < 1771
     ) ) 
          }

&nbsp;

Documentation: [Property path syntax](https://www.w3.org/TR/sparql11-property-paths/)

Alternative entre propriétés  ( | ):

Effectif: 56


    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbp: <http://dbpedia.org/property/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT DISTINCT ?o1 ?birthDate ?birthYear
    WHERE { 
    dbr:List_of_astronomers ?p ?o1.
    ?o1 a dbo:Person;
      dbp:birthDate | dbo:birthDate ?birthDate.
    BIND(xsd:integer(SUBSTR(STR(?birthDate), 1, 4)) AS ?birthYear)
    FILTER ( (?birthYear >= 1371
      #          && ?birthYear < 1771 
                ) ) 
          }
    ORDER BY ?birthYear

&nbsp;

Astonomes et _astrologues_ et _mathématiciens_:

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbp: <http://dbpedia.org/property/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT DISTINCT ?o1 ?birthDate ?birthYear
    WHERE { 
    {
      {dbr:List_of_astronomers ?p ?o1.}
      UNION
      {dbr:List_of_astrologers ?p ?o1.}
      UNION
            {?o1 ?p dbr:Astrologer.}
      UNION
            {?o1 ?p dbr:Astronomer.}
      UNION
            {?o1 ?p dbr:Mathematician.}

    }
    ?o1 a dbo:Person;
      dbp:birthDate | dbo:birthDate ?birthDate.
    BIND(xsd:integer(SUBSTR(STR(?birthDate), 1, 4)) AS ?birthYear)
    FILTER ( (?birthYear >= 1371
      #          && ?birthYear < 1771 
                ) ) 
          }
    ORDER BY ?birthYear

&nbsp;

Effectif : 292 / 5138 mathématiciens, astrologues, astronomes

__Définition de la population__: cette requêtes définit la population — cette requête sera la base de toutes les autres

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbp: <http://dbpedia.org/property/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT (COUNT(*) as ?effectif)
    WHERE {
      SELECT DISTINCT ?o1 ?birthDate ?birthYear
      WHERE { 
        {
              {dbr:List_of_astronomers ?p ?o1.}
          UNION
              {dbr:List_of_astrologers ?p ?o1.}
          UNION
              {?o1 ?p dbr:Astrologer.}
          UNION
              {?o1 ?p dbr:Astronomer.}
          UNION
              {?o1 ?p dbr:Mathematician.}

        }
        ?o1 a dbo:Person;
          dbp:birthDate | dbo:birthDate ?birthDate.
        BIND(xsd:integer(SUBSTR(STR(?birthDate), 1, 4)) AS ?birthYear)
        FILTER ( (?birthYear >= 1371
           #         && ?birthYear < 1771 
                    
                    ) ) 
              }
      }

        

&nbsp;
Avec les labels des noms (en anglais), même effectif:

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbp: <http://dbpedia.org/property/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT (COUNT(*) as ?effectif)
    WHERE {
      SELECT DISTINCT ?o1 ?birthDate ?birthYear ?label
      WHERE { 
        {
              {dbr:List_of_astronomers ?p ?o1.}
          UNION
              {dbr:List_of_astrologers ?p ?o1.}
          UNION
              {?o1 ?p dbr:Astrologer.}
          UNION
              {?o1 ?p dbr:Astronomer.}
          UNION
              {?o1 ?p dbr:Mathematician.}
          UNION
              {?o1 ?p dbr:Physicist.}

        }
        ?o1 a dbo:Person;
          dbp:birthDate | dbo:birthDate ?birthDate;
          rdfs:label ?label.
        BIND(xsd:integer(SUBSTR(STR(?birthDate), 1, 4)) AS ?birthYear)
        FILTER ( (?birthYear >= 1371
              #      && ?birthYear < 1771 
                  )
                    && LANG(?label) = 'en') 
              }
      }


&nbsp;

## Les propriétés

### Liste des propriétés sortantes

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT ?p1 (COUNT(*) as ?eff)
    WHERE { 
        {
              {dbr:List_of_astronomers ?p ?o1.}
          UNION
              {dbr:List_of_astrologers ?p ?o1.}
          UNION
              {?o1 ?p dbr:Astrologer.}
          UNION
              {?o1 ?p dbr:Astronomer.}
          UNION
              {?o1 ?p dbr:Mathematician.}
          UNION
              {?o1 ?p dbr:Physicist.}    
        }
    ?o1 a dbo:Person;
    dbp:birthDate | dbo:birthDate ?birthDate;
        ?p1 ?o2.
    BIND(xsd:integer(SUBSTR(STR(?birthDate), 1, 4)) AS ?birthYear)
    FILTER ( (?birthYear >= 1371  )) 
      }
    GROUP BY ?p1
    ORDER BY DESC(?eff)

### Liste des propriétés entrantes

    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT ?p1 (COUNT(*) as ?eff)
    WHERE { 
        {
              {dbr:List_of_astronomers ?p ?o1.}
          UNION
              {dbr:List_of_astrologers ?p ?o1.}
          UNION
              {?o1 ?p dbr:Astrologer.}
          UNION
              {?o1 ?p dbr:Astronomer.}
          UNION
              {?o1 ?p dbr:Mathematician.}
          UNION
              {?o1 ?p dbr:Physicist.}
        }
    ?o1 a dbo:Person;
    dbp:birthDate | dbo:birthDate ?birthDate.
    ?o2 ?p1 ?o1.
    BIND(xsd:integer(SUBSTR(STR(?birthDate), 1, 4)) AS ?birthYear)
    FILTER ( (?birthYear >= 1371  )) 
      }
    GROUP BY ?p1
    ORDER BY DESC(?eff)

&nbsp;



### Liste des astronomes—astrologues

Cette requête produit une liste de personnes.

Si on veut importer les données dans une base de données SQLite, suivre les [instructions indiquées sur ces pages](Importer_DBpedia_base_personnelle.md): 



    PREFIX dbr: <http://dbpedia.org/resource/>
    PREFIX dbp: <http://dbpedia.org/property/>
    PREFIX dbo: <http://dbpedia.org/ontology/>
    SELECT DISTINCT ?o1  (str(?label) as ?name) ?birthYear
    WHERE {
    SELECT DISTINCT ?o1 ?birthDate ?birthYear ?label
    WHERE { 
      {
            {dbr:List_of_astronomers ?p ?o1.}
        UNION
            {dbr:List_of_astrologers ?p ?o1.}
        UNION
            {?o1 ?p dbr:Astrologer.}
        UNION
            {?o1 ?p dbr:Astronomer.}
        UNION
            {?o1 ?p dbr:Mathematician.}
        UNION
            {?o1 ?p dbr:Physicist.}

      }
      ?o1 a dbo:Person;
        dbp:birthDate | dbo:birthDate ?birthDate;
        rdfs:label ?label.
      BIND(xsd:integer(SUBSTR(STR(?birthDate), 1, 4)) AS ?birthYear)
      FILTER ( (?birthYear >= 1371
      #            && ?birthYear < 1771 
                  )
                  && LANG(?label) = 'en') 
            }
    }
    ORDER BY ?birthYear

  
