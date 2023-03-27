# Rome: Corso

# UPDATE PLEAE LOOK HERE:

https://github.com/biblhertz/Corso

## Step 1

Querying OSM for polygons.

OSM Polygons, eg. https://www.openstreetmap.org/way/80220845

Query per getting OSM polygons from https://overpass-turbo.eu/#:

```sql
[out:json][timeout:25];
(
  way["building"](41.89336481112412,12.47446060180664,41.912497421968425,12.4859619140625);
  relation["building"](41.89336481112412,12.47446060180664,41.912497421968425,12.4859619140625);
);
out body;
>;
>out skel qt;
```

## Step 2

Querying Wikidata for data.

Wikidata, eg. https://www.wikidata.org/wiki/Q54863408

Query for getting WikiData from https://query.wikidata.org/#:

```sparql
SELECT DISTINCT ?place ?place_short ?placeLabel ?location ?osm ?gnd ?viaf ?worldcat ?topostext ?pleiades ?oclc ?getty ?dare ?arachne ?archinform ?mibact ?iccd ?iccdcf ?geonames ?inception ?wikimedia ?wikipedia #?instance ?instanceLabel
WHERE {
  SERVICE wikibase:box {
      ?place wdt:P625 ?location.    
      bd:serviceParam wikibase:cornerWest "Point(12.47446060180664 41.89336481112412)"^^geo:wktLiteral.
      bd:serviceParam wikibase:cornerEast "Point(12.4859619140625 41.912497421968425)"^^geo:wktLiteral.
    }
#  OPTIONAL {    ?place wdt:P31/wdt:P279* ?instance  }
  OPTIONAL {    ?place wdt:P571 ?inception  }
  OPTIONAL {    ?place wdt:P402 ?osm  }
  OPTIONAL {    ?place wdt:P227 ?gnd  }
  OPTIONAL {    ?place wdt:P214 ?viaf  }          
  OPTIONAL {    ?place wdt:P7859 ?worldcat }    
  OPTIONAL {    ?place wdt:P243 ?oclc } 
  OPTIONAL {    ?place wdt:P1667 ?getty } 
  OPTIONAL {    ?place wdt:P1936 ?dare }
  OPTIONAL {    ?place wdt:P6787 ?arachne }
  OPTIONAL {    ?place wdt:P5383 ?archinform }
  OPTIONAL {    ?place wdt:P5782 ?mibact  }          
  OPTIONAL {    ?place wdt:P6286 ?iccd  }   
  OPTIONAL {    ?place wdt:P6287 ?iccdcf  }   
  OPTIONAL {    ?place wdt:P1566 ?geonames  }       
  OPTIONAL {    ?place wdt:P1584 ?pleiades  }       
  OPTIONAL {    ?place wdt:P8068 ?topostext  }       
  OPTIONAL {    ?place wdt:P571 ?inception  }
  OPTIONAL {    ?place wdt:P18 ?image  }
  OPTIONAL {    ?place wdt:P373 ?wikimedia  }
  OPTIONAL {    ?wikipedia schema:about ?place ; schema:isPartOf <https://it.wikipedia.org/>	}
  VALUES ?instance { wd:Q24398318 wd:Q35112127 wd:Q16560 wd:Q16970 wd:Q160742 wd:Q1128397 wd:Q12518 wd:Q120560 wd:Q1649060 wd:Q580499 wd:Q19899465 wd:Q44613 wd:Q82117 wd:Q35112127 } .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "it". }
  BIND( (REPLACE(STR(?place),"http://www.wikidata.org/entity/","")) AS ?place_short )
} ORDER BY ?distance LIMIT 1000
```

## Step 3

Consolidating data: we now see what and how much data is still lacking in OSM and WData.

Obviously, we need a user account for Wikidata and for OSM.

The lacking/incomplete data has to be corrected in Wikidata and OSM, **not** on the local databases. A re-import will then be started (1-2 minutes).

#### In OSM

In OSM, input/add all the right data, eg. for Palazzo Bonaparte: https://www.openstreetmap.org/relation/1897922

Data inherent for the map would be:

- name, other names, language variants
- wikidata id
- building:levels (alternatively, the absolute height)

Plus, maybe a polygon has to be corrected, split or added.

#### In Wikidata

In Wikidata too, see that the right data is set, eg. for https://www.wikidata.org/wiki/Q3889693

Data like:

- country, administrative territorial entity
- name, other names, language variants
- architect, engineer, architectural style, inception
- IDs: GND, VIAF, ICCD, Worldcat, ArchInform, &c.

## Step 4

Mix the above data into one (1) GeoJSON.

Here we only get buildings with Wikidata Ids in OSM: 

```sql
SELECT DISTINCT *
FROM 'osm-buildings'
LEFT JOIN 'wd-query' on place_short=wikidata
WHERE name not null and wikidata not null 
```
Load the result in QGIS as QueryLayer:Wikidata-ID

Buildings with no Wikidata ID (maybe they are regarded as unimportant) ... well, stay as they are for now.

## Follow Ups

Add more materiel into a new table (in QGIS, connecting via wdata/osm-relation-id) from eg. following sites:

- https://dati.beniculturali.it/applicazioni_/applicazione-fotografico/?FFC031013
- https://www.romasparita.eu/foto-roma-sparita/?s=via+del+corso
&c.
