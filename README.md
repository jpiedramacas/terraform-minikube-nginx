## Despliegue de Recursos en Clúster de Kubernetes (Minikube) con Terraform

Este documento detalla los pasos necesarios para desplegar recursos en un clúster de Kubernetes utilizando Minikube y Terraform. A continuación, se explicará cómo configurar el entorno, qué hacen los archivos de configuración y los comandos esenciales para gestionar la infraestructura.

### 1. Explicación y Proceso

#### Qué es Terraform y Kubernetes

**Terraform** es una herramienta de infraestructura como código (IaC) que permite definir y gestionar la infraestructura de forma declarativa. Permite automatizar el aprovisionamiento de recursos en diferentes proveedores de nube y plataformas.

**Kubernetes** es un sistema de orquestación de contenedores de código abierto que facilita la automatización del despliegue, la escalabilidad y la gestión de aplicaciones en contenedores.

#### Despliegue con Terraform y Minikube

En este caso, utilizaremos **Minikube**, que es una solución local de Kubernetes, ideal para desarrollo y pruebas en entornos locales o de una sola máquina.

### 2. Explicación de Archivos de Configuración

#### Archivo `k8s.tf`

Este archivo contiene la definición de los recursos que queremos desplegar en Kubernetes, especificados en formato Terraform:

```hcl
resource "kubernetes_namespace" "example" {
  metadata {
    name = "k8s-ns-by-tf"
  }
}

resource "kubernetes_deployment" "example" {
  metadata {
    name = "terraform-example"
    labels = {
      test = "MyExampleApp"
    }
    namespace = "k8s-ns-by-tf"
  }

  spec {
    replicas = 2

    selector {
      match_labels = {
        test = "MyExampleApp"
      }
    }

    template {
      metadata {
        labels = {
          test = "MyExampleApp"
        }
      }

      spec {
        container {
          image = "nginx:1.21.6"
          name  = "example"

          resources {
            limits = {
              cpu    = "0.5"
              memory = "512Mi"
            }
            requests = {
              cpu    = "250m"
              memory = "50Mi"
            }
          }
        }
      }
    }
  }
}
```

- **`kubernetes_namespace`**: Define un nuevo namespace llamado `k8s-ns-by-tf`.
- **`kubernetes_deployment`**: Crea un deployment llamado `terraform-example` que despliega dos réplicas de un contenedor Nginx (`nginx:1.21.6`), con límites y solicitudes de recursos definidos.

#### Archivo `providers.tf`

Este archivo especifica el proveedor de Terraform y la configuración de Kubernetes:

```hcl
terraform {
  required_providers {
    kubernetes = {
      source = "hashicorp/kubernetes"
      version = "2.11.0"
    }
  }
}

provider "kubernetes" {
  config_path    = "~/.kube/config"
  config_context = "minikube"
}
```

- **`required_providers`**: Define la versión del proveedor de Kubernetes que se utilizará en este proyecto.
- **`provider "kubernetes"`**: Configura el proveedor de Kubernetes para que Terraform pueda interactuar con el clúster de Minikube utilizando el contexto `minikube`.

### 3. Comandos Utilizados y Explicación

#### Comandos de Terraform

Los comandos básicos de Terraform que utilizaremos para gestionar la infraestructura son:

```bash
terraform init
```

- **`terraform init`**: Inicializa un directorio de trabajo de Terraform, descargando los plugins y proveedores necesarios especificados en el archivo de configuración (`providers.tf`).

```bash
terraform plan
```

- **`terraform plan`**: Crea un plan de ejecución que describe los cambios que Terraform realizará para alcanzar el estado deseado, sin realizar ningún cambio real en la infraestructura.

```bash
terraform apply
```

- **`terraform apply`**: Aplica los cambios descritos en el plan y realiza la ejecución. Este comando interactúa con la API del proveedor de nube o plataforma (en este caso, Kubernetes a través del proveedor de Terraform) para crear, modificar o eliminar recursos según sea necesario.

#### Comandos de Kubernetes (kubectl)

Una vez desplegados los recursos, puedes verificar su estado utilizando comandos de `kubectl`:

```bash
kubectl get deployment -n k8s-ns-by-tf
```

- **`kubectl get deployment -n k8s-ns-by-tf`**: Muestra el estado de los deployments en el namespace `k8s-ns-by-tf`.

```bash
kubectl get pods -n k8s-ns-by-tf
```

- **`kubectl get pods -n k8s-ns-by-tf`**: Muestra el estado de los pods desplegados en el namespace `k8s-ns-by-tf`.


### Varias Formas de Acceder a los Pods

Una vez que los pods están desplegados y el servicio está configurado, hay varias maneras de acceder a ellos:

1. **A través del Servicio (Service)**
   - Si estás utilizando Minikube, puedes obtener la IP del servicio usando `minikube service`:
     ```bash
     minikube service terraform-example-service -n k8s-ns-by-tf
     ```
   - Si estás en un entorno diferente, puedes obtener la IP del servicio con `kubectl`:
     ```bash
     kubectl get service terraform-example-service -n k8s-ns-by-tf
     ```
     La salida te mostrará la IP y el puerto donde el servicio está expuesto.

2. **Accediendo Directamente a los Pods (Port Forwarding)**
   - Puedes usar `kubectl port-forward` para acceder a un pod específico:
     ```bash
     kubectl port-forward pod/terraform-example-7997bdd7d7-557r6 8080:80 -n k8s-ns-by-tf
     ```
     Esto hará que el pod sea accesible en `http://localhost:8080`.

3. **Usando una IP Externa (si el tipo de servicio es LoadBalancer)**
   - Si el servicio está configurado como `LoadBalancer` y tu proveedor de Kubernetes soporta IPs externas, puedes acceder a la aplicación a través de la IP externa asignada al servicio:
     ```bash
     kubectl get services terraform-example-service -n k8s-ns-by-tf
     ```
     Busca el campo `EXTERNAL-IP` en la salida del comando.

### Verificación

Después de aplicar los cambios y dependiendo de la forma que elijas para acceder a los pods, puedes verificar el acceso abriendo tu navegador web y accediendo a la dirección IP y el puerto correspondiente, o utilizando herramientas como `curl` para hacer solicitudes HTTP.

```bash
curl http://localhost:8080
```

Esto debería mostrarte la página predeterminada de Nginx o cualquier otra salida configurada en tus pods.
