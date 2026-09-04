# Kubernetes Online Boutique Deployment with Helm and Helmfile

## Project Overview

This project demonstrates the deployment and management of the Google Cloud Online Boutique microservices application using Kubernetes, Helm, and Helmfile.

The project was originally deployed using standard Kubernetes manifests with `kubectl`. The application was then converted into a Helm-based deployment to demonstrate how Helm charts and Helmfile can simplify the management of multiple microservices.

The Online Boutique application consists of multiple independent microservices. Instead of maintaining separate Kubernetes deployment and service manifests for each application manually, this project uses reusable Helm charts and service-specific values files.

Helmfile is then used to orchestrate the deployment of all services from a single configuration.

## Technologies Used

* Kubernetes
* Minikube
* Helm
* Helmfile
* Docker
* YAML
* Kubernetes Deployments
* Kubernetes Services
* ConfigMaps
* Microservices Architecture

## Application Architecture

The application consists of the following microservices:

* adservice
* cartservice
* checkoutservice
* currencyservice
* emailservice
* frontend
* paymentservice
* productcatalogservice
* recommendationservice
* shippingservice
* redis-cart

The services communicate internally within the Kubernetes cluster.

The frontend service provides the user-facing interface for the Online Boutique application.

## Project Structure

```text
kubernetes-online-boutique-helm/
│
├── charts/
│   ├── microservice/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │
│   └── redis/
│       ├── .helmignore
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│
├── manifests/
│   ├── online-boutique-configmaps.yaml
│   ├── online-boutique-deployments.yaml
│   └── online-boutique-services.yaml
│
├── values/
│   ├── ad-service-values.yaml
│   ├── cart-service-values.yaml
│   ├── checkout-service-values.yaml
│   ├── currency-service-values.yaml
│   ├── email-service-values.yaml
│   ├── frontend-values.yaml
│   ├── payment-service-values.yaml
│   ├── productcatalog-service-values.yaml
│   ├── recommendation-service-values.yaml
│   ├── redis-values.yaml
│   └── shipping-service-values.yaml
│
├── .gitignore
├── .helmignore
├── helmfile.yaml
└── README.md
```

## Original Kubernetes Deployment

Before converting the application to Helm, the Online Boutique application was deployed using standard Kubernetes manifests.

The original Kubernetes manifests were exported and retained in the repository for reference.

These include:

```text
manifests/online-boutique-deployments.yaml
manifests/online-boutique-services.yaml
manifests/online-boutique-configmaps.yaml
```

The original deployment used Kubernetes Deployments and Services to manage the microservices.

The Helm implementation provides a more reusable and scalable approach for managing the same infrastructure.

## Helm Chart Design

Two Helm charts were created for the project.

### Microservice Chart

A reusable Helm chart was created for the Online Boutique microservices.

The same chart is reused by multiple services, including:

* adservice
* cartservice
* checkoutservice
* currencyservice
* emailservice
* frontend
* paymentservice
* productcatalogservice
* recommendationservice
* shippingservice

The chart contains reusable templates for Kubernetes resources such as:

* Deployment
* Service
* ServiceAccount
* HorizontalPodAutoscaler
* Ingress

The configuration for each service is provided through individual values files.

For example, a service values file defines parameters such as:

```yaml
appName:
appReplicas:
appImage:
appVersion:
containerPort:
servicePort:
containerEnvVars:
```

This approach allows one reusable Helm chart to deploy multiple microservices with different configurations.

## Redis Chart

A separate Helm chart was created for the Redis Cart service.

Redis has different deployment requirements from the other application services, so it is managed independently from the reusable microservice chart.

## Helmfile Orchestration

Helmfile is used to orchestrate all Helm releases.

Instead of manually running individual Helm commands for each microservice, Helmfile manages all releases from a single configuration file.

The Helmfile configuration references:

* The reusable microservice chart
* The Redis chart
* Individual values files for each service

Each service is deployed as an independent Helm release.

Example deployment structure:

```text
Helmfile
│
├── adservice
├── cartservice
├── checkoutservice
├── currencyservice
├── emailservice
├── frontend
├── paymentservice
├── productcatalogservice
├── recommendationservice
├── shippingservice
└── rediscart
```

