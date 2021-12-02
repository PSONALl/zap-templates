# zap-templates

## Zap files

* ZAP is a generic templating engine ZCL code and generated configuration files came from Appbuilder a Silabs tools previously.
* Provides UI for the end-user to select specific application configuration (clusters, attributes, commands, etc.)
* Customer data is saved using JSON format and Zap back-end is a node.js application.
* Users can always change the variant inside *.zap file. - Migrating the *.zap file from one SDK to another, where a template version is different, will result in the variant inside *.zap file being changed.
* Inputs to the generator are: i) ZCL metadata files (either Silicon Labs XML, or Zigbee XML) generation template files with customizations
ii) Custom owned *.zap file.
* Outputs from the generator are: Generated files that the embedded code requires (found in application's `gen/` folder) and any other possible build files or others, as required by the SDK.
 
## Design Application Framework
For details about designing your own application framework, for implementing Application Framework Directory Structure, ZCL Concepts, and to Design Application Framework API, you can follow respective Section of [Zigbee official XML format of the ZCL metafiles](https://www.silabs.com/documents/public/user-guides/ug391-zigbee-app-framework-dev-guide.pdf)
 
## Zap Schema
 
* Zap schema is created from this Zigbee official XML format’s metadata can be found [here](https://github.com/project-chip/zap/blob/master/src-electron/db/zap-schema.sql), which is diagramattically represented [here.](https://github.com/project-chip/zap/blob/master/docs/zap-schema.svg)
* You can also create custom schema like this. In above schema:
       - _ID postfix, are the primary keys 
       - _REF postfix, are the foreign key columns
* This schema can be refered to add objects like packages, static data objects like endpoint types, clusters, attributes, commands
* Data types and values to be added in the each object of databases can be referred from here
* For more details you can refer current [schema design](https://github.com/project-chip/zap/blob/master/docs/design.md)

## Template Meta Files
 
* To know which Application Configuration Files should be generated you can refer “Generate Application Configuration Files” section from [Zigbee official XML format of the ZCL metafiles](https://www.silabs.com/documents/public/user-guides/ug391-zigbee-app-framework-dev-guide.pdf)
* Generation templates and extensions: these are actual ZAP templates `(*.zapt files)`, which correspond to the files that need to be generated. The entry point is a JSON formatted file, named [app-templates.json](https://github.com/project-chip/connectedhomeip/blob/master/src/app/zap-templates/app-templates.json) which lists all the individual template files and additional extension options. For development of each individual template, you can follow the [template tutorial](https://github.com/project-chip/zap/blob/master/docs/template-tutorial.md)


## Metafiles 

* SDK is essentially a body of code that implements the ZCL concepts, such as clusters, attributes, commands, etc.
* ZCL metafiles: these are the metafiles that provide information about the ZCL specification to zap tool.
* Need to provide the accurate XML packages for their SDK, is to let ZAP know where they can be loaded from on the local hard drive. 
* Xml packages are generated from zigbee cluster library so need not to be changed
* They can be found [here](https://github.com/project-chip/connectedhomeip/tree/master/src/app/zap-templates/zcl/data-model)
* For more details see SDK integration https://github.com/project-chip/zap/blob/master/docs/sdk-integration.md

## Adding Custom ZCL entities

* To add your own device type or cluster or endpoint, you need to add your custom XML file or your data to existing [xml files](https://github.com/project-chip/connectedhomeip/blob/master/src/app/zap-templates/zcl/data-model)
* This allows for different manufacturers and users to create their own specific functionality that interacts with the ZCL data model.
* For more details see: [Custom-ZCL](https://github.com/project-chip/zap/blob/master/docs/custom-zcl.md)

**Syntax:**

```
endpoint types[
   //endpoint 0
  {
      "name":
        "deviceTypeName"
        "deviceTypeCode"
        "deviceTypeProfileId”
        clusters[
           //cluster 1
           {   
                  //attributes
                 "attributes" [
                        //attribute 1
                        {}  
                        .   
                        .   
                  ]   
           }   
          //cluster 2
          .
          .
          .
       ]
   }
   //endpoint 1
    .
    .
    .
],
endpoints[
    //endpoint 0 with Id
   {}
    //endpoint 1 with Id
    .
    .
    .
]
```
Data objects and their data types can be referred from the above schema to create following objects into database


**1. Endpoints**

* Endpoint type is combination of clusters which can be configured as per the use case
* We can add endpoint in th endpoint types array of zap file
* For each endpoint we can configure clusters we want to add in the clusters array of particular enddpoint 

```
 
"endpointTypes": [
    {    
      "name":    //Endpoint name (text value specified in xml data)
      "deviceTypeName":       //device name  (text value specified in xml data)
      "deviceTypeCode":       //device typecode (random value generated by sql script)
      "deviceTypeProfileId":  //profile Id 
.
.
.
    }]
```

 * Profile Id for a device can be selected following rules in the (Section 2.3.3.2 Profile ID Endpoint Matching Rules) from [zigbee specification](https://zigbeealliance.org/wp-content/uploads/2019/11/docs-05-3474-21-0csg-zigbee-specification.pdf)


**2. Clusters & Commands**

For adding a cluster you need to follow this semantics:
```
        {    
          "name":          // name registered this in xml file
          "code":          // code for command in zcl specs
          "mfgCode":       // id for manufacturer
          "define":        // name registered and defined in xml files
          "side":          // side should be mentioned
          "enabled":      // enables the side
  }
```
Every command has semantics:
```
commands": [
            {    
              "name":             // name of the command as per in xml files
              "Code":             // code mentioned for command in zcl specs
              "mfgCode":          // id for manufacturer
              "source":           // source side
              "incoming":         // transmission incoming
              "outgoing":         // transmission outgoing
            },  
```
* You will  find detailed Information, Ids, Default values etc. for any cluster in respective section for a cluster defined in [Zigbiee Cluster Library Specs](https://zigbeealliance.org/wp-content/uploads/2019/12/07-5123-06-zigbee-cluster-library-specification.pdf)
* You can see more details in Client Server Architecture from [Zigbiee Cluster Library Specs](https://zigbeealliance.org/wp-content/uploads/2019/12/07-5123-06-zigbee-cluster-library-specification.pdf)
* You need to add cluster for both client and server side. It helps server to register the commands which may come from client side. Server side must have enabled and included all the database attributes so that client can manipulate over them.



Note : "mfgCode": is null because The Zigbee application framework does not currently support overlapping manufacturer-specific cluster IDs within a single device. In other words, you cannot implement cluster 0xFC00 with manufacturer code 0xFEED AND cluster 0xFC00 with manufacturer code 0xBEEF on the SAME device.  



**3. Attributes**

Semantics:
```
"attributes": [
            {
              "name": 
              "code":                     
              "mfgCode":    
              "side": 
              "included":     
              "storageOption": 
              "singleton": 
              "bounded": 
              "defaultValue":  
              "reportable": 
              "minInterval": 
              "maxInterval": 
              "reportableChange": 
            },
```
* Values for name of attribute, code Id, default value can be referred by finding particular cluster’s attribute section from [Zigbiee Cluster Library Specs](https://zigbeealliance.org/wp-content/uploads/2019/12/07-5123-06-zigbee-cluster-library-specification.pdf)

* To configure other of these attributes refer details from [(section-> 10.1 ZCL Attribute Configuration) In Zigbee official XML format of the ZCL metafiles](https://www.silabs.com/documents/public/user-guides/ug391-zigbee-app-framework-dev-guide.pdf)

Note: Cluster revision attribute is must for every cluster for both client as well as server side
To generate configuration files the command is:
```
./scripts/tools/zap/generate.py <path to *.zap file> -t <path to templates.json file>
```
Here you can find about commands to generate zap templates: [zap-templates generation](https://github.com/project-chip/connectedhomeip/tree/master/src/app/zap-templates#readme)

## Zap UI

We can run ZAP with UI to configure endpoints and clusters. Steps for Adding a new endpoint/cluster through zap UI are:
1. Run script: `./scipts/tools/zap/run_zaptool.sh`. 
2. This script will install all the dependencies required to run zap UI
And then starts configurator.

3. For adding a new endpoint/cluster or modifying the existing one, select `+Add EndPoint`.
4. Select Endpoint, profile ID, Device type, Network Id and Version of Endpoint you want to add or modify and create an endpoint.
5. You will be displayed all existing Zigbee Clusters which you can add to this endpoint.
6. You can add any of the clusters in this endpoint to client/server or both sides by choosing appropriate option given for the cluster, this will add that cluster to the selected endpoint.
7. After adding all endpoints click on Save option. The custom configuration will be saved as a `.zap` file
8. If you want to add custom clusters and templates you can add them using `+Add custom ZCL` functionality.
	- You can use this functionality to add custom ZCL clusters or commands to the Zigbee Clusters Configurator.
	- Create your own metadata .json files and upload them.

 
**For more details refer:**

* [ZigBee Cluster Library Specification](https://zigbeealliance.org/wp-content/uploads/2019/12/07-5123-06-zigbee-cluster-library-specification.pdf)

* [Zigbee Application Framework Developer's Guide](https://www.silabs.com/documents/public/user-guides/ug391-zigbee-app-framework-dev-guide.pdf)

* [ZigBee Pro Specification](https://zigbeealliance.org/wp-content/uploads/2019/11/docs-05-3474-21-0csg-zigbee-specification.pdf)

* [Project-chip zap repo](https://github.com/project-chip/zap)





