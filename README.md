# Tienda de Perritos - AWS EKS Deployment

Este repositorio contiene el código fuente, la infraestructura como código (archivos manifiestos de Kubernetes) y el pipeline de CI/CD para el despliegue de la aplicación de microservicios "Tienda de Perritos" en Amazon Elastic Kubernetes Service (EKS).

## Arquitectura del Sistema

La aplicación está diseñada bajo una arquitectura de microservicios contenerizados utilizando Docker y orquestados con Kubernetes. Los componentes principales son:

* **Frontend:** Interfaz de usuario expuesta a Internet mediante un LoadBalancer de AWS.
* **Backend:** API que gestiona la lógica de negocio y la comunicación entre el frontend y la base de datos.
* **Base de Datos (MySQL):** Almacenamiento persistente para los productos de la tienda, comunicada internamente a través de un servicio `ClusterIP`.

## Tecnologías y Servicios Utilizados

* **Cloud Provider:** Amazon Web Services (AWS)
* **Orquestación:** Amazon EKS (Elastic Kubernetes Service)
* **Cómputo:** Nodos EC2 gestionados (`t3.medium`) en subredes públicas.
* **Registro de Contenedores:** Amazon ECR (Etiqueta: `eks-v1`)
* **CI/CD:** GitHub Actions
* **Observabilidad:** AWS CloudWatch (Container Insights y Logs de Auditoría)

## Configuración del Clúster (Infraestructura)

El clúster fue configurado para garantizar alta disponibilidad y escalabilidad automática:
* **Horizontal Pod Autoscaler (HPA):** Configurado tanto para el Backend como para el Frontend, escalando dinámicamente los pods en función del uso de CPU.
* **IAM Roles:** Se utilizó el `LabEksNodeRole` para autorizar a los nodos EC2 a unirse al plano de control de EKS y descargar imágenes desde ECR.

## Pipeline de Integración y Despliegue Continuo (CI/CD)

El flujo automatizado está definido en `.github/workflows/deploy.yml`. 
Cada vez que se realiza un `push` a la rama `main`, el pipeline ejecuta los siguientes pasos:
1.  Construye las imágenes de Docker.
2.  Etiqueta las imágenes con una versión específica (`eks-v1`).
3.  Sube las imágenes al registro seguro de Amazon ECR.
4.  Aplica los manifiestos de Kubernetes (ubicados en la carpeta `k8s/`) para actualizar los pods en el clúster EKS sin tiempo de inactividad.

## Comandos Útiles (Operaciones)

Si se necesita operar el clúster manualmente mediante la terminal (`CloudShell` o local con AWS CLI configurado):

Verificar estado de los Nodos:

kubectl get nodes

Verificar estado de los Pods y Servicios:

kubectl get pods -n tienda
kubectl get svc -n tienda

Revisar el estado del Autoescalado (HPA):

kubectl get hpa -n tienda
