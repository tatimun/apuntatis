+++
title = "Databricks: CICD - AzureDevOps"
date = "2025-06-18"
lastmod = 2025-12-11
author = "TatiMun"
keywords = ["AzureDevOps","Databricks"]
summary = "Troubleshooting de la documentacion de Microsoft"
draft = false
type = "post"
tags = ["AzureDevOps","Databricks"]
+++

# Azure Databricks 
Azure Databricks es un servicio de datos dentro de Azure, al momento de su creacion nos proporciona un Workspace que contiene notebooks y demas para correr codigo dentro de clusters efimeros


<br>
Contenido:

- [Azure DevOps](#¿Que-es-Azure-DevOps?)
- [Requisitos y permisos](#requisitos)
<br>

 ### Creacion de requisitos
        
- [Creacion de la app registration  ](#Creacion-de-app-registration)
- [Creacion de Certificado](#Creacion-de-Certificado)
- [Creacion del service connection](#Creacion-del-service-connection)
<br>

 ### Creacion de Pipelines  
- [Creacion de Pipeline](#Creacion-del-Pipeline)
- [Creacion de Release](#Creacion-del-Release)
- [Variable](#variables)  




- [Troubleshootings](#Troubleshootings)
- [Bibiliografia](#Bibliografia)
- [Notas](#Notas)  



## Conexion contra AzureDevOps 

- [Service Principal](#Service-Principal)
- [Managed Identity](#Managed-Identity)
- [Service Account](#Service-Account)



<br>

# Service Principal

1. Primero necesitamos crear una app registration en el [Azure Portal](https://portal.azure.com)

2. Esta app registration debera tener permisos sobre la <i> Subscription y API </i>. 
    -   Subscription: IAM > Add Role Assignment
    -   API: Ir a la App Registration > API Permissions > Buscamos la API de Databricks. En este punto le damos los permisos necesarios que utilizaremos

3. Una vez teniendo estos permisos, necesitaremos crear un secret (Certificates and Secrets >Create Secret), donde guardaremos <i> Secret Value </i>

4. Iremos al panel general del App registration y guardaremos los siguientes datos: 

    - Application (Client ID)
    - Aplication Name

5. Para avanzar, necesitaremos que la app registration tenga permisos dentro del databricks. Ir a Settings > Identity and Access Management > Service Principal > Microsoft Entra ID y completamos con los datos guardados anteriormente

6. Una vez agregado, vamos a la parte de Permissions y le damos permisos de User y Manager 



        
# Managed Identity 

Esta opcion es crear un User Managed Identity, ya que se crea desde el Azure Portal. La funcion es asociar esta identidad a recursos en particular de Azure, para evitar el uso de certificados o secrets usando acceso Federado. 

1. Ir a [Azure](https://portal.azure.com)
2. Ir a Managed Identity > Crear Managed Identity 
3. Una vez creado, solamente tendremos que ir al IAM de los servicios que querramos usar y asignarle permisos a la Managed Identity

Contra: Managed Identity no puede usarse en otro lugar que no sea por dentro de un recurso, es decir, no podemos loguearnos en nombre del Managed Identity en nuestro local o desde la cloud shell, solo ejecutar acciones desde un recurso usando la managed identity como identidad, valga la redundancia. En este caso, no nos sirve. 



# Pipeline de Integracion

Para realizar el pipeline de integracion, primero deberemos obtener los tokens para poder consumir la API de databricks

- Token de Databricks
- Token de Azure DevOps (PAT)
- Token de Azure Active Directory (Entra ID)

Para todos los tokens necesitaremos contar con:
    - Azure Tenant Id
    - Client ID
    - Client Secret Value


---

### Token Active Directory Entra ID

        curl --location --request POST 'https://login.microsoftonline.com/$(azure-tenant-id)/oauth2/token' \
        --header 'Content-Type: application/x-www-form-urlencoded' \
        --header 'Cookie: x-ms-gateway-slice=estsfd; stsservicecookie=estsfd; fpc=AmbU18E-FeJDrvAnHAneXPxfCtJrAQAAAOju390OAAAAAg9H7QEAAAD47t_dDgAAAIeFAvEBAAAA-vHf3Q4AAAA' \
        --data-urlencode 'grant_type=client_credentials' \
        --data-urlencode 'client_id=$(client_id)' \
        --data-urlencode 'client_secret=$(client_pass)' \
        --data-urlencode 'resource=https://management.core.windows.net/')
       

        access_token=$(echo $response_aad | jq -r '.access_token')


El resource es un valor default, esto nos devuelve el token y su tiempo de expiracion.

---

### Token Databricks 

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


---

### Token Azure DevOps 

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


---
---
---

# Databricks API
[Databricks API](https://docs.databricks.com/api/workspace/introduction)


### Create Git Credential

Una vez obtenido los tokens, debemos realizar request hacia la API de Databricks para poder crear el repositorio (Es decir, clonar el repositorio desde Azure DevOps hacia Databricks)

    HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
      URL="$(DATABRICKS-CI-HOST)/api/2.0/git-credentials"
      curl -v "${HEADERS[@]}" -X POST "$URL" --data-raw '{
          "personal_access_token": "${TOKEN_ADO}",
          "git_provider": "azureDevOpsServices",
          "git_username": "{{Name of the Service Principal}}"
      }'
   
Este paso se hace una unica vez por repositorio, luego debemos obtener el ID de la credential para actualizarlo. De lo contrario, nos da error de Invalid Git Credential

---

### Get Git Credential


           HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
            URL="https://adb-7576006451309545.5.azuredatabricks.net/api/2.0/git-credentials"
   
            response_credential=$(curl --location --request GET "$URL" \
            --header "Authorization: Bearer $TOKEN_DATA" \
            --header "Content-Type: application/json" \
            --data-raw '')

            git_credential=$(echo $response_credential | jq -r '.credentials[0].credential_id')
   
### Update Git Credential

        HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
        URL="https://adb-7576006451309545.5.azuredatabricks.net/api/2.0/git-credentials/${GIT_CREDENTIAL}"
   
        response_update=$(curl --location --request PATCH "$URL" \
        --header "Authorization: Bearer $TOKEN_DATA" \
        --header "Content-Type: application/json" \
        --data-raw '{
        "personal_access_token": "$(TOKEN_ADO)",
        "git_username": "{{Nombre del Service Principal}}",
        "git_provider": "azureDevOpsServices"
        }' )
  

Una vez teniendo las credenciales, podemos proceder a crear el repo

### Crear el repositorio 

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

  
<i>Nota: Es necesario crear la carpeta previamente desde la interfaz grafica para poder hacer el create repo


## Pipeline Compilacion y Despliegue Ejemplo 

```parameters:
  - name: nombreFolder
    displayName: 'Ingresá el nombre de la carpeta'
    type: string
    default: 'folder'

trigger:
- main


pool:
  name: PoolAgentName

variables:
  - group: GroupName
  - name: branchName
    value: main

steps:
- task: Bash@3
  displayName: '1. Login con Managed Identity y obtener secrets'
  inputs:
    targetType: 'inline'
    script: |
      echo "Login con Managed Identity..."
      az login --identity --username "$(az identity show --name NAME-MI --resource-group RG-GROUP --query id -o tsv)"

      CLIENT_ID=$(az keyvault secret show --vault-name NAME-KEYVAULT --name client-id --query value -o tsv)
      CLIENT_SECRET=$(az keyvault secret show --vault-name NAME-KEYVAULT --name client-secret --query value -o tsv)
      PAT_TOKEN=$(az keyvault secret show --vault-name NAME-KEYVAULT --name pat-token-databricks --query value -o tsv)


      if [[ -z "$CLIENT_ID" ]]; then
        echo "❌ ERROR: CLIENT_ID está vacío o no se pudo obtener del Key Vault"
        exit 1
      else
        echo "✅ CLIENT_ID obtenido (parcial): ${CLIENT_ID:0:5}*****"
      fi

      # Validar CLIENT_SECRET
      if [[ -z "$CLIENT_SECRET" ]]; then
        echo "❌ ERROR: CLIENT_SECRET está vacío o no se pudo obtener del Key Vault"
        exit 1
      else
        echo "✅ CLIENT_SECRET obtenido (parcial): ${CLIENT_SECRET:0:5}*****"
      fi

      # Validar PAT_TOKEN
      if [[ -z "$PAT_TOKEN" ]]; then
        echo "ERROR: PAT_TOKEN está vacío o no se pudo obtener del Key Vault"
        exit 1
      else
        echo "✅ PAT_TOKEN obtenido (parcial): ${PAT_TOKEN:0:5}*****"
      fi

        echo "-------------------------"

      echo "##vso[task.setvariable variable=PAT_TOKEN;issecret=true]$PAT_TOKEN"
      echo "##vso[task.setvariable variable=CLIENT_SECRET;issecret=true]$CLIENT_SECRET"
      echo "##vso[task.setvariable variable=CLIENT_ID;issecret=true]$CLIENT_ID"



      

- task: Bash@3
  displayName: '2. Obtener token OAuth para Databricks'
  inputs:
    targetType: 'inline'
    script: |
      echo "Obteniendo Access Token válido para Databricks"
      TOKEN_DATA=$(az account get-access-token --scope 2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default --query accessToken -o tsv)

      if [[ -z "$TOKEN_DATA" || "$TOKEN_DATA" == "null" ]]; then
        echo "No se pudo obtener el token OAuth"
        exit 1
      fi

      echo "Token OAuth obtenido"
      echo "##vso[task.setvariable variable=TOKEN_DATA]$TOKEN_DATA"

- task: Bash@3
  displayName: '3. Borrar Git Credential existente'
  inputs:
    targetType: 'inline'
    script: |
      echo "Eliminando credenciales existentes si hay..."
      URL_LIST="${DATABRICKS_CI_HOST}/api/2.0/git-credentials"
      response=$(curl -X GET "$URL_LIST" -H "Authorization: Bearer $TOKEN_DATA" -H "Content-Type: application/json")
      CREDENTIAL_ID=$(echo "$response" | jq -r '.credentials[0].credential_id')

      if [[ "$CREDENTIAL_ID" != "null" && -n "$CREDENTIAL_ID" ]]; then
        curl -X DELETE "${DATABRICKS_CI_HOST}/api/2.0/git-credentials/$CREDENTIAL_ID" -H "Authorization: Bearer $TOKEN_DATA"
        echo "Credential eliminada"
      else
        echo "No se encontraron credenciales para eliminar"
      fi

- task: Bash@3
  displayName: '4. Crear Git Credential en Databricks'
  env:
    PAT_TOKEN: $(PAT_TOKEN)
    TOKEN_DATA: $(TOKEN_DATA)
  inputs:
    targetType: 'inline'
    script: |
      echo "🛠 Creando Git Credential..."
      echo "🔍 Validando que PAT_TOKEN no esté vacío: ${PAT_TOKEN:0:5}*****"

      CREATE_URL="${DATABRICKS_CI_HOST}/api/2.0/git-credentials"

      response_create=$(curl --location --request POST "$CREATE_URL" \
        --header "Authorization: Bearer $TOKEN_DATA" \
        --header "Content-Type: application/json" \
        --data-raw "{
          \"personal_access_token\": \"$PAT_TOKEN\",
          \"git_provider\": \"azureDevOpsServices\",
          \"git_username\": \"USER@gmail.com.ar\"
        }")

      echo "📄 Respuesta completa:"
      echo "$response_create"

      GIT_CREDENTIAL=$(echo "$response_create" | jq -r '.credential_id')

      if [[ "$GIT_CREDENTIAL" == "null" || -z "$GIT_CREDENTIAL" ]]; then
        echo "❌ Error creando Git Credential"
        exit 1
      fi

      echo "✅ Git Credential creada con ID: $GIT_CREDENTIAL"nombreFolderJs
      echo "##vso[task.setvariable variable=GIT_CREDENTIAL]$GIT_CREDENTIAL"


- task: Bash@3
  displayName: '5. Crear repo si no existe (con soporte sparse_checkout)'
  env:
    TOKEN_DATA: $(TOKEN_DATA)
  inputs:
    targetType: 'inline'
    script: |
      echo "Buscando o creando repo..."

      URL="${DATABRICKS_CI_HOST}/api/2.0/repos"
      urlGitRepo="https://dev.azure.com/Organization/Repository/_git/$(Build.Repository.Name)"
      searchPath="/Repos/${{ parameters.nombreFolder }}/$(Build.Repository.Name)"
      branchName="$(branchName)"

      echo "Path: $searchPath" 
      echo "URL Git: $urlGitRepo" //https://dev.azure.com/Organization/Repository
      echo "Branch: $(branchName)"

      response_get=$(curl -s -X GET "$URL" -H "Authorization: Bearer $TOKEN_DATA")
      ExtractedID=$(echo "$response_get" | jq -r --arg path "$searchPath" '.repos[] | select(.path == $path) | .id')

      if [[ -z "$ExtractedID" || "$ExtractedID" == "null" ]]; then
        echo "Repo no existe. Se va a crear..."

        JSON_PAYLOAD=$(jq -n \
          --arg url "$urlGitRepo" \
          --arg provider "azureDevOpsServices" \
          --arg path "$searchPath" \
          --arg branch "$branchName" \
          '{url: $url, provider: $provider, path: $path, branch: $branch}')

        echo " JSON a enviar:"
        echo "$JSON_PAYLOAD"

        curl_output=$(curl -s -w "\n%{http_code}" -X POST "$URL" \
          -H "Authorization: Bearer $TOKEN_DATA" \
          -H "Content-Type: application/json" \
          --data "$JSON_PAYLOAD")

        response_body=$(echo "$curl_output" | sed '$d')
        http_status=$(echo "$curl_output" | tail -n1)

        echo " Respuesta:"
        echo "$response_body"
        echo "Status: $http_status"

        ExtractedID=$(echo "$response_body" | jq -r '.id')

        if [[ "$http_status" != "200" || -z "$ExtractedID" || "$ExtractedID" == "null" ]]; then
          echo " No se pudo crear el repo. Detalle:"
          echo "$response_body"
          exit 1
        else
          echo " Repo creado con ID: $ExtractedID"
        fi
      else
        echo "📎 Repo ya existe con ID: $ExtractedID"
      fi

      echo "##vso[task.setvariable variable=REPO_ID]$ExtractedID"

- task: Bash@3
  displayName: '6. Actualizar branch del repo'
  env:
    TOKEN_DATA: $(TOKEN_DATA)
    REPO_ID: $(REPO_ID)
  inputs:
    targetType: 'inline'
    script: |
      echo "🔁 Actualizando branch del repo..."
      URL="${DATABRICKS_CI_HOST}/api/2.0/repos/$REPO_ID"
      BRANCH=$(branchName)
      JSON_PAYLOAD=$(jq -n --arg branch "$BRANCH" '{branch: $branch}')

      echo "📤 Payload:"
      echo "$JSON_PAYLOAD"

      response_update=$(curl --location --request PATCH "$URL" \
        --header "Authorization: Bearer $TOKEN_DATA" \
        --header "Content-Type: application/json" \
        --data "$JSON_PAYLOAD")

      echo "📄 Respuesta al actualizar el repo:"
      echo "$response_update"


- task: Bash@3
  displayName: '7. Cleanup workspace'
  inputs:
    targetType: 'inline'
    script: |
      echo "Limpiando workspace..."
      rm -rf $(Build.SourcesDirectory)/*
      echo "Listo"
```