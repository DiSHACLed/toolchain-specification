# Brief
- *Description*: Generic semantic model to describe pipelines. 
- *Intention*: discoverability, replication, modular design, separation of concerns, ease of use 

# Entities
### Pipeline
In its essence, a pipeline is a directed graph of modular processing units, who generate a set of data given another set of data as input. Hence, a pipeline consists of one or more PipelineComponentInstances who are linked to each other through Relationships. PipelineComponentInstances can be configured through ParameterValues.

### PipelineComponent
A PipelineComponent is any modular unit that can make part of a pipeline through Relationships with other PipelineComponents. As a class it differs from a PipelineComponentInstance in that it is a blueprint intended for reuse. To allow easy reuse, the PipelineComponentInstance has to provide enough information in the graph that it can be compiled based on the provided information. 

### PipelineComponentInstance
A PipelineComponentInstance is an instance of a PipelineComponent in a specific Pipeline, and hence expected to form Relationships with other PipelineComponentInstances in the Pipeline. It also comes with a configuration of ParameterValues so that the exact behavior of the PipelineComponent in the Pipeline can be inferred.

### Relationship
A Relationship links two or more PipelineComponentInstances together. In other words, it describes how PipelineComponentInstances depend on each other. For example one PipelineComponentInstance may receive input from another. The Relationship entity allows to characterize this interdependence further, for example by specifying the direction of the relationship, the port, the protocol, etc. The definition of Relationships between PipelineComponentInstances is kept intentionally broad to also allow modelling of aspects of pipelines beyond dataflow, such as montoring or orchestration, at least in theory. 

### Parameter
PipelineComponents permit a set of Parameters, hence a parameter is essentially a schema of the configuration values that a PipelineComponent can accept.

### ParameterValue 
When a PipelineComponentInstance is instanced, it is expected that its configuration of ParameterValues corresponds to the Parameters that the PipelineComponent permits. The difference between Parameter and ParameterValue is hence that Parameter is a schema and ParameterValue a concrete value used while running the pipeline. 

### Codebase
To allow building a pipeline based on the RDF graph describing the pipeline alone, it is necessary to link the physical locations of where the codebase and pipeline artifacts can be fetched. This is the purpose of the Codebase entity. At the very least, it should link to the storage of the PipelineComponent, ideally it also should provide references to the documentation, installation commands, etc. 

### Dependency 
The Dependency entity lists all dependencies that are required to run a specific PipelineComponent, like gradle, python, docker etc. Think of it as corresponding to a requirements.txt- file or a package.json.

### Framework
A PipelineComponent should link to the Framework it belongs to in case this has important implications for the building of the pipeline. This allows for example to link documentation of a framework to the Framework entity, or to link Assumptions on the level of a Framework to that entity.

### Assumption
Assumptions allow to explicitly define specificities which need to be respected for the pipeline to execute correctly. Assumptions can be made on different levels, and hence Assumptions can be linked to Frameworks, PipelineComponents and Pipelines. For example, instantiating a “HTTP Forwarder” PipelineComponent to a Pipeline could entail validating its Assumption that it is linked to two other PipelineComponents via HTTP, hence effectively forwarding data via HTTP. Frameworks can make different sets of assumptions to accommodate the differences in conceptualizations between frameworks. For example, a Framework entity for LDIO may make the Assumption that all instances of PipelineComponents are linked to each other in a strictly serial way (no branching). A Framework entity of Semantic.Works may declare that all corresponding PipelineComponent instances need to be linked to a central triple store. Likewise, a set of Assumptions can be made about Pipelines, for example each PipelineComponentInstance should have at least one Relationship, each mandatory Parameter should be specified with a ParameterValue, etc. 

### Prov-O:Activity
Allows to track the provenance of pipeline runs, see https://www.w3.org/TR/prov-o/#Activity. 

### Data
Should ideally be linked to a DCAT:Dataset, so that the provenance of Data can be understood in terms of the Pipelines that generated a dataset based on one or more other datasets. 

# Future Directions 
- We want to be able to describe the expected effect that a PipelineComponentInstance has on the data it processes. Currently this is not possible because Data is not linked to PipelineComponentInstances, only to Pipelines as a whole.
- Parameter: We discussed that configurations can become very complex and that it is hence likely not feasible nor desirable to define all configuration schema’s exhaustively with this entity. It seems a more feasible approach to sometimes only specify that a parameter expects a config-file of a specific type, for example yaml or json. 
- We may want to be able to express the state of a pipeline run while it is still ongoing. This remains to be discussed, because in the current scope the rdf graph is only used to compile a pipeline build, but not used to monitor the pipeline progress. 
- In this model, pipelines are only described as concrete instances, not as reusable templates. In my view, q pipeline automatically becomes reusable by simply changing the used data and potentially some parameter values. But some people may see that differently and may want distinction of class and instance for pipelines. 
- I am not sure yet how broad the description of pipelines should go. Do we purely want to focus on describing the dataflow or do we also want to describe other aspects like orchestration, monitoring and architecture? For example, in the case of RDFConnect, do we only want to model the processors and merely mention runners and the orchestrator via the Dependency Entity? Or do we want to explicitly model that processors have relationships with runners, which in turn have relationships with the orchestrator? 
- The RDFConnect model has “Execution Context” as entity, corresponding to runtime environments like Python or Javascript. Do we need that as well or is it sufficient to mention these as Dependency of a PipelineComponent?
- Once we are happy with our generic semantic model, we should check which entities we can map onto already existing ontologies. The more we can reuse the better!
- There is a lot to learn from the common workflow language (CWL). Its idea is VERY similar, based on command line tools that are  described in standardized workflow files. This allows CWL runners to chain those workflow files together and run them as a sequence. Important for us, workflow files can also describe dependencies and expected in- and outputs. We should have a look at how exactly this works, and see what we can take over. Unfortunately, there is no easy introduction to CWL, the best I could find is here: https://www.youtube.com/watch?v=jfQb1HJWRac&list=PLWhmJecWmTVHjfttXkltdbDYjmK7R7eJw&index=6 (from circa 25:00).
