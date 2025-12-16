+++
title = "Databricks: CICD - AzureDevOps"
date = "2025-06-18"
lastmod = 2025-12-11
author = "TatiMun"
keywords = ["AzureDevOps","Databricks"]
summary = "Troubleshooting de la documentacion de Microsoft"
draft = true
type = "post"
tags = ["AzureDevOps","Databricks"]
+++

## Azure Databricks
Azure Databricks es un servicio de datos dentro de Azure, al momento de su creacion nos proporciona un Workspace que contiene notebooks y demas para correr codigo dentro de clusters efimeros

Contenido:

Azure DevOps
Requisitos y permisos



Creacion de requisitos

Creacion de la app registration  
Creacion de Certificado
Creacion del service connection



Creacion de Pipelines


Creacion de Pipeline


Creacion de Release


Variable


Troubleshootings


Bibiliografia


Notas



Conexion contra AzureDevOps

Service Principal
Managed Identity
Service Account



Service Principal


Primero necesitamos crear una app registration en el Azure Portal


Esta app registration debera tener permisos sobre la  Subscription y API .

Subscription: IAM > Add Role Assignment
API: Ir a la App Registration > API Permissions > Buscamos la API de Databricks. En este punto le damos los permisos necesarios que utilizaremos



Una vez teniendo estos permisos, necesitaremos crear un secret (Certificates and Secrets >Create Secret), donde guardaremos  Secret Value 


Iremos al panel general del App registration y guardaremos los siguientes datos:

Application (Client ID)
Aplication Name



Para avanzar, necesitaremos que la app registration tenga permisos dentro del databricks. Ir a Settings > Identity and Access Management > Service Principal > Microsoft Entra ID y completamos con los datos guardados anteriormente


Una vez agregado, vamos a la parte de Permissions y le damos permisos de User y Manager



Managed Identity
Esta opcion es crear un User Managed Identity, ya que se crea desde el Azure Portal. La funcion es asociar esta identidad a recursos en particular de Azure, para evitar el uso de certificados o secrets usando acceso Federado.

Ir a Azure

Ir a Managed Identity > Crear Managed Identity
Una vez creado, solamente tendremos que ir al IAM de los servicios que querramos usar y asignarle permisos a la Managed Identity

Contra: Managed Identity no puede usarse en otro lugar que no sea por dentro de un recurso, es decir, no podemos loguearnos en nombre del Managed Identity en nuestro local o desde la cloud shell, solo ejecutar acciones desde un recurso usando la managed identity como identidad, valga la redundancia. En este caso, no nos sirve.

Pipeline de Integracion
Para realizar el pipeline de integracion, primero deberemos obtener los tokens para poder consumir la API de databricks

Token de Databricks
Token de Azure DevOps (PAT)
Token de Azure Active Directory (Entra ID)

Para todos los tokens necesitaremos contar con:
- Azure Tenant Id
- Client ID
- Client Secret Value


Token Active Directory Entra ID

    curl --location --request POST 'https://login.microsoftonline.com/$(azure-tenant-id)/oauth2/token' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --header 'Cookie: x-ms-gateway-slice=estsfd; stsservicecookie=estsfd; fpc=AmbU18E-FeJDrvAnHAneXPxfCtJrAQAAAOju390OAAAAAg9H7QEAAAD47t_dDgAAAIeFAvEBAAAA-vHf3Q4AAAA' \
    --data-urlencode 'grant_type=client_credentials' \
    --data-urlencode 'client_id=$(client_id)' \
    --data-urlencode 'client_secret=$(client_pass)' \
    --data-urlencode 'resource=https://management.core.windows.net/')
   

    access_token=$(echo $response_aad | jq -r '.access_token')


El resource es un valor default, esto nos devuelve el token y su tiempo de expiracion.


