# Create a base pipeline

This folder holds the files for create a simple base pipeline for CI/CD practice.

## Tekton

Tekton is a powerful and flexible open-source framework for creating CI/CD systems, allowing developers to build, test, and deploy across cloud providers and on-premise systems. It provides Kubernetes-style resources for declaring CI/CD-style pipelines.

## Prerequisites

- Kubernetes cluster
- Tekton Pipelines installed on the cluster

## Applying Tekton Tasks

Tekton Tasks are the building blocks of Tekton Pipelines. They are reusable, loosely coupled units of work that can be combined in a pipeline to build your CI/CD workflows. The Tekton Tasks in this repository are defined in the [tasks.yaml](tasks.yaml) with a well commented task.

- First, apply the Tekton Tasks to your cluster.

`kubectl apply -f tasks.yaml`

## Applying Tekton Pipeline

Tekton Pipelines are a way to build CI/CD pipelines using Tekton Tasks. They are defined in the [pipeline.yaml](pipeline.yaml) with a well commented pipeline.

- Next, apply the Tekton Pipeline to your cluster.

`kubectl apply -f pipeline.yaml`

## Run the pipeline

- Finally, Run the pipeline using the Tekton CLI:

```
tkn pipeline start cd-pipeline \
    --showlog \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main"
```

The pipeline will run and you will see the output in the terminal like this:

```
PipelineRun started: cd-pipeline-run-wvfzx
Waiting for logs to be available...
[clone : checkout] Cloning into 'wtecc-CICD_PracticeCode'...

[lint : echo-message] Calling Flake8 linter...

[tests : echo-message] Running unit tests with PyUnit...

[build : echo-message] Building image for https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git ...

[deploy : echo-message] Deploying main branch of https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git ...
```
