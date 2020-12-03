TDC Porto Alegre 2020 - CI/CD com Github Actions para deploy no AKS
===================================================================

Este repositório foi criado para o TDC Porto Alegre, para apresentação em 03/12/2020 na trilha da Microsoft. O objetivo foi mostrar como utilizar a funcionalidade do Github Actions para fazer deploy no AKS.

Pré-Requisitos
--------------

Para rodar este exemplo, é necessário:

1. Ter uma conta no Azure, com créditos disponíveis (a opção de [trial](https://azure.microsoft.com/pt-br/free/) gratuito funciona!)
2. Ter o [Azure Cli](https://docs.microsoft.com/pt-br/cli/azure/install-azure-cli?view=azure-cli-latest) instalado
3. Estar com o Azure cli logado na conta que deve ser utilizado e utilizando a subscription correta
4. Ter o [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) instalado
5. Recomendo a utilização do Visual Studio Code para codificação!

Rodar no seu ambiente
---------------------

Para rodar em seu ambiente, é necessário montar o ambiente, subir o código para o um repositório seu (faça um fork!) e definir os secrets com os valores do seu ambiente.

Comandos para montagem de ambiente
----------------------------------

Para montar o ambiente, os seguinte comandos devem ser rodados no powershell, com o Azure Cli instalado:

1. Obter o id da subscrion atual: ```$subscription = (az account show | ConvertFrom-Json).id```
2. Escolha um nome para o seu resource group: ```$resourceGroup = "tdc-poa"```
3. Escolha um nome único para o seu ACR: ```$acrname = "tdcpoaacr"```
4. Escolha um nome para seu cluster AKS: ```$aksname = "tdcpoaaks"```
5. Criar resource group: ```az group create --location eastus --name $resourceGroup```
6. Criar ACR: ```az acr create -g $resourceGroup --name $acrname  --sku basic```
7. Criar AKS: ```az aks create -g $resourceGroup --name $aksname --node-count 1 --attach-acr $acrname```
8. Criar service principal: ```$servicePrincipal = az ad sp create-for-rbac --sdk-auth --skip-assignment```
9. Converter o Service principal em objeto para uso nos proximos comandos: ```$sp = $servicePrincipal |ConvertFrom-Json```
10. Atribuir o papel de Contributor para o Service Principal poder fazer deploy no AKS: ```az role assignment create --assignee $sp.clientId --scope /subscriptions/$subscription/resourcegroups/$resourceGroup/providers/Microsoft.ContainerService/managedClusters/$aksname --role Contributor```
11. Atribuir o papel de AcrPush para o Service Principal poder fazer Push do Container no ACR:  ```az role assignment create --assignee $sp.clientId  --scope /subscriptions/$subscription/resourceGroups/$resourceGroup/providers/Microsoft.ContainerRegistry/registries/$acrname --role AcrPush```
12. Conectar com o AKS localmente: ```az aks get-credentials -g $resourceGroup --name $aksname```
13. Testar conexão com AKS: ```kubectl get namespace```

Cadastrar os secrets no Github
------------------------------

Entre nas configurações do seu repositório do Github, em Secrets e cadastre os seguintes secrets com valores extraídos dos comandos abaixo:

1. AZURE_CREDENTIALS: ```$servicePrincipal```
2. CONTAINER_REGISTRY:  ```$acrName+".azurecr.io"```
3. REGISTRY_USERNAME: ```$sp.clientId```
4. REGISTRY_PASSWORD: ```$sp.clientSecret```

Happy Coding!
