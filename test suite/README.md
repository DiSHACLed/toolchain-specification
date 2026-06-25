# Test suite

This document defines test cases to validate the [spec](https://github.com/thcarsten/toolchain-specification/tree/main/semantic%20model) compliance and at runtime validity of pipelines with SHACL shapes. These test cases will help developers to better understand processor and pipeline descriptions and likely increase the adoption of the [Generic Toolchain Specification](https://github.com/thcarsten/toolchain-specification/tree/main/semantic%20model).

Test cases are defined using different categories of SHACL Shapes as input.

## 1. Specification Shapes

Shapes that verify the entities related to the `tcs:PipelineDefinition` conforms to the application profile of the Generic Toolchain Specification.

**Examples:**
- A Pipeline Definition must have at least one `tcs:InstancePipelineComponent`.
- Required metadata fields (e.g. label, description) must be present.

## 2. Pipeline consistency Shapes

Shapes that check whether a Pipeline Definition violates structural or platform constraints.

**Examples:**
- Components that run with Linked Data Interactions (LDIO) framework do not allow branching.
- An Instance Pipeline Component must be preceded by another Instance Pipeline Component.
- In RDF Connect framework, some components require an "input channel" (a previous step with an output channel):
    + e.g. a shacl processor service (like below) requires an input channel
        https://github.com/rdf-connect/shacl-processor-ts
    + e.g. an http fetcher does not require an input channel (it just requires an url to be configured)
        https://github.com/rdf-connect/http-utils-processor-ts
+ In RDF Connect framework, some components don't have an output channel (e.g. writing to a file; serving something with http)

## 3. Configuration Shapes (`configShape`)

Shapes that validate the configuration of the entities in the pipeline:

**Examples:**
- `tcs:InstancePipelineComponent` provides all required configuration properties of its related `tcs:PipelineComponent` and the properties have valid values.
- the `tcs:DockerContainer` provides all required configuration properties for running in Docker

## 4. Build Shapes (`buildShape`) *(out of scope)*

Shapes that validate the compiled `tcs:PipelineBuild` produced by the Pipeline Generator.

---

## 5. Input/Output consistency Shapes (`inputShape` / `outputShape`) *(out of scope)*

Shapes that describe and validate the input and output contracts of `tcs:InstancePipelineComponent`s.

Testing the input/output consistency requires runtime or algorithmic evaluation:

- For each connection between `tcs:InstancePipelineComponent`, verify that the output shape of the upstream component is compatible with the input shape of the downstream component.
- Uses the **shape matching algorithm**.
- **Current limitation:** mismatch between SPARQL-based and SHACL-based shape representations → needs alignment before this can be fully implemented.
- **Opportunity** For some components, we could flag and parameterize on the `tcs:PipelineComponent` that the input/output shape depend on its configuration:
E.g. in a SHACL processor component (https://github.com/rdf-connect/shacl-processor-ts), the outputShape will exactly be the SHACL shapes you have configured it with. To be checked whether and how this can be added to the spec.