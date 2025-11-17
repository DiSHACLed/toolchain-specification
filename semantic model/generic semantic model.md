# Brief
- *Description*: Generic semantic model to describe pipelines. 
- *Intention*: discoverability, replication, modular design, separation of concerns, ease of use 

# Entities
### Pipeline
In its essence, a pipeline is a directed graph of modular processing units, who generate a set of data given another set of data as input. Hence, a pipeline consists of one or more ProcessorInstances who are linked to each other through DataFlows. ProcessorInstances can be configured through Configs.

### Processor
A Processor is any modular unit that can make part of a pipeline through DataFlows with other Processors. As a class it differs from a ProcessorInstance in that it is a blueprint intended for reuse. To allow easy reuse, the Processor has to provide enough information in the graph that it can be compiled based on the provided information.

### ProcessorInstance
A ProcessorInstance is an instance of a Processor in a specific Pipeline, and hence expected to form DataFlows with other ProcessorInstances in the Pipeline. It also comes with a Config so that the exact behavior of the Processor in the Pipeline can be inferred.

### DataFlow
A DataFlow links two or more ProcessorInstances together. In other words, it describes how ProcessorInstances depend on each other. You can picture ProcessorInstances as nodes, and DataFlows as edges in a graph. One ProcessorInstance may receive input from another, this is expressed via flowTo and flowFrom -predicates. The DataFlow entity allows to characterize this interdependence further, for example by the port, the protocol, the mode, etc. of the data transfer.

### DataSchema
A Processor transforms data in predicatable ways. This can be expressed as DataSchema, which expresses the data before transformation (when linked via inputDataSchema) and after transformation (when linked via outputDataSchema). The shape of the data should be expressed in a similar way as it is done for WP1. 

### Implementer
An Implementer is any modular piece of software needed to run a pipeline, but not responsible for transforming and forwarding data. This class hence allows to express which software is needed to run a particular processor. It is also possible that another implementer is needed to run an implementer. 

### ImplementerInstance 
Implementers are instanced to run a particular pipeline. The required implementers can either be explicitly expressed by the user, or can be inferred and compiled (semi-)automatically.

### PipelineComponent
This is a superclass of Implementers and Processors, i.e. any modular and reusable piece of software needed to run a pipeline. PipelineComponents have a ConfigSchema, which defines the parameters the PipelineComponent accepts. It also links to resources needed to implement a pipeline, such as the Dockerfile, Documentation and Constraints.

### ConfigSchema
PipelineComponents permit a set of configuration values, hence a ConfigSchema is essentially a schema of the configuration values that a PipelineComponent can accept. ConfigSchema conceptually differs from DataSchema in that configuration values are typically not transformed while running the pipeline (in contrast to data) but rather define the transformation behavior itself. 

### Config 
When a PipelineComponent is instanced, it is expected that its Config corresponds to the ConfigSchema that the PipelineComponent permits. The difference between Config and ConfigSchema is hence that ConfigSchema is a schema and a Config is a set of concrete values used while running the pipeline. 

### Dockerfile
To allow building a pipeline based on the RDF graph describing the pipeline alone, it is necessary to describe how the PipelineComponent can be build from scratch. This is the purpose of the Dockerfile entity. It is a concrete installation instruction in the form of a dockerfile. 

### Documentation 
This is self-explanatory, it should link to the documentation of the PipelineComponent.


### Constraint
Modular pieces of software, i.e. PipelineComponents come with Constraints to execute properly in a pipeline. For example, the Implementer "Linked Data Interactions Orchestrator" has the important constraint that each instance of a Processor that it runs can only be linked to each other via DataFlows in a strictly sequential order (because LDIO does not allow branching within its framework). As another example, instantiating a “HTTP Forwarder” Processor to a Pipeline could entail validating its Constraint that it is linked to two other Processors via HTTP, hence effectively forwarding data via HTTP. In short, Constraint is an important entity which can be used to describe logical constraints that PipelineComponents introduce when being instantiated in a pipeline.

### Prov-O:Activity
Allows to track the provenance of pipeline runs, see https://www.w3.org/TR/prov-o/#Activity. A Prov-O:Activity could also wasInformedBy a ProcessorInstance, in which case provencance can be tracked more granularly. 

### Data
Should ideally be linked to a DCAT:Dataset, so that the provenance of Data can be understood in terms of the Pipelines that generated a dataset based on one or more other datasets. 

### Project
Probably one wants to describe a Project that used / generated Data through a Pipeline by declaring the people involved, the purpose, the homepage, publications, etc. 


# Future Directions 
- ConfigSchema: We discussed that configurations can become very complex and that it is hence likely not feasible nor desirable to define all configuration schema’s exhaustively with this entity. It seems a more feasible approach to sometimes only specify that a parameter expects a config-file of a specific type, for example yaml or json. We should look at the common workflow language (cwl) which is very mature in describing configurations. 
- In this model, pipelines are only described as concrete instances, not as reusable templates. In my view, a pipeline automatically becomes reusable by simply changing the used data and potentially some configuration values. But some people may see that differently and may want distinction of class and instance for pipelines. 
- Once we are happy with our generic semantic model, we should check which entities we can map onto already existing ontologies. The more we can reuse the better!
- It remains to be tested whether describing a pipeline in such a way really provides enough information to (semi-) automatically compile a corresponding pipeline. 
- We may want to be able to express the state of a pipeline run while it is still ongoing. This remains to be discussed, because in the current scope the rdf graph is only used to compile a pipeline build, but not used to monitor the pipeline progress. 
- We may not always want to build each PipelineComponent, some PipelineComponents may already be running. We have to think more about how to express this and how this would work in practice. 


