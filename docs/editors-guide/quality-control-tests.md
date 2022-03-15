# Custom SPARQL checks Mondo

## Mondo specific checks

###  qc-equivalent-preferred-multiple.sparql

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE {
    ?entity rdfs:subClassOf+ <http://purl.obolibrary.org/obo/MONDO_0000001> .
  	?entity oboInOwl:hasDbXref ?xref .
  	?entity oboInOwl:hasDbXref ?xref2 .
        
    ?xref_anno a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty oboInOwl:hasDbXref ;
           owl:annotatedTarget ?xref ;
           oboInOwl:source ?source0 ;
           oboInOwl:source ?source1 .
  
  	?xref_anno2 a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty oboInOwl:hasDbXref ;
           owl:annotatedTarget ?xref2 ;
           oboInOwl:source ?source2 ;
           oboInOwl:source ?source4 .
           

    FILTER(str(?xref2)!=str(?xref))
  	FILTER(STRBEFORE(str(?xref),":") = STRBEFORE(str(?xref2),":")) 
    FILTER ((str(?source0)="MONDO:preferredExternal"))
    FILTER ((str(?source4)="MONDO:preferredExternal"))
    FILTER ((str(?source1)="MONDO:equivalentTo") || (str(?source1)="MONDO:equivalentObsolete"))
  	FILTER ((str(?source2)="MONDO:equivalentTo") || (str(?source2)="MONDO:equivalentObsolete"))
    FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
    BIND(?xref as ?property)
    BIND(?xref2 as ?value)
}

```

###  qc-equivalent-to-on-deprecated.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

#description: Deprecated class employs equivalentTo

SELECT DISTINCT ?entity ?property ?value WHERE {
    ?entity oboInOwl:hasDbXref ?xref .
        
    ?xref_anno a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty oboInOwl:hasDbXref ;
           owl:annotatedTarget ?xref ;
           oboInOwl:source ?source .
    
    ?entity owl:deprecated true 

    FILTER ((str(?source)="MONDO:equivalentTo") || (str(?source)="MONDO:equivalentObsolete"))
    FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
    BIND(?xref as ?property)
    BIND(?source as ?value)
}
ORDER BY ?entity

```

###  qc-excluded-subsumption-is-inferred.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix mondo: <http://purl.obolibrary.org/obo/mondo#>
prefix mondoSparqlQcGeneral: <http://purl.obolibrary.org/obo/mondo/sparql/qc/general/>
prefix mondoSparqlQcMondo: <http://purl.obolibrary.org/obo/mondo/sparql/qc/mondo/>

