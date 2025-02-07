# README

This project illustrates how to deploy the Spring Pet Clinic Angular application
and Spring Pet Clinic Spring Boot REST application to the Azure container runtime
of your choice.

## Getting Started

### Prerequisites

1. Java 21
1. Maven

### Installation

To get started we need to apply the necessary patches to the Spring Pet Clinic 
submodules by patch their sources so you can easily deploy it to the container
runtime of your choice. To do so use the command line below:

```shell
  git apply containers.patch
```

### Container runtime

Pick your desired container runtime below:

1. [Azure Container Apps](ACA.md)
