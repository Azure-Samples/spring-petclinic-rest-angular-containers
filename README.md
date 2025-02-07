# README

This project illustrates how to deploy the Spring Pet Clinic Angular application
and Spring Pet Clinic Spring Boot REST application to the Azure container runtime
of your choice.

## Getting Started

### Prerequisites

1. Java 21
1. Maven

### Installation

To get started, we need to apply the necessary patches to the Spring Pet Clinic 
submodules by patching their sources so you can easily deploy them to the container
runtime of your choice. To do so, use the command line below:

```shell
  git apply containers.patch
```

The `containers.patch` file contains the necessary changes to configure the Spring Pet Clinic Angular application and Spring Pet Clinic Spring Boot REST application for deployment to various Azure container runtimes. This includes:

#### Angular Project Changes

- **.gitignore**: Updates to exclude build directories such as `node/` and `target/`.
- **angular.json**: Modifications to add new build configurations for local and ACA environments.
- **pom.xml**: Addition of a Maven POM file in the Angular project to support Maven builds and Docker image creation using Paketo buildpacks.
- **environment.aca.ts**: New environment configuration file for ACA deployments.
- **environment.local.ts**: New environment configuration file for local deployments.

#### Spring Boot REST Project Changes

- **pom.xml**: Updates to include a Docker build profile using the `exec-maven-plugin` for building Docker images with Paketo buildpacks.

### Container runtime

Pick your desired container runtime below:

1. [Azure Container Apps](ACA.md)
