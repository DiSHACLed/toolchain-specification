
# Table Of Contents

- [1. Overview](#overview)
    - [1.1. Introduction](#11-introduction)
    - [1.2 Term definitions](#12-term-definitions)

- [2. Ontology](#2-ontology)
    - [2.1. Namespaces](#21-namespaces)
    - [2.2. Relation to other ontologies](#22-relation-to-other-ontologies)
    - [2.3. Core Classes](#23-core-classes)
    - [2.4. Additional Classes](#24-additional-classes)
    - [2.5. Future Directions](#25-future-directions)

- [3. Pipeline Generator Specification](#pipeline-generator-specification)
    - [3.1. High-level overview](#31-high-level-overview)
    - [3.2. Additional Details](#32-additional-details)
    - [3.3. Frontend](#33-frontend)
    - [3.4. Monitoring](#34-monitoring)
<br><br>

# 1. Overview
## 1.1 Introduction
*Intention*: discoverability, replication, modular design, separation of concerns, ease of use 
<br><br>
The goal is to design a framework-agnostic, streaming-oriented pipeline orchestration system for Linked Data. Pipelines are described semantically using RDF, and execution is realized across multiple frameworks (currently LDIO, RDFConnect, semantic.works) using Docker containers and RDFConnect for orchestration. For this purpose, the *toolchain* ontology is developed.

The *toolchain* ontology allows to describe data pipelines for the purpose of replicability. Four Core Classes are established, each of which are concerned with different responsibilities. Providing a minimal **Pipeline Definition** is sufficient to allow replication of a pipeline. Pipelines are defined as a series of data transformation steps, which are executed by Pipeline Components. These Pipeline Components are collected in a **Component Catalogue**, which hence provides the resources for building a pipeline. A Pipeline Generator takes a Pipeline Definition and attempts to build an executable pipeline based on the resources at its disposal in the Component Catalogue. The **Pipeline Build** reflects  the compiled pipeline and provides all information concerning instantiation of a pipeline in a specific Environment.
<br><br>
Splitting the *toolchain* ontology into these four Core Classes leaves a clean separation of concerns: The Pipeline Definition allows the user to create new pipelines with minimal effort: It is sufficient to describe which Processors provide data to other Processors, and to provide Configs to configure each Processor used in the Pipeline. The Component Catalogue is based on the idea that each Pipeline Component is modular and should be easy to reuse. Hence, all information that is known about a PipelineComponent (and which may be relevant for a Pipeline Generator) should be included in the Component Catalogue. This allows a user to merely point to the Processor in the Component Catalogue when defining a particular Pipeline Step. Hence, essential details about the Processor do not need to be repeated with each new Pipeline Definition. Likewise, distinguishing between the Pipeline Definition and the Pipeline Build allows defining Pipelines without having to be  concerned with their implementation. It is up to the Pipeline Generator to figure out a feasible implementation of the pipeline, allowing to largely automate this process: Based on what is known about each Pipeline Component, the feasibility of the defined Pipeline can be evaluated and a corresponding Pipeline Build can be generated.  
<br>
Even if the Pipeline Generator is not utilized, the *toolchain* ontology provides a unified language for describing pipelines across different frameworks. This has several advantages. Pipelines and Pipeline Components become discoverable, allowing a community to grow around the idea of sharing modular resources for building pipelines. Datasets can be described by the pipelines that generated them, allowing full replication in research. Pipeline Definitions can be subjected to automated validation, so that the feasibility of pipelines does not have to be evaluated through trial and error. 
<br><br>

## 1.2 Term definitions

| Term | Definition |
| ----- | ----- |
| **Pipeline** | We understand a Pipeline as an arrangement of modular data transformation steps, whereby each step acts on data in a well-defined, configurable way, before forwarding the data to the next step. These steps can be arranged in a nonlinear way, similar to a directed graph: Steps can receive zero or more inputs and outputs. To qualify as Pipeline, all Steps have to be directly or indirectly chained to each other. |
| **Data** | Data is any kind of digital content that carries information of interest. This information can be extracted through transformation and made available through transportation, i.e. through Pipelines. As such, Data are the entities that are send through pipeslines to make information available at a target location. |
| **Config** | A Config defines the expected (transformation) behavior for each Step in a Pipeline. As such, it differs from Data in so far as it is not subject to transformation itself, does not provide much valuable information beyond the behavior within the Pipeline, and is also not send through the Pipeline. | 
| **Instance** | We understand instantiating Pipeline Components as installing or deploying these in Environments so that they are ready to run. | 
<br><br>


# 2. Ontology
## 2.1. Namespaces
| prefix |	namespace IRI | documentation |
|----------|----------|----------|
| tc  | toolchain  | the document you are currently reading |
| rdf  | http://www.w3.org/1999/02/22-rdf-syntax-ns#  | https://www.w3.org/TR/rdf11-concepts/ |
| rdfs | http://www.w3.org/2000/01/rdf-schema#  | https://www.w3.org/TR/rdf-schema/ |
| rdfc | https://rdf-connect.github.io/ontology/ | https://rdf-connect.github.io/specification/ |
| p-plan  | http://purl.org/net/p-plan# | https://vocab.linkeddata.es/p-plan/index.html |
| prov  | http://www.w3.org/ns/prov# | https://www.w3.org/TR/prov-o/ |
| osw  | http://ontosoft.org/software#  | https://ontosoft-earthcube.github.io/ontosoft/ontosoft%20ontology/v1.0.1/doc/index.html |
| sosa | http://www.w3.org/ns/sosa/ | https://www.w3.org/TR/vocab-ssn/ |
| ssn | http://www.w3.org/ns/ssn/ | https://www.w3.org/TR/vocab-ssn/ |
| sh  | http://www.w3.org/ns/shacl#  | https://www.w3.org/TR/shacl/ |
| dcat  | http://www.w3.org/ns/dcat#  | https://www.w3.org/TR/vocab-dcat-3/ |
<br><br>

## 2.2. Relation to other ontologies
- For the *Pipeline Definition* the p-plan ontology is used. This is because the *Pipeline Definition* reflects a plan of a pipeline to be built and executed.
- For the *Component Catalogue* the osw-ontology is used, which allows extensive metadata annotation of software, and hence provides sufficient terms for defining PipelineComponents exhaustively. 
- For the *Pipeline Build* a mix of prov and sosa is used. Prov is used for describing the Pipeline Build as a prov:SoftwareAgent, which allows to define Pipeline Runs as Activities of the Pipeline Build. Sosa is used for describing the Pipeline Build as a sosa:System, which allows describing how this system may be hosted in different Environments (sosa:Platforms). 
- A link with dcat is made twice: A Pipeline Run is a prov:Activity that used and generated dcat:datasets, allowing to link datasets to the pipelines that generated them. A *Component Catalogue* is also a dcat:dataset by itself, providing a means for publishing and sharing Pipeline Components with a wider audience. 
<br><br>

## 2.3. Core Classes
<br>

### Pipeline Generator
| | |
|----------|----------|
| **Definition** | A Pipeline Generator is a reasoning service that takes a Pipeline Definition and one or more Component Catalogues as input and produces a Pipeline Build as output. In other words, a Pipeline Generator suggests a system capable of executing a Pipeline Definition based on the Components it has at its disposal as resources. It considers osw:UsesAndAssumptions of each PipelineComponent to evaluate the feasibility of suggesting a Pipeline Build. As such it should be sufficient for a user to define a Pipeline Definition to arrive at a Pipeline Build capable of running the pipeline. A Pipeline Generator could hence automate the task of building a pipeline, making it sufficient for the user to formulate the intended pipeline. |
| **subclass of** | prov:SoftwareAgent |
<br>

### Pipeline Definition
| | |
|----------|----------|
| **Definition** | In its essence, a Pipeline Definition is a directed graph of planned PipelineSteps, each aimed to generate or transform data. For this purpose, each Pipeline Step points at a Processor, which is a Pipeline Component appointed to carry out the Pipeline Step. A Pipeline Step also receives a Config in order to define the expected behavior of the Processor during the Pipeline Step. A Pipeline Step can correspond to a Pipeline Definition, making nesting possible. <br><br> A Pipeline Definition declares intent; it is a plan of a pipeline to be run. |
| **subclass of** | p-plan:Plan |
<br>

### Component Catalogue
| | |
|----------|----------|
| **Definition** | A Component Catalogue is a collection of the PipelineComponents that can be instanced in order to create an Pipeline Build capable of executing a Pipeline Definition. Machine-readable installation instructions (thus steps needed for instancing the component) can be expressed as Dockerfiles. Dependencies between PipelineComponents can be expressed to ensure that instancing a PipelineComponent includes instancing the supporting Runners in the respective Environments. 
| **subclass of** |prov:Collection, dcat:dataset |
<br>

### Pipeline Build
| | |
|----------|----------|
| **Definition** | An Pipeline Build is the description of a system capable of executing the Pipeline Definition. It has to be sufficiently described to allow reproducibility (in contrast to the Pipeline Definition, whose main concern is declaring intent). As such, the Pipeline Build consists of a collection of ProcessorInstances, which are responsible for executing PipelineSteps. A pipeline does not run in a vacuum, hence it is also needs to be defined in which Environment these PipelineComponents are instanced. These Environments are also described as Dockerfiles. Furthermore, data transfer between ProcessorInstances is defined via Channels. This allows adding fields related to this data transfer. Runners may be instanced as well. These are PipelineComponents that Processors depend on, however they are not responsible for transforming or generating data themselves. |
| **subclass of** |sosa:System, prov:SoftwareAgent |
<br>



## 2.4. Additional Classes
<br>

### PipelineStep
| | |
|----------|----------|
| **Definition** | See Pipeline Definition. |
| **subclass of** | p-plan:Step |


<br>

### Config
| | |
|----------|----------|
| **Definition** | A Config is a data structure equivalent to the "object" type in JSON, consisting of an unordered set of name/value pairs. The name is a string and the value is a string, number, boolean, array, or object (same concept as in the [Common Workflow Language](https://www.commonwl.org/v1.2/Workflow.html#Data_concepts)). Defining a Config in this way allows for both simple configuration of Pipeline Steps (a simple set of parameter names and their values), as well as complex, nested configurations. <br><br> A Config can be serialized in different formats, and this format can differ from the format that the instanced PipelineComponent expects during runtime. As of now, we support RDF, JSON, yaml as formats. If the Config is serialized as RDF, it can directly be embedded in the RDF graph that makes up the Component Catalogue: Using the "embedded"-predicate, the Config should point to a blank node representing a data object: Field-names serve as predicates and field-values as objects. The subgraph originating from the root blind node must be acyclic to later allow conversion between formats. If the Config is serialized as JSON or yaml, it cannot be embedded in the RDF graph. In this case, the predicate "external" should either point to a url that is dereferencable. Alternatively, the predicate "literal" should point to a string which is of the corresponding format. A Config should also declare the serialization format it uses (json, yaml), although this may be inferred. |
| **subclass of** | p-plan:Variable |


<br>

### Processor
| | |
|----------|----------|
| **Definition** | A Processor is any modular unit that can generate or transform data when instantiated. As a class it differs from a ProcessorInstance in that it is a blueprint intended for reuse. To allow easy reuse, the Component Catalogue has to provide enough information about the Processor so that it can be instantiated as part of the Pipeline Build based on the provided information. |
| **subclass of** | tc:PipelineComponent |


<br>

### Runner
| | |
|----------|----------|
| **Definition** | A Runner is any modular piece of software needed as instance in the Pipeline Build to run a pipeline, but not responsible for transforming and forwarding data. It is needed in the Pipeline Build because either a Processor or another Runner depends on it. |
| **subclass of** | tc:PipelineComponent |


<br>

### PipelineComponent
| | |
|----------|----------|
| **Definition** | A Pipeline Component is a superclass of Processors and Runners. |
| **subclass of** | osw:Software |


<br>

### osw:UsesAndAssumptions
| | |
|----------|----------|
| **Definition** | In order to evaluate the feasibility of turning a Pipeline Definition into an Pipeline Build, the PipelinePlanner has to know the UsesAndAssumptions of the PipelineComponents in the ComponentCatalogue. These can be expressed as sh:NodeShapes that target different aspects of a pipeline: NodeShapes can express the kinds of data that a Processor can transform and the output it produces. NodeShapes also allow to express mandatory and optional fields of a Config that a Processor can interpret. Constraints can also be expressed on the level of the PipelineDefinition: For example, Processors may have specific constraints about the relationships they allow with other processors (via isPrecededBy). For example, an "API-call"-processor may not allow data input through a preceding Step, because it fetches new data from a remote source. Similarly, constraints can be laid on the level of the PipelineBuild, for example about the Channel to be used to transfer data. <br><br> Taken together, defining UsesAndAssumptions are a powerful tool to formulate explicit constraints that modular pipeline components impose when instantiated. These constraints can be validated to evaluate the feasibility of creating a Pipeline Build. They can also form a basis for the Pipeline Generator to reason about viable Pipeline Builds given the Pipeline Definition and Component Catalogue. These constraints are expressed as SHACL-shapes to allow automatic validation. Each Constraint should have a sh:message for human readibility. These should clearly indicate and describe what kind of constraint is imposed, to allow easier implementation of the logic behind the constraint. <br><br> In case a Config is expressed via "external" or "literal" it may be preferable to express a Config Schema in a format that may not easily expressed in RDF, such as JSON-schema. 
| **subclass of** | --- |


<br>


### SPARQL Construct
| | |
|----------|----------|
| **Definition** | If a SHACL shape is violated, a predefined SPARQL Construct query can be triggered by the Pipeline Generator. This is useful for defining an assumption that must be checked, and attempting to fix that assumption when violated. <br><br> Example: The shape could check whether the Channel that tc:writesTo the tc:ProcessorInstance has the same port number as in the Config of the PipelineStep that the ProcessorInstance implements. If this is not the case, it could trigger the SPARQL query that adds the port number to the Channel. |
| **subclass of** | --- |
| **Domain** | tc:onViolation |

<br>

### ProcessorInstance
| | |
|----------|----------|
| **Definition** | A ProcessorInstance is an instance of a Processor, linked to other ProcessorInstances via Channels, and responsible for executing a Pipeline Step. |
| **subclass of** | tc:PipelineComponentInstance |


<br>

### RunnerInstance
| | |
|----------|----------|
| **Definition** | A RunnerInstance is an instance of a Runner. |
| **subclass of** | tc:PipelineComponentInstance |


<br>

### PipelineComponentInstance
| | |
|----------|----------|
| **Definition** | A PipelineComponentInstance is an instance of a PipelineComponent and hence a superclass of ProcessorInstance and RunnerInstance. As a SubSystem of the PipelineBuild, it is responsible for executing (part of) a pipeline definition. |
| **subclass of** | prov:SoftwareAgent |


<br>

### ConfigInstance
| | |
|----------|----------|
| **Definition** | A ConfigInstance is a "compiled" Config as part of an Pipeline Build. There are several reasons as of why this instanced Config may slightly differ from the Config as part of the Pipeline Definition: Configs may be serialized in different formats, but a PipelineComponentInstance may expect a specific format. A Pipeline Generator may implement changes to a ConfigInstance. Several Configs may be compiled to one combined ConfigInstance. |
| **subclass of** | prov:Entity |

| **Range** | prov:value |
<br>

### Channel
| | |
|----------|----------|
| **Definition** | A Channel links two or more ProcessorInstances together. In other words, it describes how ProcessorInstances forward data to each other. You can picture ProcessorInstances as nodes, and Channels as edges in a graph. The Channel entity allows to qualify this interdependence further, for example by defining the port, the protocol, the mode, etc. of the data transfer. |
| **subclass of** | --- |


<br>

### Environment
| | |
|----------|----------|
| **Definition** | An Environment reflects the Runtime Environment(s) in which Pipeline Components are instantiated to prepare for a Pipeline Run. As such an Environment is mainly defined through its Dockerfile, which is machine-readible way of defining this runtime environment. |
| **subclass of** | sosa:Platform |


<br>



### Dockerfile
| | |
|----------|----------|
| **Definition** | A Dockerfile provides machine-readable instructions on how to instantiate a Pipeline Component (or Environment consisting of PipelineComponents). A Pipeline Generator may not rely on Docker, but it must be able to read Dockerfiles. |
| **subclass of** | osw:TextEntity |
<br>

## 2.5. Future Directions 
- In this model, pipelines are only described as concrete instances, not as reusable templates. In my view, a pipeline automatically becomes reusable by simply changing the used data and potentially some configuration values. But some people may see that differently and may want distinction of class and instance for pipelines. 
- We may want to be able to express the state of a pipeline run while it is still ongoing. This remains to be discussed, because in the current scope the rdf graph is only used to compile a pipeline build, but not used to monitor the pipeline progress. 
- We may not always want to build each PipelineComponent, some PipelineComponents may already be running. We have to think more about how to express this and how this would work in practice. 
- We may want to be able to express that a PipelineComponent requires one of several runners.
- We may want to be able to provide hints to the Pipeline Generator for the Pipeline Build we want. For example, saying that Pipeline Components of the same framework should be put in the same environment. Or that a Channel should be configured in a specific way.  
- Config of runners: The ontology supports that Runners can have Configs, but currently there is no way for the user to define the Config of Runners. This is because Runners do not make part of the Pipeline Definition. 
- The shape of data inputs and outputs can be defined via NodeShapes, however currently the ontology does not include Data entities. I honestly am not sure how a Data entity could be defined for pipelines that stream data, unless the stream is seen as a data dump that is processed gradually over time.  
<br>

# 3. Pipeline Generator Specification

## 3.1. High-level overview

This is a high-level overview of a Pipeline Generator proof-of-concept implementation (yet to be) written in Python. 

1. **Load the Pipeline**
    - *Ontology.ttl*, *ComponentCatalogue.ttl* and *PipelineDefinition.ttl* are loaded into Python and combined to a Graph in RDFlib.
    - If a PipelineDefinition is 'nested' (in that a PipelineStep is carried out by a Pipeline), this nesting needs to be flattened so that the PipelineDefinition only consists of PipelineSteps.
    - IRIs of the following Entities are fetched, which are implicated in the PipelineDefinition (either directly or indirectly): PipelineDefinition, Config, Processors, Runners, UsesAndAssumptions, Dockerfiles
    - Mappers create Python dictionaries based on IRIs.
    - DomainModellers create Domain Objects based on Python dictionaries. Domain Objects also contain the "raw" subgraph plus Python dictionary of the Entity.
<br><br>
2. **Validate the Pipeline Definition**
    - ConfigShapeValidators validate each Config.
    - DataShapeValidators validate that schemas of data inputs and outputs match between subsequent Processors
    - PipelineShapeValidators validate that the flow of data between processors does not validate any assumptions imposed by Pipeline Components.
<br><br>
3. **Enrich the Pipeline.graph with the Pipeline Build**
    - Add *PipelineBuild* to the graph. Add predicate *prov:hadPlan*.
    - Based on all *Dockerfiles*, decide which *PipelineComponentInstances* may be included in the same *Environment*. 
    - Add *PipelineComponentInstance*, *Environment* and *Dockerfile* to the graph. Add predicates *prov:AtLocation*, *tc:isInstanceOf*, *ssn:hasSubsystem*, *prov:actedOnBehalfOf*, *tc:implementsStep*, *sosa:isHostedBy*
    - Add *Channels* to the graph. In some cases this may require inserting extra "glue"- *PipelineComponentInstances* as bridges between *Environments*. Add predicates *tc:writesTo* and *tc:readsFrom*
    - Compile ConfigInstances based on the Config of the Steps that each ProcessorInstance implements. The exact layout and logic of ConfigInstances are framework-specific and hence this compilation of ConfigInstances has to be hard-coded per framework.
<br><br>
4. **Validate the Pipeline Build**
    - Attempt to validate any remaining *NodeShapes* and execute *SPARQL-queries* on violation iteratively. 
<br><br>
5. **Return results**
    - Return the enriched Pipeline.graph. It provides information on: The pipeline that was attempted to build (Pipeline Definition), the result of that attempt (Pipeline Build) and the resources that were used for that (PipelineComponents)
    - Return the Dockerfiles and the ConfigInstances. These can be directly used by the user to instantiate and run the pipeline.
<br><br>

## 3.2. Additional Details
### 3.2.1. Mapping
Entities in RDF graphs are converted to Python objects in two steps. They are first converted to Python dictionaries and only afterwards to Python Domain Objects (classes). This has two advantages: 1) Should the ontology change, it is sufficient to redefine how Python dictionaries map onto Domain Objects. 2) Since PipelineComponents can stem from different frameworks, and different frameworks may require different additional attributes, it is impossible to define all possible attributes for Domain Objects. However it IS possible to convert subgraphs to Python dictionaries exhaustively (more on that below). This means that the Domain Objects can have all attributes that are expected by the toolchain-ontology, and any additional attributes are included in the dict-attribute of the respective Domain Object. This hence allows to still access any additional attributes natively in Python, even if they are not defined in the toolchain-ontology.

Python dictionaries are created based on IRIs in the following way: 
- A recursive function follows all predicates and objects stemming from the root IRI recursively. The extracted triples are called "subgraph". It is possible to define which predicates not to follow.
- The subgraph is serialized as JSON-LD
- A frame is applied to the JSON-LD, with the settings *embed: always*, *explicit: false* and *id: root_IRI* (referring to the IRI of the extracted entity). This creates a nested JSON-object, whereby each predicate is a key and each object the value. This JSON-LD object is compacted (shortening prefixes) and returned
- The only precondition for this to work properly is that the subgraph should be tree-like (acyclic). Conversion to JSON-LD will not fail if the graph is cyclic (since JSON-LD is just a serialization of RDF), however it will not be possible to properly express all triples as a single nested dictionary
<br><br> 

### 3.2.2. Config Validation
ConfigValidation means that the Config of a PipelineStep that a ProcessorInstance implements has to be validated by the corresponding NodeShape of the Processor that is instantiated. If the Config is "embedded" (aka serialized as RDF), this is straightforward, because then the Config schema can be expressed in SHACL.
<br><br>
However it is a different matter if the Config is "literal" or "external" and in JSON- or yaml-format, for example. We should provide different formats for defining a Config other than RDF, because users will most likely want to express Configs in the serialization format that is expected at runtime, or link to an external Config that already exists. 
<br><br>
If we allow Config in different formats, we should also allow Config schemas to be written in different formats. For example, a JSON-schema could validate JSON and yaml-Configs. In this case, th Config Schema cannot be expressed as NodeShape, but it can still be attached to osw:UsesAndAssumptions. A PipelineGenerator should then support validation by JSON-schemas and SHACL-shapes. See [LinkML](https://linkml.io/linkml/intro/overview.html) for related work. 
<br><br>

### 3.2.3. DataShape Validation
DataShape Validation has to evaluate whether input shapes of Processors match with the output shape of the Processor preceeding it. This is a complex problem and to my knowledge, not much research has been done. It requires to evaluate whether two given SHACL-shapes are mutually exclusive or not.
<br><br>
For two SHACL shapes to be mutually exclusive, it must be impossible that their target can validate both at the same time. For simple constraints, it is "easy" to figure that out, for example when expected data types, formats or value ranges do not match. However for complex SHACL shapes this requires a more involved approach.
<br><br>
In my brief desk research, I stumbled upon [a java-package, which converts SHACL into a "first order logic"- language](https://github.com/paolo7/shacl2fol). I am not sure whether this allows you to check whether the corresponding statements are mutually exclusive, but this seems like a promising starting point. 

### 3.2.4. Dockerfiles
Dockerfiles of Environments are compiled based on the Dockerfiles attached to each PipelineComponent via "osw:hasInstallationInstructions". The compiled Dockerfile should be a [common denominator](https://en.wikipedia.org/wiki/Lowest_common_denominator#Colloquial_usage) of all PipelineComponnts included in the Environment. 
<br><br>
This means that Dockerfiles of each Pipeline Component should be self-sufficient, meaning that a Pipeline Component should be instantiable based on its Dockerfile alone. An exception are dependencies on runners, which should be expressed via osw:hasDependency instead. However, given that a common "merged" Dockerfile is compiled for the Pipeline Build, a PipelineGenerator should be tolerant to redundancy across Dockerfiles. 
<br><br>
Compiling a common Dockerfile based on several Dockerfiles is again [a complex problem](https://stackoverflow.com/questions/39626579/is-there-a-way-to-combine-docker-images-into-1-container) that needs further research in order to evaluate its feasibility. If merge-conflicts arise, a user may be prompted to resolve these manually, or PipelineComponents may be split into different Environments instead. 
<br><br>
If a Pipeline Build is hosted on more than one Enviroment, not only the Dockerfiles, but also a Docker Compose file needs to be compiled. This Docker Compose file needs to include build instructions for each Dockerfile of each Environment. 
<br><br>
It may also be required to define any dependencies and health checks for containers in a Dockerfile. This will add another layer of complexity and needs to be figured out when we get there. 
<br><br>

### 3.2.5. Returning Results 
In an ideal world, the resulting Config Files and Dockerfiles should be sufficient to run a PipelineBuild. Likely, this will not be the case, because I am not confident we can build a PipelineGenerator that is smart enough to handle all possible complexities and edge cases with grace. Nevertheless, the PipelineGenerator should be seen and used as *assistant* to building complex cross-framework pipelines. It should validate Pipelines automatically, provide clear human-readible feedback where validation fails, and ConfigInstances and Dockerfiles should require only minor tweaking to function properly. This alone would already be a huge step forward, because frameworks today provide no online feedback when writing Config-files and Dockerfiles by hand. 
<br><br>

## 3.3. Frontend
Akin to the [model-view-controller pattern](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), a frontend should provide a user interface that provides a visual representation of Domain Model Objects and should allow interacting with them. 

Specifically, the Pipeline Definition should be visually represented as Nodes and Edges, and it should be possible to interactively change both the Config and the assigned Processor of a Node (representing a Pipeline Step). Edges should be changable as well. These user inputs should automatically translate to updates in the underlying model.

The list of Processors that can be selected should be populated by the Component Catalogue. Ideally, new user inputs should prompt validation (in the background) on the fly, so that timely feedback can be provided and suggestions can be made. For example, assigning a new Processor to a step may either invalidate a Pipeline Definition, or an associated Config, or limit which Processors can be used for subsequent Pipeline Steps.

It should be possible to add new Component Catalogues to the model by providing a link. The list of selectable Processors should be updated accordingly. 

Configs should be presented as forms by automatically converting their ConfigSchemas to web forms. If Configs are written as literals, a multiline-textfield can be presented instead. 

Multiple views should be provided. At the very least, I would provide both a user interface as a view, such as described above, and a more traditional text-view, which allows to write Pipeline Definitions in RDF directly. Examples are [RDF playground](https://rdfplayground.dcc.uchile.cl/) or [SHACL playground](https://shacl-playground.zazuko.com/). These views may be intermixed or changed on click.

There should be a console output which displays the sh:message for validations that failed. Ideally, the target of the corresponding NodeShape should be highlighted. 

It should be possible to trigger the Pipeline Generator in the frontend. The results of generating the Pipeline Build may be represented in the view, allowing editing of the newly generated entities. 
<br><br>

## 3.4. Monitoring 
As of now, the Pipeline Generator will only generate the files needed to run a pipeline, but not initiate the running of the pipeline itself. However an extension of this architecture could make this possible. Since the Pipeline Generator / backend is written in Python, the docker library in Python can be used to build and start docker containers and hence the pipeline directly. 

Since the frontend is already used to visualize the pipeline, it may also be used to visualize the monitoring of the pipeline once it runs. For this purpose, it is needed to fetch information from the Docker API: It will provide information basic monitoring information, like container health, logs (stdout/stderr), resource usage. This can indicate whether all Environments and initiated correctly. Fetching stdout and stderr in this way is convenient because it automatically fetches all logs that each Pipeline Component produces, no matter the framework it runs on. 

Ideally we would also want some information on the data throughput. This could be done by having the "glue"-Processors, i.e. the bridges that are interjected between docker containers to provide cross-container communication, provide an API for this. At the very least, it would allow capturing whether data goes in and out of each container, and hence whether the pipeline is running. 