SELECT DISTINCT ?entity ?property ?value  
WHERE 
{ 
  ?entity mondo:excluded_subClassOf ?parent ;
      rdfs:subClassOf* ?parent .
  
    FILTER NOT EXISTS {
       ?entity mondo:excluded_from_qc_check mondoSparqlQcMondo:qc-excluded-subsumption-is-inferred.sparql .
    }
  
  FILTER (isIRI(?parent) && STRSTARTS(str(?parent), "http://purl.obolibrary.org/obo/MONDO_"))
  FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
  FILTER( ?entity=?parent)
  BIND(mondo:excluded_subClassOf as ?property)
  BIND(str(?parent) as ?value)
}
ORDER BY ?entity
```

###  qc-no-subclass-between-genetic-disease.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix mondo: <http://purl.obolibrary.org/obo/mondo#>
prefix mondoSparqlQcGeneral: <http://purl.obolibrary.org/obo/mondo/sparql/qc/general/>
prefix mondoSparqlQcMondo: <http://purl.obolibrary.org/obo/mondo/sparql/qc/mondo/>

SELECT DISTINCT ?entity ?property ?value WHERE 
{

?entity rdfs:subClassOf [ rdf:type owl:Restriction ;
owl:onProperty <http://purl.obolibrary.org/obo/RO_0004020> ;
owl:someValuesFrom ?gene1 ] . 
?entity rdfs:label ?entity_label .

?entity rdfs:subClassOf* ?value .

?value rdfs:subClassOf [ rdf:type owl:Restriction ;
owl:onProperty <http://purl.obolibrary.org/obo/RO_0004020> ;
owl:someValuesFrom ?gene2 ] .

FILTER NOT EXISTS { ?value rdfs:subClassOf [ rdf:type owl:Restriction ;
owl:onProperty <http://purl.obolibrary.org/obo/RO_0004020> ;
owl:someValuesFrom ?gene1 ] . }

FILTER NOT EXISTS {
   ?entity mondo:excluded_from_qc_check mondoSparqlQcMondo:qc-no-subclass-between-genetic-disease.sparql .
}

FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
FILTER (isIRI(?value) && STRSTARTS(str(?value), "http://purl.obolibrary.org/obo/MONDO_"))
FILTER (isIRI(?gene1) && STRSTARTS(str(?gene1), "http://identifiers.org/hgnc/"))
FILTER (isIRI(?gene2) && STRSTARTS(str(?gene2), "http://identifiers.org/hgnc/"))
FILTER ( ?entity!=?value )
FILTER ( ?gene1!=?gene2 )
BIND(rdfs:subClassOf as ?property)
}
ORDER BY ?entity
# FILTER ( ?gene1!=?gene2 ) this is a hack to circumvent the case that a disease pertains to the same gene but is further specified.

# Using the subclass selector turned out way too costly computationally:
# ?gene rdfs:subClassOf* <http://purl.obolibrary.org/obo/SO_0001217> .
# ?gene2 rdfs:subClassOf* <http://purl.obolibrary.org/obo/SO_0001217> .
```

###  qc-no-superclass.sparql

```
prefix xsd: <http://www.w3.org/2001/XMLSchema#>
prefix oio: <http://www.geneontology.org/formats/oboInOwl#>
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix disease: <http://purl.obolibrary.org/obo/MONDO_0000001>
prefix disease_characteristic: <http://purl.obolibrary.org/obo/MONDO_0021125>
prefix disease_stage: <http://purl.obolibrary.org/obo/MONDO_0021007>
prefix disease_susceptibility: <http://purl.obolibrary.org/obo/MONDO_0042489>
prefix xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?entity ?property ?value WHERE 
{
  ?entity a owl:Class ;
     rdfs:label ?n
  FILTER (strstarts(str(?entity),"http://purl.obolibrary.org/obo/MONDO_"))
  FILTER (!isBlank(?entity))
  FILTER ( NOT EXISTS {?entity owl:deprecated "true"^^xsd:boolean} )
  FILTER ( NOT EXISTS
    {
      {?entity rdfs:subClassOf* disease: }
        UNION
      {?entity rdfs:subClassOf* disease_characteristic: }
        UNION
      {?entity rdfs:subClassOf* disease_susceptibility: }
        UNION
      {?entity rdfs:subClassOf* disease_stage: }
    } )
    BIND(rdfs:subClassOf as ?property)
}
ORDER BY ?entity
```

###  qc-obsolete-equivalent-on-non-deprecated.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE {
    ?entity oboInOwl:hasDbXref ?xref .
        
    ?xref_anno a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty oboInOwl:hasDbXref ;
           owl:annotatedTarget ?xref ;
           oboInOwl:source ?source .
    
    FILTER NOT EXISTS { ?entity owl:deprecated true }

    FILTER ((str(?source)="MONDO:obsoleteEquivalent") || (str(?source)="MONDO:obsoleteEquivalentObsolete"))
    FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
    BIND(?xref as ?property)
    BIND(str(?entity2) as ?value)
}
ORDER BY ?entity