## Deploying the Application

The application can be deployed using Helmfile.

Run:

```bash
helmfile sync
```

Helmfile processes each release and performs the equivalent of Helm install or upgrade operations.

During deployment, Helmfile builds the required chart dependencies and deploys each microservice.

Example output:

```text
Building dependency release=redis
Building dependency release=currencyservice
Building dependency release=productcatalogservice
Building dependency release=recommendationservice
Building dependency release=paymentservice
Building dependency release=shippingservice
Building dependency release=adservice
Building dependency release=frontendservice
Building dependency release=checkoutservice
Building dependency release=emailservice
Building dependency release=cartservice
```

Helmfile then installs or upgrades the releases.

Example:

```text
Release "rediscart" does not exist. Installing it now.
Release "checkoutservice" does not exist. Installing it now.
Release "currencyservice" does not exist. Installing it now.
Release "frontendservice" does not exist. Installing it now.
Release "cartservice" does not exist. Installing it now.
```

## Checking Helm Releases

After deployment, Helm releases can be checked using:

```bash
helm list
```

Or:

```bash
helm ls
```

Helmfile releases can also be checked using:

```bash
helmfile list
```

This displays the Helm releases managed by Helmfile.

## Checking Kubernetes Pods

After running the Helmfile deployment, Kubernetes pods can be checked using:

```bash
kubectl get pods
```

Example output:

```text
NAME                                      READY   STATUS
adservice-85d987c66-jvzw                  1/1     Running
adservice-85d987c66-zc7jv                 1/1     Running
cartservice-7847487d96-kmkhc              1/1     Running
cartservice-7847487d96-qsx67              1/1     Running
checkoutservice-5885b66fcc-ns8cn          1/1     Running
checkoutservice-5885b66fcc-rwfgv          1/1     Running
emailservice-7d7dccb79-gjzkg              1/1     Running
emailservice-7d7dccb79-zdfs2              1/1     Running
frontend-b8cd96f5c-61ck8                  1/1     Running
paymentservice-7596f79c97-z6kpg           1/1     Running
productcatalogservice-558bbf7d5f-qbskq    1/1     Running
recommendationservice-5d7dd49f4-c8s5v     1/1     Running
redis-cart-686d6f8c98-9rkpp               1/1     Running
shippingservice-6fdd89d67-djzj8           1/1     Running
```

Most services successfully started and reached the `Running` state after deployment.

## Known Issue: Currency Service CrashLoopBackOff

During one of the Helmfile deployments, the `currencyservice` pods entered a `CrashLoopBackOff` state.

The affected pods were:

```text
currencyservice-669585dd-jgnvf
currencyservice-669585dd-xfvwd
```

Example status:

```text
READY   STATUS             RESTARTS
0/1     CrashLoopBackOff   6
0/1     CrashLoopBackOff   6
```

This indicates that the container starts but repeatedly crashes.

This is an important Kubernetes troubleshooting scenario because a successful Helm installation does not necessarily mean that every application container is healthy.

Helm and Helmfile can report a release as successfully deployed while an application running inside the Kubernetes cluster experiences runtime failures.

The following commands can be used to investigate the issue:

```bash
kubectl logs <currencyservice-pod-name>
```

For example:

```bash
kubectl logs currencyservice-669585dd-jgnvf
```

The previous container logs can also be inspected using:

```bash
kubectl logs <currencyservice-pod-name> --previous
```

Pod events and configuration can be inspected using:

```bash
kubectl describe pod <currencyservice-pod-name>
```

For example:

```bash
kubectl describe pod currencyservice-669585dd-jgnvf
```

The deployment can also be inspected using:

```bash
kubectl describe deployment currencyservice
```

Possible areas to investigate include:

* Incorrect environment variables
* Incorrect container arguments
* Missing dependencies
* Incorrect service configuration
* Incorrect image configuration
* Incorrect port configuration
* Application startup errors

This issue will be investigated as part of the Kubernetes troubleshooting process.

## Helmfile Sync

To deploy or update all services:

```bash
helmfile sync
```

This command:

