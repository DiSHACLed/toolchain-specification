# Introduction

Welcome to the Github on **interoperable tooling**, which is the second objective of the [DiSHACLed](https://github.com/DiSHACLed) project. The goal of this part of the project is to specify a **Generic Toolchain Architecture**, which allows the discovery and reuse of processors as part of ETL data pipelines in a semi-automatic fashion.

&nbsp;

# Framework requirements
- Performs ETL / ELT
- Uses reusable (modular) processors
- Can combine processors across different runtime environments 
- Pipelines can be validated before execution
- Processors are discoverable based on their described functionality (e.g. as SHACL shapes)
- Can process both data streams and batches
- Strong support for linked data (rdf)
- Fully open-source
- Can be deployed anywhere (locally and cloud)
- to-be-discussed: Can handle branching pipeline steps (conditionals, for example different processors per file-format)

&nbsp;

# RDF-connect: A conceptual overview

![Generic Toolchain Architecture](https://github.com/user-attachments/assets/1ece8d0a-bcb0-4875-9308-19710def05eb)

[RDF-connect](https://github.com/rdf-connect/) will be the reference framework for the Generic Toolchain Architecture. In contrast to all other comparable frameworks, it fulfills all of the suggested requirements. 

Conceptual overview of RDF-connect: Any **pipeline** that extracts, transforms and loads (ETL) data may be understood as a chain of subsequent **processors**. A processor is a component within a pipeline that performs one well-defined job by transforming an input into an output. Processors are typically language-specific (since they are being written in languages such as Python or Javascript) and can hence require different runtime environments. It is the goal of the Generic Toolchain Architecture to enable the linking of processors across different runtime environments. For this purpose, data is passed over **channels**, which can cross runtime environments by being language-agnostic (such as html or kafka). For this purpose, **connectors** read and write data to specific channels from within a specific runtime environment. For example, a connector in Python may allow writing data in a Python runtime environment to html, so that a connector in Javascript can read the same data from the html-channel, allowing a Javascript processor to transform the data further. 

A pipeline can be described on an abstract, high level as a configuration file (such as an rdf-file), which merely states the order and configuration of subsequent processors and channels. A processor can be described high-level as well, by describing the expected input arguments as SHACL shapes. A **runner** is code which effectively initializes and executes processors in a specific runtime environment. An **orchestrator** mediates between different runners and hence ensures that the pipeline can be run as a whole: It interprets the pipeline configuration file, instantiates the runners, forwards processor configurations, and facilitates communication over channels. A **pipeline validator** is executed before the orchestrator and checks whether a pipeline may be executed successfully. It does so by checking whether the configuration of processors in the configuration file satisfy the constraints of each processor (such as expected input arguments) as specified in their SHACL shapes.

Such Generic Toolchain Architecture allows a separation of concerns in that the functionality of the pipeline and its processors is described independently of the code responsible for execution. This has several advantages: Pipelines can be *validated* before being executed. Pipelines may be *generated* (semi-)automatically, by identifying which processors may be linked to achieve a certain outcome. Processors become *discoverable* and hence *reusable* by searching for SHACL-shapes of processors that meet certain constraints. One goal of this project is to extend the description of processors as SHACL shapes to aspects beyond input arguments, to further enhance discoverability of processors. 

&nbsp;

# Related frameworks

| framework | data streaming | modular processors | multi-language support | pipeline configuration as file | processor description / requirements as file | branching pipeline | 
| ----- | ----- | ----- | ----- | ----- | ----- |  ----- |
|[RDF-connect](https://github.com/rdf-connect/)| ✅ | ✅ | ✅ | ✅ (rdf) | ✅ | ❔ |
|[Linked Data Interactions Orchestrator](https://informatievlaanderen.github.io/VSDS-Tech-Docs/)| ✅ | ✅ | ❌(Java) | ✅ (yaml, automatic pipeline building via docker compose) | ❌ | 🟡(linear pipelines, but multiple pipelines can be linked as branches) |
|[Apache Nifi](https://nifi.apache.org/nifi-docs/getting-started.html)| ✅ | ✅ | ❌(Java) | ✅ (xml, very verbose, can be shared and reused) | ❔ | ✅ |
| [Semantic.works](https://abb-vlaanderen.gitbook.io/abb/development/architecture/semantic-works-application-framework) |❌(batch-based, limited by SPARQL-queries to virtuoso) | ✅ | ❌(Javascript) | ✅ (json) | ❌ | 🟡(pipelines are currently linear, but is generally possible) |
|[Barnard59](https://data-centric.zazuko.com/docs/workflows/)| ✅ | ✅ | 🟡(Javascript-based, other languages via wrappers) | ✅ (rdf, has a pipeline validator) | ❌ | ❔ |
|[LinkedPipes ETL](https://etl.linkedpipes.com/)| ❌(batch-based) | ✅ | ❌(JVM) | ✅ (JSON-LD, pipeline fragments can be shared and reused) | ✅ | ✅ |
|[Knime](https://www.knime.com/getting-started-guide)|🟡(primarily batch-based, some streaming support)| ✅ | ✅ | ❌ | ❔ (has a processor database called Community Hub) | ✅ |

*Note.* All listed frameworks perform ETL/ELT, are open-source, should be deployable anywhere, and have support for linked data.

&nbsp;

| framework | description |
| ----- | ----- |
| Linked Data Interactions Orchestrator | The pipeline configuration is specified as a yaml-file and posted to the LDIO via a HTTP-post request. The corresponding Java modules and the LDIO itself are build via docker compose, so that the correct environment can be build before the pipeline is added to the LDIO and run. | 
| Apache Nifi | Pipelines are build with a GUI. Templates can be exported for reuse of entire pipelines or parts of it. No multi-language support, with the exception for Jupyter (Python wrapper for Java). Good for branching pipelines, streaming, queueing and error-handling. |
| Semantic.works | It is a framework of modular microservices for building web applications. These web apps are user-facing, meaning that microservices are triggered and respond to user input: When clicking ‘login’ on a page, or entering a command in a CLI, a microservice responds by reading and writing rdf-data to a virtuoso triplestore. Microservices may hence fetch data to show the user, or store new user information in the triplestore. Semantic.works was hence originally not conceived as ETL-framework. However, it is possible to set up data processing pipelines via the ‘job-controller-service’ microservice. Here, a pipeline can be specified as a chain of javascript functions. These can either be scheduled (as CRON-job), directly triggered by the user (by having a job-frontend webpage), or triggered by changes in the triplestore (such as when a new login is registered). Semantic.works also made efforts to allow separation between the microservice-infrastructure acting on the data (e.g. the pipeline) and the microservice-infrastructure presenting the data to the user (e.g. a webpage). For this, they added delta-producer and delta-consumer mircoservices to semantic.works. A delta-producer microservice produces changes to a database, and ‘announces’ these changes as delta-files ('delta' in the sense of ‘changes since last state’). The 'delta-consumer' can then mirror the database of the delta-producer by synchronizing its database via those delta-files. This allows to set up two databases, one where the changes are produced (i.e. via a pipeline) and one where the changes are mirrored, but different microservices can act on it (i.e. when building a webpage for data inspection).| 
| Bardnard59 | barnard59  is a modular system that is very similar to RDF-connect conceptually. Between processors, (Node.js) streams are used as channels. barnard59 is written in JavaScript and is using the Node.js for execution. Pipelines are defined as RDF and interpreted and executed by the barnard59 package. |
| LinkedPipes ETL | Pipelines are build with a GUI. They can also be created via a SPARQL construct-query, since they are defined as rdf-files. Very similar to Knime in that regard, although it seems less extensive.|
| Knime | Pipelines are build with a GUI. The processors for the pipeline can be downloaded from the Knime Community Hub or written yourself. Processors in different languages are supported.|

&nbsp;

# Further reading

### RDF-connect
- RDF-connect paper: https://biblio.ugent.be/publication/01J84X94FCJTDFAAH6QZ2DQXAD
- RDF-connect example: https://github.com/rdf-connect/RDF-Connect-RINF-LDES
- Theory: https://the-connector-architecture.github.io/site/docs/1_Home
- More on the orchestrator: https://github.com/rdf-connect/orchestrator/tree/main

### Linked Data Interactions Orchestrator
- https://github.com/Informatievlaanderen/VSDS-Linked-Data-Interactions/tree/main/ldi-core

### Apache Nifi
- Good tutorial: https://github.com/LeapBeyond/nifi-tutorials/blob/master/introduction/README.md

### Semantic.works
- Video introduction: https://abb-vlaanderen.gitbook.io/abb/development/architecture/semantic-works-application-framework
- https://github.com/lblod/delta-tutorial
- https://github.com/lblod/app-lblod-harvester/
- https://github.com/lblod/delta-producer-background-jobs-initiator/tree/master
- https://github.com/lblod/scheduled-job-controller-service
- https://github.com/lblod/app-verenigingen-loket-harvester/tree/master
- https://github.com/lblod/delta-producer-background-jobs-initiator
- https://github.com/lblod/job-controller-service


### Bardnard59
- https://data-centric.zazuko.com/docs/workflows/tutorial/first-pipeline/


### LinkedPipes ETL
- https://docs.google.com/presentation/d/1UgEc2k2EuvHT9CPEtNKN1DUDW2iffaWtJ9PVEN56XMM/edit#slide=id.g202cd5e51c2c43eb_147

### Knime
- Python Support: https://www.knime.com/blog/4-steps-for-your-python-team-to-develop-knime-nodes
- More on Knime cross-language support: https://www.knime.com/blog/not-all-low-code-is-created-equal











