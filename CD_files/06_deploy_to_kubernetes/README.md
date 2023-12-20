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

## Step 5: Apply Changes and Run the Pipeline

- Ensure that you are in the `CD_files/06_deploy_to_kubernetes` directory.

- Now that you have made all of the changes to the pipeline, you can apply them to your cluster and run the pipeline.

1. Apply the changes to the pipeline by running `kubectl apply -f pipeline.yaml`

2. Start the pipeline by running :

When you start the pipeline, you now need to pass in the `app-name` parameter, which is the name of the application to deploy.

Now, start the pipeline to see your new deploy task run. Use the Tekton CLI `pipeline start` command to run the pipeline, passing in the parameters `repo-url`, `branch`, `app-name`, and `build-image` using the `-p` option. Specify the workspace `pipeline-workspace` and persistent volume claim `pipelinerun-pvc` using the `-w` option:

```
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch=main \
    -p app-name=hitcounter \
    -p build-image=image-registry.openshift-image-registry.svc:5000/$SN_ICR_NAMESPACE/tekton-lab:latest \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

## Step 6: Check the Deployment

- You can check the status of the deployment using the Tekton CLI `tkn pipelinerun list` command:

`tkn pipelinerun list`

- You should see the following output:

```
NAME             STARTED         DURATION    STATUS
cd-pipeline-run  2 minutes ago   2 minutes   Succeeded
```

- You can also check the status of the deployment using the `oc` command:

`oc get deployment`

- You should see the following output:

```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hitcounter   1/1     1            1           2m
```

- Now, check to see if the deployment is running. Use the `kubectl` command to check that your deployment is in a running state.

`kubectl get all -l app=hitcounter`

- You should see the following output:

```
NAME                              READY   STATUS    RESTARTS   AGE
pod/hitcounter-7c9f95784d-rk4tf   1/1     Running   0          2m46s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hitcounter   1/1     1            1           2m46s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/hitcounter-7c9f95784d   1         1         1       2m46s
```