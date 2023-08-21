# InnoTech Demo

## Initialise  Terraform

```cmd

terraform init

```

## Terraform Plan

```cmd

terraform plan

```

## Terraform Apply

```cmd

terraform apply -auto-approve

```

## Connect to GKE cluster

```cmd

gcloud container clusters get-credentials <GKEClusterName> --region asia-southeast1 --project <ProjectName>

```

## Connect to GKE cluster

```cmd

git clone https://github.com/stegacteam/appdemo

```

## Create Namespace, market-demo

```bash

kubectl create ns market-demo

```

## Deploy App container

```bash

#kubectl apply --recursive --filename
#kubectl apply --recursive -f <Folder>

kubectl apply -f api-deployment.yaml

kubectl apply -f api-ingress.yaml

kubectl apply -f api-service.yaml

kubectl apply -f web-deployment.yaml

kubectl apply -f web-service.yaml

```
