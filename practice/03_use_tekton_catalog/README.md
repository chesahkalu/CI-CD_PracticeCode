# Tekton Pipeline Setup Using Tekton Catalog, PVCs, and Workspaces

This guide provides step-by-step instructions on setting up a Tekton pipeline using the Tekton Catalog, Persistent Volume Claims (PVCs), and workspaces.

## Overview

The Tekton Catalog offers reusable tasks for your CI/CD pipelines. By using tasks from the catalog, you can leverage community-maintained components like `git-clone` for robust functionality. Workspaces in Tekton allow tasks within a pipeline to share data, and PVCs are used to create these workspaces.

## Prerequisites

- Kubernetes cluster
- Tekton Pipelines and Tekton Triggers installed on the cluster

## Step 1: Add the git-clone Task from Tekton Hub

Replace your custom `checkout` task with the `git-clone` task from the Tekton Catalog:

```tkn hub install task git-clone --version 0.8```

The git-clone task is now installed in your active namespace.

## Step 2: Create a PersistentVolumeClaim

Create a PVC to use as a workspace in your pipeline. Workspaces allow you to share data across tasks and bind to volumes in Kubernetes.

Apply the PVC to the cluster:

`kubectl apply -f pvc.yaml`

You should see the output indicating that the PVC is created:

`persistentvolumeclaim/pipelinerun-pvc created`

## Step 3: Modify the Pipeline to Use git-clone and Workspace

Edit your pipeline.yaml file:

1. Add a workspaces definition under spec: named pipeline-workspace.
2. Update the clone task to use the git-clone task and the output workspace.
3. Modify the parameter names to align with those expected by git-clone:
    - Change repo-url to url.
    - Change branch to revision.
Your pipeline.yaml should now include the workspace and reference the git-clone task.

## Step 4: Run the Pipeline

Run the pipeline using the Tekton CLI:

```
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/chesahkalu/CI-CD_PracticeCode.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

The pipeline will start, and logs will be displayed:

```
PipelineRun started: cd-pipeline-run-mndgw
Waiting for logs to be available...
```

Output from the pipeline run will include steps like cloning the repo, linting, running tests, building images, and deployment.