```

###  qc-omim-subsumption.sparql

```
prefix omim: <http://identifiers.org/omim/>
prefix skos: <http://www.w3.org/2004/02/skos/core#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix xsd: <http://www.w3.org/2001/XMLSchema#>
prefix mondo: <http://purl.obolibrary.org/obo/mondo#>
prefix mondoSparqlQcGeneral: <http://purl.obolibrary.org/obo/mondo/sparql/qc/general/>
prefix mondoSparqlQcMondo: <http://purl.obolibrary.org/obo/mondo/sparql/qc/mondo/>
prefix mondoPatterns: <http://purl.obolibrary.org/obo/mondo/patterns/>

SELECT DISTINCT ?entity ?property ?value WHERE {
   ?entity rdfs:subClassOf ?parent .
   ?exp owl:annotatedSource ?entity ;
        owl:annotatedProperty oboInOwl:hasDbXref ;
        owl:annotatedTarget ?xref;
        oboInOwl:source ?source .
   ?exp_p owl:annotatedSource ?parent ;
       owl:annotatedProperty oboInOwl:hasDbXref ;
       owl:annotatedTarget ?xref_p;
       oboInOwl:source ?source_p .
   ?entity rdfs:label ?entity_label .
   
   FILTER NOT EXISTS {
      ?entity mondo:excluded_from_qc_check mondoSparqlQcMondo:qc-omim-subsumption.sparql .
   }
   
   FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
   FILTER (isIRI(?parent) && STRSTARTS(str(?parent), "http://purl.obolibrary.org/obo/MONDO_"))
   FILTER(STRSTARTS(str(?xref), "OMIM:"))
   FILTER(STRSTARTS(str(?xref_p), "OMIM:"))
   FILTER(str(?source)="MONDO:equivalentTo")
   FILTER(str(?source_p)="MONDO:equivalentTo")
   BIND(rdfs:subClassOf as ?property)
   BIND(REPLACE(STR(?parent),"[<>]","") AS ?value)
}
ORDER BY ?entity
```

###  qc-omimps-should-be-inherited.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix xsd: <http://www.w3.org/2001/XMLSchema#>
prefix mondo: <http://purl.obolibrary.org/obo/mondo#>
prefix mondoSparql: <http://purl.obolibrary.org/obo/mondo/sparql/>
prefix mondoPatterns: <http://purl.obolibrary.org/obo/mondo/patterns/>

SELECT DISTINCT ?entity ?property ?value WHERE {
  ?exp owl:annotatedSource ?entity ;
       owl:annotatedProperty oboInOwl:hasDbXref ;
       owl:annotatedTarget ?xref;
       oboInOwl:source ?source .

  FILTER NOT EXISTS {
    ?entity rdfs:subClassOf [ rdf:type owl:Restriction ;
    owl:onProperty <http://purl.obolibrary.org/obo/RO_0002573> ;
    owl:someValuesFrom <http://purl.obolibrary.org/obo/MONDO_0021152> ] .
  }
  
  FILTER NOT EXISTS { ?entity owl:deprecated "true"^^xsd:boolean . }
  
  FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
  FILTER(STRSTARTS(str(?xref), "OMIMPS:"))
  FILTER(str(?source)="MONDO:equivalentTo")
  BIND(<http://purl.obolibrary.org/obo/RO_0002573> as ?property)
}
ORDER BY ?entity
```

###  qc-predispose-subClassOf.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix mondo: <http://purl.obolibrary.org/obo/mondo#>

