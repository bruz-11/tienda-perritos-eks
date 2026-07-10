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
* **Infraestructura como Código (IaC):** El clúster fue provisionado utilizando el archivo de configuración `eks-cluster.yaml` mediante la herramienta `eksctl`.
* **Horizontal Pod Autoscaler (HPA):** Configurado tanto para el Backend como para el Frontend, escalando dinámicamente los pods en función del uso de CPU.
* **IAM Roles:** Se utilizó el `LabEksNodeRole` para autorizar a los nodos EC2 a unirse al plano de control de EKS y descargar imágenes desde ECR.

## Justificación de Umbrales de Autoescalado (HPA) y Simulación

Se configuraron los siguientes umbrales estratégicos para el HPA:
* **Frontend (60% CPU):** Se estableció un umbral más agresivo porque la interfaz de usuario es el primer punto de contacto. Al alcanzar este límite, el sistema escala rápidamente para evitar latencia en la experiencia del usuario final.
* **Backend (70% CPU):** La API interna tiene un umbral ligeramente más alto ya que procesa la lógica de negocio y puede encolar conexiones hacia la base de datos de manera más eficiente antes de requerir una nueva réplica.

**Simulación de Carga:**
Para validar el comportamiento del HPA, se ejecutó un generador de tráfico utilizando una imagen temporal de BusyBox para saturar el servicio del Frontend:
```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://frontend-tienda; done"
Pipeline de Integración y Despliegue Continuo (CI/CD)
El flujo automatizado está definido en .github/workflows/deploy.yml.
Cada vez que se realiza un push a la rama main, el pipeline ejecuta los siguientes pasos:

Construye las imágenes de Docker.

Etiqueta las imágenes con una versión específica (eks-v1).

Sube las imágenes al registro seguro de Amazon ECR.

Aplica los manifiestos de Kubernetes (ubicados en la carpeta k8s/) para actualizar los pods en el clúster EKS sin tiempo de inactividad.

Análisis de Métricas y Logs del Pipeline
Tras la ejecución de GitHub Actions, se documentan las siguientes métricas de rendimiento:

Tiempo total de ejecución: ~49 segundos promedio.

Desglose de fases: La fase más costosa es la construcción y envío de imágenes (Build & Push a ECR) tomando ~40s, mientras que el despliegue en EKS (kubectl apply) toma ~4s.

Análisis de Logs (Rollout): Los logs del pipeline confirman que Kubernetes aplica exitosamente una estrategia de "Rolling Update". Los contenedores antiguos solo son terminados cuando las nuevas réplicas (con la etiqueta eks-v1) reportan sus Readiness Probes como exitosos, garantizando cero caídas de servicio (Zero Downtime).

Comandos Útiles y Validación Operativa
Si se necesita operar el clúster manualmente mediante la terminal (CloudShell o local con AWS CLI configurado):

Verificar estado de los Nodos:

Bash
kubectl get nodes
Verificar estado de los Pods y Servicios:

Bash
kubectl get pods -n tienda
kubectl get svc -n tienda
Revisar el estado del Autoescalado (HPA):

Bash
kubectl get hpa -n tienda
Validación de Comunicación Interna (Logs)
Para comprobar que los servicios se comunican correctamente dentro de la red del clúster y que las variables de entorno inyectadas funcionan, se auditan los logs directamente desde los contenedores:

1. Logs del Frontend (Validación de tráfico entrante 200 OK):

Bash
kubectl logs -l app=frontend -n tienda --tail=20
2. Logs del Backend (Validación de conexión exitosa a MySQL vía ClusterIP):

Bash
kubectl logs -l app=backend -n tienda --tail=20