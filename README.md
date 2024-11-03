# 🌐 Práctica CI/CD con ArgoCD y Kubernetes

Este proyecto implementa un pipeline de CI/CD utilizando herramientas como **Docker**, **CircleCI**, **ArgoCD** y **Kubernetes** para desplegar una aplicación en un clúster de Kubernetes.

## 🚀 Tecnologías Usadas
- **Docker**: Conteneriza la aplicación para asegurar que se ejecute en cualquier entorno.
- **CircleCI**: Orquestación del pipeline de CI/CD.
- **ArgoCD**: Controla y monitorea el despliegue en Kubernetes.
- **Kubernetes**: Orquesta contenedores y asegura la alta disponibilidad de la aplicación.

## 📂 Estructura del Proyecto
- `.circleci/`: Configuración de CircleCI para el pipeline de CI/CD.
- `backend/`: Código de la aplicación backend.
- `frontend/`: Código de la aplicación frontend (si aplica).
- `hooks/`: Hooks adicionales para el proyecto.
- `img/`: Imágenes utilizadas en el proyecto.
- `k8s/`: Archivos de configuración de Kubernetes para el despliegue de la aplicación en el clúster.
- `scripts/`: Scripts útiles para la automatización y administración.
- `tests/`: Pruebas unitarias y de integración.
- `.flake8`: Configuración para el linter de Python.
- `copier.yml`: Archivo de configuración para copier.
- `docker-compose.yml`: Archivo de configuración de Docker Compose.
- `Dockerfile`: Configuración para construir la imagen Docker de la aplicación.
- `LICENSE`: Archivo de licencia del proyecto.
- `README.md`: Documentación del proyecto.
- `release-notes.md`: Notas de las versiones.
- `requirements.txt`: Dependencias de Python requeridas para la aplicación.
- `SECURITY.md`: Información de seguridad del proyecto.
- `setup.cfg`: Configuración adicional de la aplicación.
- `sonar-project.properties`: Configuración para SonarCloud.

## ✅ Pre-requisitos
- **Docker** instalado en tu máquina local.
- **Kind** (Kubernetes in Docker) para crear el clúster local.
- **kubectl** para interactuar con Kubernetes.
- **ArgoCD** instalado en el clúster.
- **CircleCI** configurado con las variables de entorno necesarias (token de DockerHub, SonarCloud, y ArgoCD).

## ⚙️ Instalación y Configuración

### 1. Crear el Clúster de Kubernetes usando Kind
Ejecuta los siguientes comandos para crear el clúster en Kind y configurar el controlador Ingress:

```bash
readonly KIND_IMAGE=kindest/node:v1.30.4
readonly NAME=argo
kind create cluster --name $NAME --image $KIND_IMAGE --config kind/cluster.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
sleep 15

kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s

2. Instalar ArgoCD en el Clúster
Usa Helm para instalar ArgoCD en Kubernetes:
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade --install --wait --timeout 15m --atomic --namespace argocd --create-namespace argo-cd argo/argo-cd

3. Configurar el Acceso Local a ArgoCD
Agrega argocd.local a tu archivo /etc/hosts:
echo "127.0.0.1 argocd.local" | sudo tee -a /etc/hosts
Luego, ejecuta el port-forwarding:kubectl port-forward svc/argocd-server -n argocd 8080:443

4. Autenticación en ArgoCD
Obtén la contraseña inicial para el usuario admin:kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

5. Configurar ArgoCD para Desplegar la Aplicación
Usa el archivo de configuración application.yaml en la carpeta k8s/ para definir el despliegue de la aplicación. Actualiza los datos de repoURL y path con los de tu repositorio.
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: practica-cicd
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/voydelau28/practica_cicd.git'
    targetRevision: main
    path: k8s
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
Aplica este archivo con:kubectl apply -f k8s/application.yaml

📋 Pipeline de CircleCI
CircleCI está configurado para ejecutar pruebas, análisis de calidad y desplegar la aplicación en Kubernetes. Asegúrate de tener las siguientes variables de entorno configuradas en tu proyecto de CircleCI:

DOCKERHUB_USERNAME
DOCKERHUB_PASSWORD
SONAR_TOKEN
El archivo .circleci/config.yml define el flujo del pipeline de CI/CD:

Construcción y pruebas: Construye la imagen Docker, ejecuta flake8 y pytest para verificar el código y la cobertura.
Análisis de calidad: Usa SonarCloud para analizar el código.
Despliegue: Publica la imagen Docker y despliega en Kubernetes a través de ArgoCD.

📜 Comandos Útiles
Desplegar manualmente en ArgoCD:
argocd app sync practica-cicd
Ver el estado de la aplicación en ArgoCD:
argocd app get practica-cicd

📝 Notas Adicionales
Revisa que todos los servicios estén en estado "Running" en el namespace argocd:
kubectl get pods -n argocd