SELECT DISTINCT ?entity ?property ?value WHERE {
  ?entity owl:equivalentClass ?intersection .
  ?intersection owl:intersectionOf ?lst .
  ?lst rdf:rest*/rdf:first ?restriction .
  ?restriction owl:someValuesFrom ?value .
  ?restriction owl:onProperty mondo:predisposes_towards .
  ?entity rdfs:subClassOf ?value . 
  FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
  BIND(mondo:predisposes_towards as ?property)
}
ORDER BY ?entity
```

###  qc-proxy-merge-equiv.sparql

```
prefix oio: <http://www.geneontology.org/formats/oboInOwl#>
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE {
  ?entity owl:equivalentClass ?x .
  ?c2 owl:equivalentClass ?x .

  FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
  FILTER (isIRI(?c2) && STRSTARTS(str(?c2), "http://purl.obolibrary.org/obo/MONDO_"))
  FILTER(?entity != ?c2)
  BIND(owl:equivalentClass as ?property)
}
ORDER BY ?entity
```

###  qc-proxy-merges.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

#description: No two Mondo IDs should ever point to the same external ID

SELECT DISTINCT ?entity ?property ?value WHERE {
    ?entity oboInOwl:hasDbXref ?xref .
    
    ?entity2 oboInOwl:hasDbXref ?xref .
    
    ?xref_anno2 a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty oboInOwl:hasDbXref ;
           owl:annotatedTarget ?xref ;
           oboInOwl:source ?source1 .
    	
  		?xref_anno a owl:Axiom ;
           owl:annotatedSource ?entity2 ;
           owl:annotatedProperty oboInOwl:hasDbXref ;
           owl:annotatedTarget ?xref ;
           oboInOwl:source ?source2 .

  	FILTER (?entity2!=?entity)
    FILTER ((str(?source1)="MONDO:equivalentTo") || (str(?source1)="MONDO:obsoleteEquivalent") || (str(?source1)="MONDO:equivalentObsolete") || (str(?source1)="MONDO:obsoleteEquivalentObsolete"))
  	FILTER ((str(?source2)="MONDO:equivalentTo") || (str(?source2)="MONDO:obsoleteEquivalent") || (str(?source2)="MONDO:equivalentObsolete") || (str(?source2)="MONDO:obsoleteEquivalentObsolete"))
    FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
    FILTER (isIRI(?entity2) && STRSTARTS(str(?entity2), "http://purl.obolibrary.org/obo/MONDO_"))
    BIND(?xref as ?property)
    BIND(str(?entity2) as ?value)
}
ORDER BY ?entity

```

###  qc-related-exact-synonym-omim.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE {
  { 
    ?entity oboInOwl:hasRelatedSynonym ?related ;
      rdfs:label ?entity_label ;
      oboInOwl:hasExactSynonym ?exact ;
      a owl:Class .
      [
         a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty oboInOwl:hasExactSynonym ;
           owl:annotatedTarget ?exact ;
           oboInOwl:hasDbXref ?xref1 
      ] .
      [
         a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty oboInOwl:hasRelatedSynonym ;
           owl:annotatedTarget ?related ;
           oboInOwl:hasDbXref ?xref2 
      ] .
    
    FILTER (str(?related)=str(?exact))
    FILTER (regex(str(?xref1), "OMIM^"))
    FILTER (regex(str(?xref2), "OMIM^"))
    FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
    BIND(oboInOwl:hasExactSynonym as ?property)
  }
}

ORDER BY ?entity

```

## General quality checks

###  qc-def-lacks-xref.sparql

```
prefix oio: <http://www.geneontology.org/formats/oboInOwl#>
prefix def: <http://purl.obolibrary.org/obo/IAO_0000115>
prefix owl: <http://www.w3.org/2002/07/owl#>

SELECT ?entity ?property ?value WHERE 
{
  ?entity def: ?value .
  
  ?def_anno a owl:Axiom ;
  owl:annotatedSource ?entity ;
  owl:annotatedProperty def: ;
  owl:annotatedTarget ?value .
  
  FILTER NOT EXISTS {
    ?def_anno oio:hasDbXref ?x .
  }
  
  FILTER (!isBlank(?entity))
  FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
  BIND(def: as ?property)
}
ORDER BY ?entity

```

###  qc-definition-containing-underscore.sparql

```
# description: Checks wether definitions contain underscore characters, which could be an indication of a typo.

prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix IAO: <http://purl.obolibrary.org/obo/IAO_>

