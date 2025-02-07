# Building and deploying to Azure Container Apps

This guide provides step-by-step instructions to deploy the Angular front-end and the Spring Boot REST back-end of the Spring Petclinic application to Azure Container Apps.

## Prerequisites

1. **Azure CLI**: Ensure you have the Azure CLI installed. You can download it from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
2. **Docker**: Ensure you have Docker installed. You can download it from [here](https://docs.docker.com/get-docker/).
3. **Paketo `pack` binary**: Ensure you have the Paketo `pack` binary installed. You can download it from [here](https://buildpacks.io/docs/tools/pack/).

## Step 1: Build the Angular Front-end Docker image

1. Navigate to the Angular project directory:
    ```bash
    cd frontend
    ```

2. Build the Docker image using Maven:
    ```bash
    mvn clean install -Pdocker
    ```

> **Note:** The `docker` Maven profile uses the `pack` binary and the `paketobuildpacks/builder-jammy-base` builder with the `paketo-buildpacks/nginx` buildpack to produce the Docker image.

## Step 2: Build the Spring Boot Back-end Docker image

1. Navigate to the Spring Boot project directory:
    ```bash
    cd ../backend
    ```

2. Build the Spring Boot application and Docker image using Maven:
    ```bash
    mvn clean install -Pdocker
    ```

> **Note:** The `docker` Maven profile uses the `pack` binary and the `paketobuildpacks/builder-jammy-base` builder and the `target/${project.build.finalName}.jar` JAR file to produce the Docker image.

## Step 3: Create Azure Container Registry

1. Log in to Azure:
    ```bash
    az login
    ```

2. Create a resource group:
    ```bash
    az group create --name myResourceGroup --location eastus
    ```

3. Create an Azure Container Registry (ACR):
    ```bash
    az acr create --resource-group myResourceGroup --name myAcrRegistry --sku Basic
    ```

4. Log in to the ACR:
    ```bash
    az acr login --name myAcrRegistry
    ```

## Step 4: Tag the local Docker images and push them to the Azure Container Registry

1. Tag and push the Angular frontend Docker image:
    ```bash
    docker tag spring-petclinic-angular:latest myacrregistry.azurecr.io
    docker push myacrregistry.azurecr.io/spring-petclinic-angular:latest
   ```

2. Tag and push the Spring Boot REST backed Docker image:
    ```bash
    docker tag spring-backend:latest myacrregistry.azurecr.io/spring-backend:latest
    docker push myacrregistry.azurecr.io/spring-backend:latest
    ```

## Step 5: Deploy to Azure Container Apps

1. Enable the Azure Container Apps extension:
    ```bash
    az extension add --name containerapp --upgrade
    ```

2. Create an Azure Container Apps environment:
    ```bash
    az containerapp env create --name myContainerAppEnv --resource-group myResourceGroup --location eastus
    ```

3. Deploy the Spring Boot back-end (private):
    ```bash
    az containerapp create --name spring-backend --resource-group myResourceGroup --environment myContainerAppEnv --image myacrregistry.azurecr.io/spring-backend:latest --target-port 8080 --ingress 'internal'
    ```

4. Deploy the Angular front-end:
    ```bash
    az containerapp create --name angular-frontend --resource-group myResourceGroup --environment myContainerAppEnv --image myacrregistry.azurecr.io/angular-frontend:latest --target-port 80 --ingress 'external' --env-vars REST_API_URL=http://spring-backend:8080/petclinic/api/
    ```

## Conclusion

You have successfully deployed the Angular front-end and the Spring Boot REST back-end to Azure Container Apps. The back-end is private and only accessible by the front-end.

To get the URL for the front-end, use the following Azure CLI command:
```bash
az containerapp show --name angular-frontend --resource-group myResourceGroup --query properties.configuration.ingress.fqdn
```
