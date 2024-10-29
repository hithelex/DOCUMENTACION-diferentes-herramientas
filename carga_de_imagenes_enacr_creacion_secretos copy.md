# Guía para Cargar una Imagen Docker a Azure Container Registry (ACR) y Crear un Secreto en Kubernetes

## 🚀 Carga de Imagen Docker a Azure Container Registry (ACR)

Sigue estos pasos para cargar una imagen Docker a tu registro de contenedores en Azure Container Registry (ACR).

### 1. Iniciar Sesión en tu ACR

Primero, inicia sesión en tu ACR utilizando Azure CLI. Ejecuta el siguiente comando, reemplazando `<nombre-de-tu-ACR>` por el nombre de tu registro de contenedores:

```bash
az acr login --name <nombre-de-tu-ACR>

```

### 2. Etiquetar la Imagen Docker

Luego, etiqueta la imagen Docker que deseas cargar. Asegúrate de usar el formato adecuado:

```bash
docker tag <nombre-de-la-imagen> <nombre-de-tu-ACR>.azurecr.io/<nombre-de-la-imagen>:<tag>

```

Ejemplo:

```bash
docker tag hhuaranga/kuska-bpm mi-registro.azurecr.io/kuska-bpm:v1

```

### 3. Subir la Imagen a ACR

Sube la imagen etiquetada a tu registro de contenedores:

```bash
docker push <nombre-de-tu-ACR>.azurecr.io/<nombre-de-la-imagen>:<tag>

```

Ejemplo:

```bash
docker push mi-registro.azurecr.io/kuska-bpm:v1

```

### 4. Verificar la Imagen en ACR

Para confirmar que la imagen se haya subido correctamente, ejecuta el siguiente comando para listar las imágenes en tu ACR:

```bash
az acr repository list --name <nombre-de-tu-ACR> --output table

```

---

## 🛠️ Creación de un Secreto en Kubernetes para ACR

Para permitir que Kubernetes extraiga las imágenes desde tu ACR, debes crear un secreto que almacene las credenciales del registro. A continuación se presentan los comandos necesarios para realizar esta tarea.

### 1. Obtener Credenciales del ACR

Primero, necesitas el nombre de usuario y la contraseña del ACR. Puedes obtenerlas ejecutando los siguientes comandos con Azure CLI:

### Nombre de Usuario:

```bash
az acr credential show --name <nombre-de-tu-ACR> --query "username" --output tsv

```

### Contraseña:

```bash
az acr credential show --name <nombre-de-tu-ACR> --query "passwords[0].value" --output tsv

```

### 2. Crear el Secreto en Kubernetes

Utiliza las credenciales obtenidas para crear un secreto de tipo `docker-registry` en Kubernetes. Asegúrate de especificar el namespace si no es el `default`.

```bash
kubectl create secret docker-registry <nombre-del-secreto> \\
    --docker-server=<nombre-de-tu-ACR>.azurecr.io \\
    --docker-username=<nombre-de-usuario> \\
    --docker-password=<contraseña> \\
    --docker-email=<email> \\
    -n <nombre-del-namespace>

```

Ejemplo:

```bash
kubectl create secret docker-registry acr-secret \\
    --docker-server=mi-registro.azurecr.io \\
    --docker-username=<nombre-de-usuario> \\
    --docker-password=<contraseña> \\
    --docker-email=mi-email@ejemplo.com \\
    -n mi-namespace

```

### 3. Usar el Secreto en un Deployment

Para usar el secreto en un deployment, agrega la referencia al secreto en tu archivo YAML. Aquí se muestra un ejemplo de archivo YAML que puedes utilizar:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
      - name: mi-app
        image: <nombre-de-tu-ACR>.azurecr.io/kuska-bpm:v1
      imagePullSecrets:
      - name: acr-secret

```

### 4. Verificar el Secreto

Para confirmar que el secreto se haya creado correctamente, ejecuta:

```bash
kubectl get secret acr-secret --output=yaml -n <nombre-del-namespace>

```