Token Databricks

    curl --location --request POST "https://login.microsoftonline.com/$(azure-tenant-id)/oauth2/v2.0/token" \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --header 'Cookie: x-ms-gateway-slice=estsfd; stsservicecookie=estsfd; fpc=AmbU18E-FeJDrvAnHAneXPxfCtJrAQAAAOju390OAAAA' \
    --data-urlencode "grant_type=client_credentials" \
    --data-urlencode "client_id=$(client_id)" \
    --data-urlencode "client_secret=$(client_pass)" \
    --data-urlencode "scope=2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default")

    token_data=$(echo "$response_data2" | jq -r '.access_token'
   

    access_token=$(echo $response_aad | jq -r '.access_token')


El resource es un valor default, esto nos devuelve el token y su tiempo de expiracion.


Token Azure DevOps

    curl --location --request POST "https://login.microsoftonline.com/$(azure-tenant-id)/oauth2/v2.0/token" \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --header 'Cookie: x-ms-gateway-slice=estsfd; stsservicecookie=estsfd; fpc=AmbU18E-FeJDrvAnHAneXPxfCtJrAQAAAOju390OAAAA' \
    --data-urlencode "grant_type=client_credentials" \
    --data-urlencode "client_id=$(client_id)" \
    --data-urlencode "client_secret=$(client_pass)" \
    --data-urlencode "scope=499b84ac-1321-427f-aa17-267ca6975798/.default")

    # Extract access token from response
    token_ado=$(echo "$response_ado" | jq -r '.access_token')


El resource es un valor default, esto nos devuelve el token y su tiempo de expiracion.




Databricks API
Databricks API

Create Git Credential
Una vez obtenido los tokens, debemos realizar request hacia la API de Databricks para poder crear el repositorio (Es decir, clonar el repositorio desde Azure DevOps hacia Databricks)

HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
  URL="$(DATABRICKS-CI-HOST)/api/2.0/git-credentials"
  curl -v "${HEADERS[@]}" -X POST "$URL" --data-raw '{
      "personal_access_token": "${TOKEN_ADO}",
      "git_provider": "azureDevOpsServices",
      "git_username": "{{Name of the Service Principal}}"
  }'


Este paso se hace una unica vez por repositorio, luego debemos obtener el ID de la credential para actualizarlo. De lo contrario, nos da error de Invalid Git Credential


Get Git Credential

       HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
        URL="https://adb-xxxxxxx.5.azuredatabricks.net/api/2.0/git-credentials"

        response_credential=$(curl --location --request GET "$URL" \
        --header "Authorization: Bearer $TOKEN_DATA" \
        --header "Content-Type: application/json" \
        --data-raw '')

        git_credential=$(echo $response_credential | jq -r '.credentials[0].credential_id')



Update Git Credential

    HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
    URL="https://adb-xxxxxxxxx.5.azuredatabricks.net/api/2.0/git-credentials/${GIT_CREDENTIAL}"

    response_update=$(curl --location --request PATCH "$URL" \
    --header "Authorization: Bearer $TOKEN_DATA" \
    --header "Content-Type: application/json" \
    --data-raw '{
    "personal_access_token": "$(TOKEN_ADO)",
    "git_username": "{{Nombre del Service Principal}}",
    "git_provider": "azureDevOpsServices"
    }' )


Una vez teniendo las credenciales, podemos proceder a crear el repo

Crear el repositorio

    URL="https://adb-7576006451309545.5.azuredatabricks.net/api/2.0/repos"


    HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json" -H "X-Databricks-Azure-SP-Management-Token: ${ACCESS_TOKEN}" -H "X-Databricks-Org-Id: 7576006451309545" -H "X-Databricks-Azure-Workspace-Resource-Id: /subscriptions/34fbaf6f-e7aa-413f-b35d-f35c2f5a6927/resourceGroups/lakehouse-rg/providers/Microsoft.Databricks/workspaces/dbw-lakehouse-desa")

    curl -v "${HEADERS[@]}" -X POST "$URL" --data-raw '{
   "personal_access_token": "${TOKEN_ADO}",
   "url" : "$(Build.Repository.Uri)",
   "provider": "azureDevOpsServices",
   "path": "/Repos/{FolderName}/{RepoName}",
   "sparse_checkout": {
       "patterns": [
         "parent-folder/child-folder"
       ]
     }
    }'


Esto va a clonar el repo desde azure devops hacia databricks bajo el nombre del service principal
Nota: Es necesario crear la carpeta previamente desde la interfaz grafica para poder hacer el create repo