1. Reads the Helmfile configuration.
2. Processes all defined releases.
3. Builds chart dependencies.
4. Performs Helm install operations for new releases.
5. Performs Helm upgrade operations for existing releases.

This provides a single command for managing the complete microservices application.

## Helmfile List

To view all releases managed by Helmfile:

```bash
helmfile list
```

This provides visibility into the releases configured within the Helmfile project.

## Destroying the Application

Helmfile can also remove all deployed releases.

Run:

```bash
helmfile destroy
```

This removes the Helm releases managed by the Helmfile configuration.

After destruction, Kubernetes resources can be checked using:

```bash
kubectl get pods
```

And:

```bash
helm list
```

This demonstrates the complete lifecycle management of the application:

```text
Deploy
   ↓
helmfile sync
   ↓
Verify
   ↓
helmfile list
kubectl get pods
   ↓
Destroy
   ↓
helmfile destroy
```

## Helm Commands Used During Development

Individual Helm charts can be validated using:

```bash
helm lint charts/microservice
```

A Helm chart can be rendered locally without installing it:

```bash
helm template <release-name> charts/microservice
```

A service-specific values file can be applied during template rendering:

```bash
helm template \
  -f values/<service-values-file>.yaml \
  <release-name> \
  charts/microservice
```

This allows Helm templates to be tested before deployment.

## Kubernetes Commands Used

Check pods:

```bash
kubectl get pods
```

Check pods across all namespaces:

```bash
kubectl get pods -A
```

Check all resources in a namespace:

```bash
kubectl get all -n <namespace>
```

Check available namespaces:

```bash
kubectl get namespaces
```

Check pod logs:

```bash
kubectl logs <pod-name>
```

Describe a pod:

```bash
kubectl describe pod <pod-name>
```

## Screenshots

The project includes screenshots demonstrating the deployment process.

### Kubernetes Pods

The Kubernetes pod output demonstrates the deployment of the Online Boutique microservices using Helm and Helmfile.

The majority of the services successfully reached the `Running` state.

The screenshot also captured a `CrashLoopBackOff` issue affecting the `currencyservice`, which demonstrates a real Kubernetes troubleshooting scenario.

Screenshot file:

```text
04-kubernetes-pods.png
```

### Helmfile Deployment

The Helmfile sync output demonstrates Helmfile processing multiple releases and installing the microservices.

The output is extensive because multiple Helm releases are deployed.

The screenshot captures part of the deployment process, including:

* Building Helm chart dependencies
* Installing the Redis release
* Installing microservice releases
* Deploying services through Helmfile

Screenshot file:

```text
02-helmfile-sync.png
```

## Key Learning Outcomes

Through this project, I gained practical experience with:

* Deploying applications to Kubernetes
* Managing Kubernetes Deployments and Services
* Converting Kubernetes deployments into Helm charts
* Creating reusable Helm templates
* Using values files for environment-specific configuration
* Managing multiple microservices using a reusable Helm chart
* Creating a separate Helm chart for Redis
* Using Helmfile to orchestrate multiple Helm releases
* Deploying multiple services with a single command
* Managing Helm release lifecycle
* Destroying infrastructure using Helmfile
* Troubleshooting Kubernetes pod failures
* Investigating `CrashLoopBackOff` errors
* Understanding the difference between successful infrastructure deployment and application runtime health

## Future Improvements

Future improvements to this project include:

* Investigating and resolving the `currencyservice` CrashLoopBackOff issue
* Adding readiness and liveness probes consistently across services
* Deploying the application to Linode Kubernetes Engine (LKE)
* Configuring cloud-based load balancing
* Adding a CI/CD pipeline using GitHub Actions
* Adding automated Helm chart validation
* Adding Helm chart testing
* Implementing monitoring using Prometheus and Grafana
* Adding centralized logging
* Deploying the application using Infrastructure as Code

## Author

**Olusola Ayeni**

DevOps and Cloud Engineer

### Skills

* AWS
* Kubernetes
* Docker
* Helm
* Helmfile
* Jenkins
* GitHub Actions
* Terraform
* Linux
* CI/CD
* Cloud Infrastructure