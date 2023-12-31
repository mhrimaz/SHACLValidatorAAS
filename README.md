# SHACLValidatorAAS

[Presentation slides](https://github.com/mhrimaz/SHACLValidatorAAS/blob/main/SF_ETFA2023_SHACL_Paper.pdf)

Proof of concept for AAS to RDF with RML and use SHACL shape to validate AAS content.

Since we faced so many issues with RML and there are many limitations, RML can't really be used for AAS to RDF conversion. We suggest to use [aas4j](https://github.com/eclipse-aas4j/aas4j) library for RDF conversion.

If you are using our idea, we would appreciate to site our paper:

```bibtex
@INPROCEEDINGS{10275629,
  author={Rimaz, Mohammad Hossein and Plociennik, Christiane and Kunz, Leonhard and Ruskowski, Martin},
  booktitle={2023 IEEE 28th International Conference on Emerging Technologies and Factory Automation (ETFA)}, 
  title={Am I in Good Shape? Flexible Way to Validate Asset Administration Shell Data Entry via Shapes Constraint Language}, 
  year={2023},
  volume={},
  number={},
  pages={1-6},
  doi={10.1109/ETFA54631.2023.10275629}}
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


Another example:
The constraint: `The price for Berlin should be more than 200`

`shacl.bat validate --shapes 002_Dummy_SHACL_sparql.ttl --data 002_Dummy_simple_AAS_one_property.ttl`

```turtle
@prefix aas: <https://admin-shell.io/aas/3/0/> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix xs: <http://www.w3.org/2001/XMLSchema#> .
@prefix ex: <http://www.dfki.de/> .


ex:ExpensiveCityShape a sh:NodeShape ;
    sh:targetClass aas:Submodel;
    
	sh:sparql [
		a sh:SPARQLConstraint ;
		sh:message "The price for Berlin cannot be less than 200." ;
		sh:select """
		SELECT ?this
		WHERE {
		  ?s1 <https://admin-shell.io/aas/3/0/Property/value> ?value1; 
			 <https://admin-shell.io/aas/3/0/Referable/idShort> ?o1;
		  Optional{
			?s2 <https://admin-shell.io/aas/3/0/Property/value> ?value2; 
				<https://admin-shell.io/aas/3/0/Referable/idShort> ?o2;
		  }
		  Filter(?o1 = "InstallLocation" &&  ?value1 = "Berlin" && ?o2 = "Price" && ?value2 < 200)  
		}
		"""
	] 
    
.
```

![image](https://github.com/mhrimaz/SHACLValidatorAAS/assets/17963017/125e2b35-fb33-4a51-b8ed-de29e7925fc6)


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