SELECT DISTINCT ?entity ?property ?value WHERE 
{
  VALUES ?property {
    IAO:0000115
  }
  ?entity ?property ?value .
  FILTER( regex(STR(?value), "_"))
  FILTER (!isBlank(?entity))
}
ORDER BY ?entity

```

###  qc-deprecated-class-reference.sparql

```
# # Deprecated Class Reference
#
# **Problem:** A deprecated class is used in a logical axiom. A deprecated class can be the child of another class (e.g. ObsoleteClass), but it cannot have children or be used in blank nodes or equivalent class statements. Additionally, a deprecated class should not have any equivalent classes or anonymous parents.
#
# **Solution:** Replace deprecated class.

PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT DISTINCT ?entity ?property ?value WHERE {
  {
   VALUES ?property {
     rdfs:subClassOf
   }
   ?entity a owl:Class;
           owl:deprecated true ;
           ?property ?value .
   FILTER ( ?value NOT IN (oboInOwl:ObsoleteClass, owl:Thing) )
  }
  UNION
  {
   VALUES ?property {
     owl:equivalentClass
     owl:disjointWith
   }
   ?entity a owl:Class;
           owl:deprecated true ;
           ?property ?value .
  }
  UNION
  {
     VALUES ?property {
       rdfs:subClassOf
       owl:equivalentClass
       owl:disjointWith
     }
     ?entity a owl:Class;
             owl:deprecated true .
     ?value ?property ?entity .
  }
  UNION
  {
   VALUES ?property {
     owl:ObjectProperty
     owl:DataProperty
   }
   ?entity a owl:Class ;
           owl:deprecated true ;
           ?property ?value .
  }
  UNION
  {
   VALUES ?property {
     owl:someValuesFrom
     owl:allValuesFrom
   }
   ?value a owl:Class ;
          owl:deprecated true .
   ?rest a owl:Restriction ;
         ?property ?value .
   BIND("blank node" as ?entity)
  }
}
ORDER BY ?entity
```

###  qc-equivalent-classes.sparql

```
# description: checks for named equivalent classes, which are not allowed in Mondo

prefix oio: <http://www.geneontology.org/formats/oboInOwl#>
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE 
{
  VALUES ?property {
    owl:equivalentClass
  }
  
  ?entity ?property ?value .
  
  FILTER (!isBlank(?entity)) .
  FILTER (!isBlank(?value))
}
ORDER BY ?entity

```

###  qc-illegal-axiom-annotation.sparql

```
PREFIX IAO: <http://purl.obolibrary.org/obo/IAO_>
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

# Checks wether Axiom annotation are one of oboInOwl:source, oboInOwl:hasDbXref or oboInOwl:hasSynonymType .

