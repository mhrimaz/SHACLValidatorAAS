# SHACLValidatorAAS

Proof of concept for AAS to RDF with RML and use SHACL shape to validate AAS content.

Since we faced so many issues with RML and there are many limitations, RML can't really be used for AAS to RDF conversion. We suggest to use [aas4j](https://github.com/eclipse-aas4j/aas4j) library for RDF conversion.

If you are using our idea, we would appreciate to site our paper:

```bibtex
Will be added.
```

## Limmitations
- RML conversion does not handle all data types and metamodel elements (only Property).
- Provided YARRML won't work with [Matey](https://rml.io/yarrrml/matey/). `xs` namepsace in `xs:double` automatically replaced by the engine. As a shortcut you can remove `xs` in both JSON and YARRML file as a shortcut.
- In examples provided by the aas-spec repository, blank node used for submodelElements. This makes targeting specific elements hard. Furthermore, the datatype of literals are always string.

### SHACL Usage

For SHACL validation we used Apache Jena [apache-jena-4.9.0](https://dlcdn.apache.org/jena/binaries/apache-jena-4.9.0.zip). `JAVA_HOME` and `JENA_HOME` should be in your environment variables.

Use the following command to run validation:

`shacl.bat validate --shapes 002_Dummy_SHACL_Dummy_simple_AAS_one_property.ttl --data 002_Dummy_simple_AAS_one_property.ttl` 


Sample SHACL with SPARQL-Target

```turtle
@prefix dash: <http://datashapes.org/dash#> .
@prefix aas: <https://admin-shell.io/aas/3/0/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix schema: <http://schema.org/> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix xs: <http://www.w3.org/2001/XMLSchema#> .
@prefix ex: <http://www.dfki.de/> .


ex:CityShape a sh:NodeShape;
    a sh:NodeShape ;
	sh:message "Only 3 Cities is Valid: Kaiserslautern, Berlin, Pirmasens" ;
    sh:target [
        a sh:SPARQLTarget ;
        sh:select """
            select ?s where {
           ?s <https://admin-shell.io/aas/3/0/Referable/idShort> ?o;
              <https://admin-shell.io/aas/3/0/Referable/idShort> ?o;
           FILTER(?o = "InstallLocation")
            }
            """ ;
    ] ;
    sh:property [
		sh:path <https://admin-shell.io/aas/3/0/Property/value> ;
		sh:in ("Kaiserslautern" "Berlin" "Pirmasens");
    ] 
.
```


When the content is Valid

![image](https://github.com/mhrimaz/SHACLValidatorAAS/assets/17963017/9ad08b6f-356a-4faa-9ba2-057515c0feb4)

When you change the `InstallLocation` to an invalid value like `Paris` you will get the following errror:

![image](https://github.com/mhrimaz/SHACLValidatorAAS/assets/17963017/e778fe98-5d44-4eb6-b4bd-47dc764f0dac)




### AAS to RDF Usage
Example input:
```json
{
  "id":"Installation672023",
  "idShort":"InstallationInformation",
  "kind":"Instance",
  "modelType":"Submodel",
  "semanticId":{
    "keys":[
      {
        "type":"GlobalReference",
        "value":"www.example.com/ids/sm/InstallationInformation"
      }
    ],
    "type":"ExternalReference"
  },
  "submodelElements":[
    {
      "idShort":"InstallLocation",
      "modelType":"Property",
      "value":"Kaiserslautern",
      "valueType":"xs:string"
    },
    {
      "idShort":"Price",
      "modelType":"Property",
      "value":"200",
      "valueType":"xs:double"
    }
  ]
}
```

The input should be in the `data.json` file. [rmlmapper-6.2.1-r368-all.jar](https://github.com/RMLio/rmlmapper-java/releases/download/v6.2.1/rmlmapper-6.2.1-r368-all.jar) used for the mapping.
With the following command you should get the output in the image: 

`java -jar rmlmapper-6.2.1-r368-all.jar -m input_submodel_mapping.txt`

![image](https://github.com/mhrimaz/SHACLValidatorAAS/assets/17963017/eec7968f-a56f-4828-88d8-0ca81b6d3a62)


## YARRML

We used YARRML to generate the RML file, however as mentioned in the limitations, we faced a bug, and we had to manually change the generated RML.

```yaml
prefixes:
  grel: http://users.ugent.be/~bjdmeest/function/grel.ttl#
  idlab-fn: http://example.com/idlab/function/
  aas: "https://admin-shell.io/aas/3/0/"


mappings:
  submodel:
    sources:
      - ['data.json~jsonpath', '$']
    s: http://example.com/submodels/$(id)
    po:
      - [a, aas:Submodel]
      - [https://admin-shell.io/aas/3/0/Identifiable/id, $(id)]
      - [https://admin-shell.io/aas/3/0/HasKind/kind, https://admin-shell.io/aas/3/0/ModellingKind/$(kind)~iri]
      - p: https://admin-shell.io/aas/3/0/HasSemantics/semanticId
        o:
         - mapping: semanticid
      - p: https://admin-shell.io/aas/3/0/Submodel/submodelElements
        o:
         - mapping: submodelElementDoubleProperty
      - p: https://admin-shell.io/aas/3/0/Submodel/submodelElements
        o:
         - mapping: submodelElementStringProperty
 
           
      
  semanticid:
    sources:
      - ['data.json~jsonpath', '$.semanticId']
    s: null 
    po:
      - [rdf:type, aas:Reference]
      - [https://admin-shell.io/aas/3/0/Reference/type, https://admin-shell.io/aas/3/0/ReferenceTypes/$(type)~iri]
      - p: https://admin-shell.io/aas/3/0/Reference/keys
        o:
         - mapping: keys   
      
  keys:
    sources:
      - ['data.json~jsonpath', '$.semanticId.keys.*']
    s: null 
    po:
      - [rdf:type, aas:Key]
      - [https://admin-shell.io/aas/3/0/Key/type, https://admin-shell.io/aas/3/0/KeyTypes/$(type)~iri]
      - [https://admin-shell.io/aas/3/0/Key/value, $(value)]
      
  submodelElementStringProperty:
    sources:
      - ['data.json~jsonpath', '$.submodelElements.*']
    s: null
    condition:
      function: idlab-fn:equal
      parameters:
        - [grel:valueParameter, "xss:string"]
        - [grel:valueParameter2, $(valueType)]
    po: 
      - [rdf:type, aas:Property]
      - [https://admin-shell.io/aas/3/0/Referable/idShort, $(idShort)]
      - [https://admin-shell.io/aas/3/0/Property/valueType, https://admin-shell.io/aas/3/0/DataTypeDefXsd/String~iri]
      - [https://admin-shell.io/aas/3/0/Property/value, $(value)]
      
  submodelElementDoubleProperty:
    sources:
      - ['data.json~jsonpath', '$.submodelElements.*']
    s: null
    condition:
      function: idlab-fn:equal
      parameters:
        - [grel:valueParameter, "xss:double"]
        - [grel:valueParameter2, $(valueType)]
    po: 
      - [rdf:type, aas:Property]
      - [https://admin-shell.io/aas/3/0/Referable/idShort, $(idShort)]
      - [https://admin-shell.io/aas/3/0/Property/valueType, https://admin-shell.io/aas/3/0/DataTypeDefXsd/Double~iri]
      - [https://admin-shell.io/aas/3/0/Property/value, $(value)]

       
```
