# Todo App Java On Azure

This TodoList app is an Azure Java application. It provides end-to-end CRUD operation to todo list item from front-end AngularJS code. Behind the scene, todo list item data store is [Azure CosmosDB DocumentDB](https://docs.microsoft.com/en-us/azure/cosmos-db/documentdb-introduction). Credentials of Azure Cosmos DB is stored as [Azure Key Vault Secrets](https://docs.microsoft.com/en-us/rest/api/keyvault/about-keys--secrets-and-certificates#BKMK_WorkingWithSecrets). This application uses [Azure CosmosDB DocumentDB Spring Boot Starter](https://github.com/Microsoft/azure-spring-boot/tree/master/azure-starters/azure-documentdb-spring-boot-starter), [Azure Key Vault Secrets Spring Boot Starter](https://github.com/Microsoft/azure-spring-boot/tree/master/azure-starters/azure-keyvault-secrets-spring-boot-starter) and AngularJS to interact with Azure. This sample application provides several deployment options to deploy to Azure, pls see deployment section below. With Azure support in Spring Starters, maven plugins and Eclipse/IntelliJ plugins, Azure Java application development and deployment is effortless now.

- Run ansible from Visual Studio Code  
  - From Terminal.    
    - On Windows, run ansible inside docker.    
    - On Non-windows platform, provide option to run ansible from docker or from local ansible installation.  
  - From [Cloud Shell](https://azure.microsoft.com/en-us/features/cloud-shell/).  
    Run ansible from cloud shell

## TOC

* [Requirements](#requirements)
* [Create Azure Cosmos DB documentDB](#create-azure-cosmos-db-documentdb)
* [Configuration](#configuration)
* [Run it](#run-it)
* [Contribution](#contribution)
* Add new features
    * [Add AAD](https://github.com/Microsoft/todo-app-java-on-azure/tree/aad-start)
    * [Add KeyVault Secrets](https://github.com/Microsoft/todo-app-java-on-azure/tree/keyvault-secrets)
* Deployment
    * [Deploy to Azure Container Service Kubernetes cluster using Maven plugin](./doc/deployment/deploy-to-azure-container-service-using-maven-plugin.md)
    * [Deploy to Azure Web App for Containers using Maven plugin](./doc/deployment/deploy-to-azure-web-app-using-maven-plugin.md)

## Requirements

* [JDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 1.8 and above
* [Maven](https://maven.apache.org/) 3.0 and above

## Create Azure Cosmos DB documentDB

You can follow our steps using [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) to deploy an Azure Cosmos DB documentDB,
or follow [this article](https://docs.microsoft.com/en-us/azure/cosmos-db/create-documentdb-java) to create it from Azure portal.

1. login your Azure CLI, and set your subscription id 
    
    ```bash
    az login
    az account set -s <your-subscription-id>
    ```
1. create an Azure Resource Group, and note your group name

    ```bash
    az group create -n <your-azure-group-name>
    ```

1. create Azure Cosmos DB with DocumentDB kind. Note the `documentEndpoint` field in the response.

   ```bash
   az cosmosdb create --kind GlobalDocumentDB -g <your-azure-group-name> -n <your-azure-documentDB-name>
   ```
   
1. get your Azure Cosmos DB key, get the `primaryMasterKey` of the DocumentDB you just created.

    ```bash
    az cosmosdb list-keys
    ```

## Save Cosmos DB credentials in Azure Key Vault Secrets

You can follow [this article](https://blogs.technet.microsoft.com/kv/2015/06/02/azure-key-vault-step-by-step/) to save credentials of Azure Cosmos DB in Azure Key Vault Secrets. Secrets keys should be:
```txt
- azure-documentdb-key
- azure-documentdb-uri
- azure-documentdb-database
```

Or you could follow our steps using [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) to save above credentials into Azure Key Vault Secrets.


1. create one Azure service principal, and note your `appId` and `password` in output.

```txt
az ad sp create-for-rbac --name <your-azure-service-principal-name>
```

1. create Aazure Key Vault Secrets, note your `vaultUri` in output.

```txt
az keyvault create --name <your-vault-name> --resource-group <your-azure-resource-group-name> --location <your-azure-resource-group-location> --enabled-for-deployment true --enabled-for-disk-encryption true --enabled-for-template-deployment true --sku standard
```

1. set Key Vault Policy, to assign permission to your service principal created at step 1.

```txt
az keyvault set-policy --name <your-vault-name> --secret-permission set get list delete --object-id <your-service-principal-id>
```
`<your-service-principal-id>` is `appId` you already noted at step 1.


1. save credentials to Azure Key Vault Secrets.

```txt
az keyvault secret set --vault-name <your-vault-name> --name azure-documentdb-uri --value <your-azure-cosmosdb-uri>
az keyvault secret set --vault-name <your-vault-name> --name azure-documentdb-key --value <your-azure-cosmosdb-key>
az keyvault secret set --vault-name <your-vault-name> --name azure-documentdb-database --value <your-azure-cosmosdb-database>
```



## Configuration

* Note your Azure service principal `appId`, `password` and Azure Key Vault Uri from above, then modify `src/main/resources/application.properties` file and save it.

``` txt
azure.keyvault.uri=put-your-keyvault-uri-here
azure.keyvault.client-id=put-your-service-principal-appid-here
azure.keyvault.client-key=put-your-service-principal-password-here
``` 

## Run it

1. package the project using `mvn package`
1. Run the project using `java -jar target/todo-app-java-on-azure-1.0-SNAPSHOT.jar`
1. Open `http://localhost:8080` you can see the web pages to show the todo list app

## Clean up

Delete the Azure resources you created by running the following command:

```bash
az group delete -y --no-wait -n <your-resource-group-name>
```

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Useful link
- [Azure Spring Boot Starters](https://github.com/Microsoft/azure-spring-boot)
- [Azure Maven plugins](https://github.com/Microsoft/azure-maven-plugins)