SELECT distinct ?entity ?property ?value
WHERE 
{ 
      [
         a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty ?property ;
           owl:annotatedTarget ?x ;
           ?value ?y 
      ] .
    FILTER(?property!=rdfs:subClassOf && ?value!=rdfs:comment)
  	FILTER(?y!=?x && ?y!=?property && ?y!=?entity && ?y!=owl:Axiom)
  	FILTER ((?value != oboInOwl:source) && (?value != oboInOwl:hasDbXref) && (?value != oboInOwl:hasSynonymType)) .
    FILTER (isIRI(?entity) && regex(str(?entity), "^http://purl.obolibrary.org/obo/MONDO_"))

}
```

###  qc-illegal-properties.sparql

```
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE {
  {
  VALUES ?property { <http://purl.obolibrary.org/obo/mondo#source>
                     <http://www.geneontology.org/formats/oboInOwl#http://purl.obolibrary.org/obo/mondo#source> }
  ?entity ?property ?value .
  } UNION {
    VALUES ?entity { <http://purl.obolibrary.org/obo/mondo#source>
                     <http://www.geneontology.org/formats/oboInOwl#http://purl.obolibrary.org/obo/mondo#source> }
  	?entity ?property ?value .
  }
}
ORDER BY DESC(UCASE(str(?value)))

```

###  qc-missing-label.sparql

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>

SELECT DISTINCT ?entity ?property ?value WHERE {
 VALUES ?property { rdfs:label }
 ?entity ?any ?o .
 FILTER NOT EXISTS { ?entity ?property ?value }
 FILTER NOT EXISTS { ?entity a owl:Ontology }
 FILTER NOT EXISTS { ?entity owl:deprecated true }
 FILTER NOT EXISTS {
   ?entity rdfs:subPropertyOf oboInOwl:SubsetProperty .
 }
 FILTER EXISTS {
   ?entity ?prop2 ?object .
   FILTER (?prop2 != rdf:type)
   FILTER (?prop2 != owl:equivalentClass)
   FILTER (?prop2 != owl:disjointWith)
   FILTER (?prop2 != owl:equivalentProperty)
   FILTER (?prop2 != owl:sameAs)
   FILTER (?prop2 != owl:differentFrom)
   FILTER (?prop2 != owl:inverseOf)
 }
 FILTER (!isBlank(?entity))
 FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
}
ORDER BY ?entity
```

###  qc-misused-replaced-by.sparql

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE {
 VALUES ?property { <http://purl.obolibrary.org/obo/IAO_0100001> }
 ?entity ?property ?value .
 FILTER NOT EXISTS { ?entity owl:deprecated true }
 FILTER (?entity != oboInOwl:ObsoleteClass)
}
ORDER BY ?entity
```

###  qc-multiple-logical-definitions.sparql

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?entity ?property ?value WHERE {
 VALUES ?property { owl:equivalentClass }
 ?entity ?property [ owl:intersectionOf ?value ] .
 ?entity ?property [ owl:intersectionOf ?value2 ] .
 FILTER (?value != ?value2)
 FILTER (!isBlank(?entity))
}
ORDER BY ?entity
```

###  qc-no-genus-logical-definitions.sparql

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>


SELECT DISTINCT ?entity ?property ?value WHERE {
 VALUES ?property { owl:equivalentClass }
 ?entity ?property ?value .
 ?value rdf:type owl:Restriction .
 FILTER (!isBlank(?entity))
}
ORDER BY ?entity
```

###  qc-nolabels.sparql

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX replaced_by: <http://purl.obolibrary.org/obo/IAO_0100001>

SELECT DISTINCT ?entity ?property ?value WHERE {
	?entity a owl:Class
	FILTER NOT EXISTS {?entity rdfs:label ?lab}
	FILTER NOT EXISTS {?entity replaced_by: ?replCls}
  FILTER (!isBlank(?entity))
  BIND(rdfs:label as ?property)
  BIND("Must have a label" as ?value)
	FILTER (strstarts(str(?entity),"http://purl.obolibrary.org/obo/MONDO_"))
}
ORDER BY ?entity

```

###  qc-owldef-self-reference.sparql

```
prefix rdf:   <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix oio: <http://www.geneontology.org/formats/oboInOwl#>
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE 
{
  { ?entity owl:equivalentClass [ owl:intersectionOf [ rdf:rest*/rdf:first ?entity ] ] }
    UNION
  { ?entity owl:equivalentClass [ owl:intersectionOf [ rdf:rest*/rdf:first [ owl:someValuesFrom ?entity ] ] ] }
  BIND(owl:equivalentClass as ?property)
  BIND("Entity referencing itself in equivalentClass expression!" as ?value)
}
ORDER BY ?entity
```

###  qc-permitted-properties.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix IAO: <http://purl.obolibrary.org/obo/IAO_>
prefix RO: <http://purl.obolibrary.org/obo/RO_>
prefix mondo: <http://purl.obolibrary.org/obo/mondo#>
prefix skos: <http://www.w3.org/2004/02/skos/core#>
prefix dce: <http://purl.org/dc/elements/1.1/>
prefix dc: <http://purl.org/dc/terms/>

SELECT DISTINCT ?term ?property ?value WHERE 
{
	?term ?property ?value .
  	?property a owl:AnnotationProperty .
	FILTER (isIRI(?term) && regex(str(?term), "^http://purl.obolibrary.org/obo/MONDO_"))
  	FILTER(!isBlank(?value))
  FILTER (?property NOT IN (dce:creator, dce:date, IAO:0000115, IAO:0000231, IAO:0100001, mondo:excluded_subClassOf, mondo:excluded_from_qc_check, mondo:excluded_synonym, mondo:pathogenesis, mondo:related, mondo:confidence, dc:conformsTo, mondo:should_conform_to, oboInOwl:consider, oboInOwl:created_by, oboInOwl:creation_date, oboInOwl:hasAlternativeId, oboInOwl:hasBroadSynonym, oboInOwl:hasDbXref, oboInOwl:hasExactSynonym, oboInOwl:hasNarrowSynonym, oboInOwl:hasRelatedSynonym, oboInOwl:id, oboInOwl:inSubset, owl:deprecated, rdfs:comment, rdfs:isDefinedBy, rdfs:label, rdfs:seeAlso, RO:0002161, skos:broadMatch, skos:closeMatch, skos:exactMatch, skos:narrowMatch))
}

```

###  qc-reflexive.sparql

```
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?entity ?property ?value 
WHERE {
    ?entity ?property ?entity .
    FILTER ( isIRI(?entity))
    FILTER(?entity!=owl:Nothing)
    BIND("Entity referencing itself in triple!" as ?value)
}
ORDER BY ?entity
```

###  qc-related-exact-synonym.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE 
{ 
    ?entity oboInOwl:hasRelatedSynonym ?value ;
      oboInOwl:hasExactSynonym ?exact ;
      a owl:Class .
    
    ?entity rdfs:label ?entity_label .
    
    FILTER (str(?value)=str(?exact))
    FILTER (isIRI(?entity))
    BIND("oboInOwl:hasExactSynonym" as ?property)
}
ORDER BY ?entity
```

###  qc-same-label.sparql

```
SELECT DISTINCT ?entity ?property ?value WHERE {
?entity <http://www.w3.org/2000/01/rdf-schema#label> ?n0 .
?c2 <http://www.w3.org/2000/01/rdf-schema#label> ?n0 .
FILTER NOT EXISTS {?entity <http://www.w3.org/2002/07/owl#deprecated> ?value} .
FILTER NOT EXISTS {?c2 <http://www.w3.org/2002/07/owl#deprecated> ?v2} .
FILTER (?entity != ?c2)
BIND(<http://www.w3.org/2002/07/owl#deprecated> as ?property)
}
ORDER BY ?entity
```

###  qc-single-child.sparql

```
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix mondoSparqlQcGeneral: <http://purl.obolibrary.org/obo/mondo/sparql/qc/general/>
prefix mondoSparqlQcMondo: <http://purl.obolibrary.org/obo/mondo/sparql/qc/mondo/>
prefix mondo: <http://purl.obolibrary.org/obo/mondo#>

SELECT DISTINCT ?entity ?property ?value WHERE {
    ?entity rdfs:subClassOf ?parent .
    FILTER NOT EXISTS {
       ?child2 rdfs:subClassOf ?parent .
       FILTER (?child2 != ?entity)
       FILTER (isIRI(?child2) && STRSTARTS(str(?child2), "http://purl.obolibrary.org/obo/MONDO_"))
    }
    FILTER NOT EXISTS {
       ?entity mondo:excluded_from_qc_check mondoSparqlQcGeneral:qc-single-child.sparql .
    }
    FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
    FILTER (isIRI(?parent) && STRSTARTS(str(?parent), "http://purl.obolibrary.org/obo/MONDO_"))
    BIND(rdfs:subClassOf AS ?property)
    BIND(str(?parent) AS ?value)
}
ORDER BY ?entity
```

###  qc-subclass-cycle.sparql

```
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?entity ?property ?value WHERE {
    ?entity rdfs:subClassOf+ ?entity .
   FILTER(?entity!=owl:Nothing)
   BIND(rdfs:subClassOf AS ?property)
   BIND(?entity AS ?value)
}
ORDER BY ?entity
```

###  qc-syn-equal-label.sparql

```
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX replaced_by: <http://purl.obolibrary.org/obo/IAO_0100001>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>

SELECT distinct ?term ?label ?related ?synonym
WHERE 
{ 
  VALUES ?synonym { oboInOwl:hasRelatedSynonym oboInOwl:hasNarrowSynonym oboInOwl:hasBroadSynonym oboInOwl:hasCloseSynonym }
    ?term ?synonym ?related ;
          rdfs:label ?label ;
      	a owl:Class .
    
  	FILTER(str(?label)=str(?related))
    FILTER (isIRI(?term) && regex(str(?term), "^http://purl.obolibrary.org/obo/MONDO_"))
}
```

###  qc-syn-source-not-xref.sparql

```
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?entity ?property ?value
WHERE 
{ 
  VALUES ?property {
    oboInOwl:hasRelatedSynonym
    oboInOwl:hasExactSynonym
    oboInOwl:hasNarrowSynonym
    oboInOwl:hasBroadSynonym
  }
    ?entity ?property ?value ;
      a owl:Class .
      ?anno a owl:Axiom ;
           owl:annotatedSource ?entity ;
           owl:annotatedProperty ?property ;
           owl:annotatedTarget ?value ;
           oboInOwl:source ?xref .  
   FILTER (isIRI(?entity) && STRSTARTS(str(?entity), "http://purl.obolibrary.org/obo/MONDO_"))
}
```

###  qc-trailing-whitespace.sparql

```
# home: hp/sparql/trailing-whitespace-violation.sparql
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE 
{
  ?entity ?property ?value .

  FILTER( regex(STR(?value), "^ ") || regex(STR(?value), " $")  )
  FILTER( ?property != owl:annotatedTarget )
}
ORDER BY ?entity
```

###  qc-two-pattern.sparql

```
SELECT DISTINCT ?entity ?property ?value WHERE {
?entity <http://purl.org/dc/terms/conformsTo> ?value .
?entity <http://purl.org/dc/terms/conformsTo> ?n1 .
FILTER ( ?value!=?n1 ) .
FILTER ( isIRI(?c1))
BIND(<http://purl.org/dc/terms/conformsTo> AS ?property)
}
ORDER BY ?entity
```

###  qc-undeclared-subset.sparql

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX oio: <http://www.geneontology.org/formats/oboInOwl#>

SELECT DISTINCT ?entity ?property ?value WHERE {
   ?c oio:inSubset ?entity .
   FILTER NOT EXISTS { ?entity rdfs:subPropertyOf oio:SubsetProperty } 
   BIND(oio:inSubset AS ?property)
   BIND("Subset not declared." as ?value)
}
ORDER BY ?entity

```

###  qc-undeclared-synonym-type.sparql

```
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX oio: <http://www.geneontology.org/formats/oboInOwl#>

SELECT DISTINCT ?entity ?property ?value WHERE {
       ?axiom oio:hasSynonymType ?entity .
       FILTER NOT EXISTS { ?entity rdfs:subPropertyOf oio:SynonymTypeProperty } 
       BIND(oio:hasSynonymType AS ?property)
}
ORDER BY ?entity

```

###  qc-xref-syntax.sparql

```
# home: hp
prefix hasDbXref: <http://www.geneontology.org/formats/oboInOwl#hasDbXref>
prefix oio: <http://www.geneontology.org/formats/oboInOwl#>
prefix owl: <http://www.w3.org/2002/07/owl#>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?entity ?property ?value WHERE 
{
  ?entity hasDbXref: ?x .

  FILTER( regex(STR(?x), " ") || regex(STR(?x), ";") || STR(?x) = ""  )
  BIND(hasDbXref: AS ?property)
}
ORDER BY ?entity
```
