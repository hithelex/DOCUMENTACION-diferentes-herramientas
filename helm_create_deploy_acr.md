# Helm Chart Deployment to Azure Container Registry (ACR)

Este documento describe los pasos para crear un Helm Chart, empaquetarlo y subirlo a un Azure Container Registry (ACR). También se incluyen recomendaciones para asegurar un proceso eficiente y seguro.

## Prerrequisitos

- Azure CLI instalado y configurado.
- Cuenta de Azure con permisos suficientes.
- Helm instalado (versión 3.0 o superior).

## Paso 1: Crear un Helm Chart

1. **Instalar Helm**: Asegúrate de tener Helm instalado en tu máquina. Puedes seguir las instrucciones en [Helm.sh](https://helm.sh/docs/intro/install/).

2. **Crear un nuevo Chart**:
   ```bash
   helm create dope-library
   ```
    Esto creará un directorio ``dope-library`` con una estructura básica de un Helm Chart.

3. **Empaquetar el Chart:**
   ```bash
   helm package dope-library 
   ```
   Esto generará un archivo ``dope-library-0.1.0.tgz``.

## Paso 2: Configurar Azure Container Registry (ACR)

1. **Crear un grupo de recursos y un (ACR)**
    >[!NOTE]
    >La creación del registry tambien se puede lograr desde ``azure portal``

    ```bash
    az group create --name myResourceGroup --location eastus
    az acr create --resource-group myResourceGroup --namemyRegistry --sku Basic
    ```
2. **Habilitar el acceso de administrador en el ACR**
    ```bash
    az acr update --name myRegistry --admin-enabled true
    ```
3. **Obtener las credenciales de acceso**
    ```bash
    ACR_LOGIN_SERVER=$(az acr show --name myRegistry --query loginServer --output tsv)
    ACR_USERNAME=$(az acr credential show --name myRegistry --query username --output tsv)
    ACR_PASSWORD=$(az acr credential show --name myRegistry --query 'passwords[0].value' --output tsv)
    ```
## Paso 3: Autenticar y subir el Chart al ACR

1. **Autenticarse en el ACR usando Helm**
    ```bash
    helm registry login $ACR_LOGIN_SERVER --username $ACR_USERNAME --password $ACR_PASSWORD
    ```
2. **Subir el Chart como un artefacto OCI**
    ```bash
    helm push dope-library-0.1.0.tgz oci://$ACR_LOGIN_SERVER/helm
    ```
## Paso 4: Verificar el Chart en ACR

1. **Listar los Charts en el ACR**
    ```bash
    az acr repository show --name myRegistry --repository helm/dope-library
    ```