# SHACLValidatorAAS


# Usage
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

![image](https://github.com/mhrimaz/SHACLValidatorAAS/assets/17963017/ec492a8a-9002-4a88-bf90-e42d2922d03e)
