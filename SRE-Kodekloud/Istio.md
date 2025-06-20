

## What is Istio

_Istio is a service Mesh popular solution to manages communication between microservices_


## What is the main features


Istio add a extra layer to networking increasing security to your cluster and end to end services and pod communication.

Istio also add retry logic to increase system reliability

Istio also add monitoring collecting metrics 

Istio can be a sidecar proxy in the pod working beside of the main application container

## Traffic Split

Istio can also split the work between different deployment version working as Canarian deployment.

it allow you having 2 or more versions of your application and redirect 10% of traffic to the new version of deployment. Increasing by 10% each day you gain confidence on your new application

## Configuration

Istio uses Yaml files to get deployed
It extend the kube api capability 
All configuration like traffic routing, which service can communicate, traffic split, retry rules can be configured on yaml file for the Istio configuration.

- Virtual Service
- Destination Rules

We create CRD's and Istiod convert these rules.

## Ingress Gateway

Istio can also works as a Gateway for your cluster improving security who can access your cluster, once it grants access to your cluster Istio will also validate your access for each pod, it double security.

