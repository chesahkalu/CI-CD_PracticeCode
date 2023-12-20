# Deploy to Kubernetes / OpenShift

In this guide we will deploy our application to a Kubernetes cluster using Tekton Pipelines.

## Overview and Prerequisites

- Kubernetes cluster
- Tekton Pipelines and Tekton Triggers installed on the cluster
- It is imperativee to establish that your enviroment has all the necessary previous steps completed

## Step 1: Check for ClusterTasks

- Knowing that you want to deploy to OpenShift, you search Tekton Hub for “openshift” and you see there is a task called `openshift-client` that will execute OpenShift commands on your cluster. You decide to use the `openshift-client` task in your pipeline to deploy your image.

- Knowing that you want to deploy to OpenShift, you search Tekton Hub for “openshift” and you see there is a task called `openshift-client` that will execute OpenShift commands on your cluster. You decide to use the openshift-client task in your pipeline to deploy your image.

- Check that the `openshift-client` task is installed as a ClusterTask using the Tekton CLI:

`tkn clustertask ls`

## Step 2: Reference the openshift-client task in the Pipeline

- You must now reference the new `openshift-client` ClusterTask that you want to use in the `deploy` pipeline task.

```
    - name: deploy
      taskRef:
        name: openshift-client
        kind: ClusterTask
```

## Step 3: Update the Task Parameters

- The documentation for the `openshift-client` task details that there is a parameter named `SCRIPT` that you can use to run `oc` commands. Any command you can use with `kubectl` can also be used with `oc`. This is what you will use to deploy your image.

The command to deploy an image on OpenShift is:

`oc create deployment {name} --image={image-name}`

- Since you might want to reuse this pipeline to deploy different applications, you should make the deployment name a parameter that can be passed in when the pipeline runs. You already have the image name as a parameter from the build task that you can use.

```
    - name: deploy
      taskRef:
        name: openshift-client
        kind: ClusterTask
      params:
        - name: SCRIPT
          value: "oc create deploy $(params.app-name) --image=$(params.build-image)"
```

## Step 4: Update the Pipeline Parameters

Now that you are passing in the `app-name` parameter to the deploy task, you need to go back to the top of the `pipeline.yaml` file and add the parameter there so that it can be passed into the pipeline when it is run.

```
spec:
  params:
    - name: app-name
```

