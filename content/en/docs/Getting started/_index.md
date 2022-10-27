
---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 1
date: 2021-01-15
description: >
  This page tells you how to get started with Modelix
---

In production modelix uses docker images running in a kubernetes cluster.
During development you can run the different components without docker/kubernetes.
You need a PostgreSQL database, the model server and the MPS plugin for the UI server.

## Running without kubernetes

*   Clone the project on [modelix](https://github.com/modelix/modelix) repository
*   Add `gpr.user` and `gpr.key` to [gradle.properties](https://github.com/modelix/modelix/blob/mps/2020.3/gradle.properties)

```plaintext
...

gpr.user="your_github_username" 
gpr.key="your_github_personal_access_token"
```

*   Install JAVA11.0.16 or higher. Check java and gradle compatibility matrix via [gradle.org](https://docs.gradle.org/current/userguide/compatibility.html#:~:text=A%20Java%20version%20between%208,used%20for%20compile%20or%20test.)
*   `./gradlew` in the root directory. If you are using Windows run via **cygwin** or **gitbash** terminal.
*   open the project in the **mps** folder with the MPS version specified in [mps-version.properties](https://github.com/modelix/modelix/blob/mps/2020.3/mps-version.properties)
*  Navigate to [http://localhost:33333](http://localhost:33333)

This allows you to edit the models stored locally in MPS.
Optionally, you can run the model server and connect your MPS to it:

- database
  - option 1: run the docker image
    - install docker: <https://docs.docker.com/get-docker/>
    - `./docker-build-db.sh`
    - `./docker-run-db.sh`
  - option 2: use your own PostgreSQL server
    - check the file [./db/initdb.sql](./db/initdb.sql) for the required schema
    - adjust the connection properties in [model-server/src/main/resources/org/modelix/model/server/database.properties](https://github.com/modelix/modelix/blob/master/model-server/src/main/resources/org/modelix/model/server/database.properties)
*   model server
    *   Clone the repository on [modelix.core](https://github.com/modelix/modelix.core)
    *   `cd model-server`
    *   **NOTE**: before `../gradlew run` make sure that db container is running.
    *   `../gradlew run`
- connect MPS to the model server
  - open the "Cloud" view in the bottom left corner
  - In the context menu of the root node labeled "Cloud" choose "Add Repository"
  - type `http://localhost:28101/`
  - navigate to "default tree (default)" > "data [master]" > "ROOT #1" and choose "Add Module" from the context menu
  - add a model to that module using the context menu
  - choose "Bind to Transient Module" from the context menu of the module
  - you should now see that module and the new model in the "Cloud" section at the end in the "Project" view
  - open the "Model Properties" of the new model and add at least one language dependency
  - now you are able to add new root nodes to the model from the MPS "Project" view

## Running a local kubernetes cluster

- Option 1: Docker Desktop
    - Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop)
    - Enable kubernetes in the preferences
    - Increase memory to 4-8 GB in Preferences > Resources
- Option 2: minikube
    - `minikube start --cpus=4 --memory=8GB --disk-size=40GB`
    - `eval $(minikube -p minikube docker-env)`
- `./kubernetes-modelsecret.sh`
- SSL certificate
    - `cd ssl`
    - `./generate.sh`
    - `./kubernetes-create-secret.sh`
    - `cd ..`
- Option 1: Use the latest published docker images
    - `./kubernetes-use-latest-tag.sh`
- Option 2: Build your own docker images
    - `./docker-build-all.sh`
- `./kubernetes-apply-local.sh`
- Wait ~2 minutes. You can check the status of the cluster using `kubectl get all` (or `minikube dashboard`)
- Docker Desktop: `./kubernetes-open-proxy.sh`, minikube: `minikube service proxy`
- connect MPS to the model server: same steps as described at "Running without kubernetes", but use the URL you see when you executed `./kubernetes-open-proxy.sh` / `minikube service proxy` and append "model/" (e.g. http://192.168.64.2:31894/model/)

## Running in the google cloud

- https://console.cloud.google.com/kubernetes/list?project=webmps
- Create cluster
    - Name: modelix
    - Zone: europe-west-3c
    - Pool
        - Number of nodes: 1
        - Machine type: n1-standard-2
- `gcloud container clusters get-credentials modelix`
- `./gradlew`
- `./docker-build-all.sh`
- `./docker-push-hub.sh`
- `kubectl create secret generic cloudsql-instance-credentials --from-file=./kubernetes/secrets/cloudsql.json`
- `./kubernetes-modelsecret.sh`
- SSL certificate (not supported yet)
    - `cd ssl`
    - `./generate.sh`
    - `./kubernetes-create-secret.sh`
- `./kubernetes-apply-gcloud.sh`
- `watch kubectl get all -o wide`
- `kubectl delete horizontalpodautoscaler.autoscaling/ui-autoscaler`
- Entities example
    - `cd samples/entities/`
    - `./docker-build.sh`
    - `./docker-push.sh`
    - `kubectl apply -f deployment.yaml -f service.yaml`
