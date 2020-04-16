## Azure Function running selenium webdriver using a customer docker image
The base Azure Function does not contain the necessary chromium packages to run selenium webdriver. This project creates a customer docker image with the required libraries such that it can be run as Azure Function

### 1. Prequesites

- [Docker desktop](https://docs.docker.com/get-docker/)
- [Azure Container Registry](https://docs.microsoft.com/nl-nl/azure/container-registry/container-registry-get-started-portal)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Core Tools version 2.x](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#v2)
- [(optional) Visual Studio Code](https://code.visualstudio.com/)

### 2. Create your own docker image using the DockerFile in docker desktop

Run the following commands:

$acr_id = "\<\<your acr\>\>.azurecr.io"  
docker login $acr_id -u \<\<your username\>\> -p \<\<your password\>\>  
docker build --tag $acr_id/selenium .  
docker push $acr_id/selenium:latest

### 3. Create Azure Function using docker image

Run the following commands:

$rg = "\<\<your resource group name\>\>"  
$loc = "\<\<your location\>\>"  
$plan = "\<\<your azure function plan P1v2\>\>"  
$stor = "\<\<your storage account adhering to function\>\>"  
$fun = "\<\<your azure function name\>\>"  
$acr_id = "\<\<your acr\>\>.azurecr.io"  

az group create -n $rg -l $loc  
az storage account create -n $stor -g $rg --sku Standard_LRS  
az appservice plan create --name $plan --resource-group $rg --sku P1v2 --is-linux  
az functionapp create --resource-group $rg --os-type Linux --plan $plan --deployment-container-image-name $acr_id/selenium:latest --name $fun --storage-account $stor  

### 4. Run Azure Function as HTTP trigger

The following code in __init__.py will return all URLs in the following webpage:

`
from selenium import webdriver
driver.get('http://www.ubuntu.com/')
links = driver.find_elements_by_tag_name("a")
`