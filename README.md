# SRE for apiSampleJava

## Description

**Multibranch Jenkins pipeline** implementation for **Continuous Blue/Green Deployment** of a Spring Boot Java sample on Kubernetes cluster (minikube).

**Blue/Green Deployment** implementation in Kubernetes is a release management strategy that reduces risk and minimizes downtime. It uses two production environments, known as Blue and Green, to provide reliable testing, continuous no-outage upgrades, and instant rollbacks. One of them go live and the other is held in reserve. It provides:
- Identify problems before going to production by testing hotfixes or new features in an isolated environment identical to live environment (testing parity)
- Deploying the application in a high availability environment without downtime by switching environments to go live
- Instant rollback (the cut-over works both ways)
- Hot standby for disaster recovery (as long as we hadn't put Blue and Green on the same availability zone)
- Postmortem: Rollbacks always leave the failed deployment intact for analysis

A multibranch Jenkins pipeline allows to implement a master pipeline for going to production and different pipelines for other branches to work on development/testing local environments.

This project only focus on master branch to play with a Blue/Green deployment sample. See **Jenkins shared library** used and **master pipeline** implemented for **apiSampleJava** here: [pipelineForApiSampleJavaMaster](https://github.com/jprietor/jenkinsfile-shared-library/tree/main/vars/pipelineForApiSampleJavaMaster.groovy)

## Content

- **Jenkinsfile:** Keeps pipeline parameters/configuration for this branch to run specific pipeline through [pipelineSelector](https://github.com/jprietor/jenkinsfile-shared-library/tree/main/vars/pipelineSelector.groovy) from shared library ([pipelineForApiSampleJavaMaster](https://github.com/jprietor/jenkinsfile-shared-library/tree/main/vars/pipelineForApiSampleJavaMaster.groovy))
- **Dockerfile:** Build apiSampleJava docker image from scratch with Maven contenerized (*TODO: Avoid Maven dependency downloads -Up to 3 minutes in Build stage- building docker image by mounting cache volumen from Docker host or copying target jars from host*)
- **manifests/:** Configuration files for Kubernetes objects
  - *deployment.yaml* and *service.yaml*: ApiSampleJava deployment and service templates
  - *service-patch.yaml*: ApiSampleJava service patch template to apply for Blue/Green switch
  - *serviceAccount.yaml*: Service account for 'deployer' role to access cluster for automation

## Solution design

Master branch pipeline implemented [pipelineForApiSampleJavaMaster](https://github.com/jprietor/jenkinsfile-shared-library/tree/main/vars/pipelineForApiSampleJavaMaster.groovy) with these stages:
1. **Checkout:** Get code from GitHub repository
2. **Compile:** Run maven package (from Jenkins node) skipping tests
3. **Test:** Run tests (from Jenkins node)
4. **Build:** Build Docker image with env parameters
5. **Deliver:** Push Docker image to DockerHub
6. **Deploy:** (*TODO: Split in more stages to keep it simple an avoid code for control flow*)
    * Create api-sample-java service (if there are no deployments)
    * Delete oldest deployment (if there are already 2 deployments: Blue and Green) (*TODO: Consider to delete deployment that is not in production instead of the older one...*)
    * New version deployment
7. **Blue/Green switch:** Interactive stage with manual confirmation for going live with the new version deployment
    * Allowed to restart the pipeline from this stage just to switch Blue/Green environments for instant rollback or go live
    * Patch service to switch and go live with deployment in reserve

When a developer commits/merges to master with a new version -feature/hotfix included and Jenkinsfile VERSION config parameter upgraded-, a GitHub webhook will trigger master pipeline execution (as long as Jenkins is exposed to Internet) and a new Blue/green deployment will be performed on minikube (1 node, 3 Blue pods deployment, 3 Green pods deployment and 1 service api-sample-java patched to blue or green deployment).

New release deployment | Blue/Green switch
---------------------- | -----------------
![cicd-blue-green.jpg](https://user-images.githubusercontent.com/79945638/110260487-c8820a00-7fac-11eb-8847-3f4760824ccf.png) | ![cicd-blue-green-2.jpg](https://user-images.githubusercontent.com/79945638/110260599-34647280-7fad-11eb-8fb4-ae93a6401d9f.png)

## Use and technology requirements

Requirements and tools used to run this sample on 1 Docker host:

- **Minikube** (local **Kubernetes cluster**) and kubectl installed in Docker host and started in Docker bridge network:
  ```
    $ minikube start --driver=docker --network=bridge
  ```

- **Jenkins** Docker container with:
  - Plugins: Recommended plugins from installation + Maven integration + docker-plugin + docker-workflow
  - 2 Nodes:
    - main (Jenkins master): For schedule, checkout and maven stages
    - docker (Docker host): For build, deliver and deploy stages (docker image and kubectl commands from docker network)

Access to live environment (with minikube ip: 172.17.0.4):
```
  $ curl http://172.17.0.4:30000
```

Port forward to access/test reserve environment:
```
  $ kubectl port-forward deployment/api-sample-java-<reserve_version> 8080:8080
  (from other terminal) $ curl http://localhost:8080
```

## Observability

To enable quick and easy debug of the solution:
- **JMX (Java Management Extensions)** should be enabled to get metrics from the Java application
- **ELK** could provide a good stack for logs management and **Elastic APM** has good integration with JMX
- **Prometheus** can ingest JMX data via the [JMX exporter](https://github.com/prometheus/jmx_exporter) which exposes metrics in Prometheus format

### Service-Level Indicators and Objetives (SLIs and SLOs)

Proposed SLIs to start measuring from external monitoring system (**Prometheus, Pandora FMS**):

Type of service | Type of SLI | Description | Implementation
--------------- | ----------- | ----------- | --------------
Request-driven | Availability | Number of successful HTTP requests / total HTTP requests (success rate) | sum(rate(http_requests_total{host="api-sample", status!~"5.."}[7d])) / sum(rate(http_requests_total{host="api-sample"}[7d])
Request-driven | Latency | Number of HTTP requests that completed successfully in < x ms / total HTTP requests requests | http_request_duration_seconds{host="api", le="0.1"} -cumulative histogram...- histogram_quantile(0.9, rate(http_request_duration_seconds_bucket[7d]))

After several weeks the previous metrics (total HTTP requests, successful HTTP requests, latency 90th percentile) should show numbers to calculate SLOs, error budget and decide next steps and priorities for reliability continuous improvement.

SLOs example:

Type | Objective
---- | ---------
Availability | 99%
Latency | 90% of requests < 250 ms
