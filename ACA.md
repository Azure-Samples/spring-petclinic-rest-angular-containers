# Building and Deploying to Azure Container Apps

This guide provides step-by-step instructions to deploy the Angular front-end and the Spring Boot REST back-end of the Spring Petclinic application to Azure Container Apps.

## Prerequisites

1. **Azure CLI**: Ensure you have the Azure CLI installed. You can download it from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
2. **Docker**: Ensure you have Docker installed. You can download it from [here](https://docs.docker.com/get-docker/).
3. **Paketo `pack` binary**: Ensure you have the Paketo `pack` binary installed. You can download it from [here](https://buildpacks.io/docs/tools/pack/).

## Step 1: Build the Angular Front-end Docker Image

1. Navigate to the Angular project directory:
    ```bash
    cd spring-petclinic-angular
    ```

2. Build the Docker image using Maven:
    ```bash
    mvn clean install -Pdocker
    ```

> **Note:** The `docker` Maven profile uses the `pack` binary and the `paketobuildpacks/builder-jammy-base` builder with the `paketo-buildpacks/nginx` buildpack to produce the Docker image. 

The following options are passed to `pack`:
> - `--buildpack paketo-buildpacks/nginx`: Specifies the buildpack to use for building the image.
> - `--builder paketobuildpacks/builder-jammy-base`: Specifies the builder image to use.
> - `--env PORT=8080`: Sets the environment variable `PORT` to `8080`.
> - `--env BP_NODE_RUN_SCRIPTS=build`: Sets the environment variable `BP_NODE_RUN_SCRIPTS` to `build`.
> - `--env BP_WEB_SERVER=nginx`: Sets the environment variable `BP_WEB_SERVER` to `nginx`.
> - `--env BP_WEB_SERVER_ENABLE_PUSH_STATE=true`: Sets the environment variable `BP_WEB_SERVER_ENABLE_PUSH_STATE` to `true`.
> - `--env BP_NODE_VERSION=22.13.0`: Sets the environment variable `BP_NODE_VERSION` to `22.13.0`.
> - `--env BP_WEB_SERVER_ROOT=dist`: Sets the environment variable `BP_WEB_SERVER_ROOT` to `dist`.

## Step 2: Build the Spring Boot Back-end Docker Image

1. Navigate to the Spring Boot back-end project directory:
    ```bash
    cd ../spring-petclinic-rest
    ```

2. Build the Spring Boot application and Docker image using Maven:
    ```bash
    mvn clean install -Pdocker
    ```

> **Note:** The `docker` Maven profile uses the `pack` binary and the `paketobuildpacks/builder-jammy-base` builder and the `target/${project.build.finalName}.jar` JAR file to produce the Docker image. 

The following options are passed to `pack`:
> - `--builder paketobuildpacks/builder-jammy-base`: Specifies the builder image to use.
> - `--buildpack paketo-buildpacks/microsoft-openjdk`: Specifies the buildpack to use for the JDK.
> - `--buildpack paketo-buildpacks/java`: Specifies the buildpack to use for Java applications.
> - `--env BP_JVM_VERSION=21`: Sets the environment variable `BP_JVM_VERSION` to `21`.
> - `--volume ${project.build.directory}/../src/main/paketo/bindings/application-insights:/platform/bindings/application-insights`: Mounts the Application Insights bindings directory.

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

## Step 4: Tag the Local Docker Images and Push Them to the Azure Container Registry

1. Tag and push the Angular front-end Docker image:
    ```bash
    docker tag spring-petclinic-angular:latest myacrregistry.azurecr.io/spring-petclinic-angular:latest
    docker push myacrregistry.azurecr.io/spring-petclinic-angular:latest
    ```

2. Tag and push the Spring Boot REST back-end Docker image:
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

## Step 6: Enable Application Insights (Optional)

1. Create an Application Insights resource:
    ```bash
    az monitor app-insights component create --app myAppInsights --location eastus --resource-group myResourceGroup
    ```

2. Retrieve the connection string for the Application Insights resource:
    ```bash
    az monitor app-insights component show --app myAppInsights --resource-group myResourceGroup --query connectionString --output tsv
    ```

3. Update the Azure Container Apps to use the Application Insights connection string:
    ```bash
    az containerapp update --name spring-backend --resource-group myResourceGroup --environment myContainerAppEnv --env-vars APPLICATIONINSIGHTS_CONNECTION_STRING=<your_connection_string>
    az containerapp update --name angular-frontend --resource-group myResourceGroup --environment myContainerAppEnv --env-vars APPLICATIONINSIGHTS_CONNECTION_STRING=<your_connection_string>
    ```

## Conclusion

You have successfully deployed the Angular front-end and the Spring Boot REST back-end to Azure Container Apps. The back-end is private and only accessible by the front-end.

To get the URL for the front-end, use the following Azure CLI command:
```bash
az containerapp show --name angular-frontend --resource-group myResourceGroup --query properties.configuration.ingress.fqdn
```

For your reference we are including some screenshots below that show:

1. **Azure Portal - Front-end Deployment**: This image shows the Angular front-end deployment details in the Azure portal.

    ![Azure Portal - Front-end Deployment](./images/frontend-azure-portal.jpg)

2. **Edge Browser - Front-end Application**: This image shows the Angular front-end application running in the Edge browser.

    ![Edge Browser - Front-end Application](./images/frontend-edge-browser.jpg)

3. **Azure Portal - Back-end Deployment**: This image shows the Spring Boot back-end deployment details in the Azure portal.

    ![Azure Portal - Back-end Deployment](./images/backend-azure-portal.jpg)

4. **Swagger API Docs - Back-end**: This image shows the Swagger API documentation for the Spring Boot back-end.

    ![Swagger API Docs - Back-end](./images/backend-swagger-apidocs.jpg)

## Azure DevOps

See [Azure DevOps Pipeline example](azure-pipelines-aca.yml) for a CI/CD pipeline example.

If you are looking to redeploy the two applications after your initial deployment, we recommend using the [AzureContainerApps@1 task](https://learn.microsoft.com/azure/container-apps/azure-pipelines#deploy-an-existing-container-image-to-container-apps) instead of the `AzureCLI@2 task` in the `Deploy` stage in the pipeline example.
