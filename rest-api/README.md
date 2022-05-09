# REST API


### 1. Utworzenie Service Principal
Service Principal potrzebny nam będzie w celu autoryzacji w Postmanie, nadajemy mu uprawnienia tylko jako Reader w danej subskrypcji
* [Create an Azure service principal with the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli)

```bash
SUB_ID="748173f1-20c4-4e68-ac58-641f67a83501"
az account set --subscription $SUB_ID

# Create Service principal
az ad sp create-for-rbac -n "testReaderApp" --role reader --scopes /subscriptions/$SUB_ID 
# Zapisujemy otrzymane dane w bezpiecznym miejscu

APP_ID="53a10bbf-b0d1-4c36-b1dc-31397e194cdc"
az ad sp show --id $APP_ID
az role assignment list --assignee $APP_ID --include-inherited --include-groups
# az ad sp delete --id $APP_ID

# sp list in tenant
# az ad sp list --all --query "[?appOwnerTenantId == '<TENANT_ID>']"
```

### 2. REST APIs with Postman
Instrukcja konfiguracji Postmana:
* [Azure REST APIs with Postman (2021) - youtube](https://youtu.be/6b1J03fDnOg)
* [Azure REST APIs with Postman (2021) - blog](https://blog.jongallant.com/2021/02/azure-rest-apis-postman-2021/)


### 3. Wysyłanie zapytań do API z Cloud Shell / Azure CLI
Istnieje możliwość wysłania zapytania do API bez konieczności tworzenia service principal za pomocą polecenia `az rest` (`This command automatically authenticates using the logged-in credential`)
* [az rest](https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest#az-rest)
* [Azure Cloud Shell](https://shell.azure.com/)


```bash
# GET - resource groups list in given subscription
# {subscriptionId} will be automatically replaced with currently used subscription `az account list --output table`
az rest -m get -u 'https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups?api-version=2020-09-01'

# POST - send request to Resource Graph, get 5 resources (random)
cat <<EOF > body.json
{
    "query": "Resources | project name, type | limit 5"
}
EOF

az rest -m post -u 'https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2021-03-01' --body @body.json

# Azure CLI for Graph Explorer
# az graph query -q 'Resources | project name, type | limit 5'
```


## Linki
* [Azure REST APIs with Postman (2021) - youtube](https://youtu.be/6b1J03fDnOg)
* [Azure REST APIs with Postman (2021) - blog](https://blog.jongallant.com/2021/02/azure-rest-apis-postman-2021/)
* [Create an Azure service principal with the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli)
* [az rest](https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest#az-rest)
* [Azure Cloud Shell](https://shell.azure.com/)
