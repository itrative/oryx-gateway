# Oryx Gateway Eventhub

Oryx Gateway Eventhub is a TMFC019 Event Management Component for the TM Forum ODA.
This repo contains the helm charts for installing Oryx Gateway Evenhub in kubernetes in an ODA Canvas.
Please request a token for the images when you want to test this in your environment.

## Pre-requisites:
• A Kubernetes environment with at least 3 nodes

## Installing ODA Canvas

Oryx Gateway Eventhub assumes a previous installation of the ODA Canvas.
Install the ODA-CA canvas using the instructions found here:
https://tmforum-oda.github.io/oda-ca-docs/canvas/installation/README.html

## Requesting a security token for the docker images

A security token can be requested by sending email to 

The token is installed on kubernetes using:

`kubectl create secret docker-registry gateway-registry --docker-username=oryxgateway2 --docker-password=$SECRET --namespace components`

`kubectl create secret docker-registry gateway-registry --docker-username=oryxgateway2 --docker-password=$SECRET --namespace canvas`

## replacing the component controller

Replace the canvas component-controller to add support for automatically wiring notification to the TMFC019
This step will not be required in the future. The code for the controller update exists in the ODA-CA github but the image published by TM Forum needs to be updated.

Edit the deployment using:

`kubectl edit deployment oda-controller-ingress -n canvas`

In this file in the spec part, add the following group:

`imagePullSecrets:
- name: gateway-registry`

and replace the container image for the component-controller with:

`oryxgateway/component-controller:latest`


## installing Eventhub gateway

To add the Helm Repo run the following command:

`helm repo add oryx-gateway https://oryx-gateway.github.io/charts/`

To install the Gateway with default values (multi-broker kafka):

`helm upgrade --install --namespace components gateway oryx-gateway/oryx-gateway`

To install the gateway in a single-node mode please use the following values:

`https://github.com/oryx-gateway/charts/blob/master/gateway-extras/values-minikube.yaml`

## installing LeanRepo
Deploy lean-repo configured for a specific OpenAPI. This will install a microservice
with a generic TMF630 implementation for any OpenAPI compliant endpoint. The endpoint
will emit Notifications on the kafka topics when POST, PATCH or DELETE operation are
performed. When GET operation is performed it will query the eventually consistent
queryStore in MongoDB.
e.g.
Leanrepo (TMF629 Customer):

`helm upgrade --install --values ../helm/leanrepo/values.yaml -
-values ../helm/leanrepo/values-local.yaml --values
../helm/leanrepo/values-tmf629-customer.yaml --namespace
components customer ../helm/leanrepo`

Leanrepo (TMF637 Product):

`helm upgrade --install --values ../helm/leanrepo/values.yaml -
-values ../helm/leanrepo/values-local.yaml --values
../helm/leanrepo/values-tmf637-product.yaml --namespace
components product ../helm/leanrepo`

The yaml file contains:
- The Json-schema definition of the entity; for the notifications a standard event
header will be added.
- A json object describing the finite state machine:
  - Permitted state transitions.
  - Timestamp fields to be updated on state-transitions

## On security

The Security implementation is currently incomplete, pending creation of the security specs by the ODA Canvas team.
Currently the gateway supports several permissions that can be accessed with these tokens:

### TOPIC_ADMIN
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJPcnl4R2F0ZXdheSIsImlhdCI6MTYwOTQ1OTIwMCwiZXhwIjoxODkzNDU2MDAwLCJhdWQiOiJ3d3cuZXhhbXBsZS5jb20iLCJzdWIiOiJkZW1vQGV4YW1wbGUuY29tIiwicm9sZXMiOlsiVG9waWNBZG1pbiIsIioiXX0.C0tmCX4bX9I_8r__y3io-pRYteLVlRQVtduT5TjdtmU

### HUB_ADMIN
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJPcnl4R2F0ZXdheSIsImlhdCI6MTYwOTQ1OTIwMCwiZXhwIjoxODkzNDU2MDAwLCJhdWQiOiJ3d3cuZXhhbXBsZS5jb20iLCJzdWIiOiJkZW1vQGV4YW1wbGUuY29tIiwicm9sZXMiOlsiSHViQWRtaW4iLCIqIl19.yNqx9R2IdTpPsyKS5g-3pdiWcWyTYzJrQsRfFvBmhVE

### PUBLISHED
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJPcnl4R2F0ZXdheSIsImlhdCI6MTYwOTQ1OTIwMCwiZXhwIjoxODkzNDU2MDAwLCJhdWQiOiJ3d3cuZXhhbXBsZS5jb20iLCJzdWIiOiJkZW1vQGV4YW1wbGUuY29tIiwicm9sZXMiOlsiRXZlbnRQdWJsaXNoZXIiLCIqIl19.z-IDMZHHf5JLNh4W4qJxpCK4oOHGmU532nSrpuWvTYY

### QUERIER
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJPcnl4R2F0ZXdheSIsImlhdCI6MTYwOTQ1OTIwMCwiZXhwIjoxODkzNDU2MDAwLCJhdWQiOiJ3d3cuZXhhbXBsZS5jb20iLCJzdWIiOiJkZW1vQGV4YW1wbGUuY29tIiwicm9sZXMiOlsiUXVlcmllciIsIioiXX0.faD-qSwIKN-qH-i8dwQpb8YbDpzpmO-ifYzi54Pcic4
