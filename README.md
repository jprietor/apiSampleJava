# apiSampleJava

## Description

Multibranch Jenkins pipeline implementation for Continuous Blue/Green Deployment of a Spring Boot Java sample on Kubernetes cluster.

A multibranch Jenkins pipeline allows to...
- Implement a master pipeline for going to production and different pipelines for other branches to work on development/testing local environments

Blue/Green Deployment implementation in Kubernetes helps us...
- Identify problems before going to production by testing hotfixes or new features in an isolated environment identical to live environment
- Deploying the application in a high availability environment without downtime by switching environments to go live
- Easy postmortem


## Solution design
- Master branch pipeline implemented <Link to shared library>
- 2 identical deployments implemented on Minikube

<Design diagram: Commit/merge to master with new version (feature/hotfix included) -> Webhook to trigger pipeline execution -> Blue/green deployment on minikube (1 node, 3 blue pods deployment, 3 green pods deployment, 1 service api-sample-java patched to blue or green)>

- Improvement: Maven downloading dependencies on each build 

## Requirements and used tools

- Jenkins Docker container with recommended plugins 

## Observability
