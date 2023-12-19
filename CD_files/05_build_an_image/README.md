# Building an Image

In this guide, a Docker image is built and pushed to a Image registry as part of the CI/CD pipeline. Using a ClusterTask from the Tekton Catalog called `Buildah`.

## Overview and Prerequisites

- Kubernetes cluster
- Tekton Pipelines and Tekton Triggers installed on the cluster
- It is imperativee to establish that your enviroment has all the necessary previous steps completed:

Issue the following commands to install everything from the previous labs.

1. Run `tkn hub install task git-clone` to install the git-clone task from the Tekton Catalog.
Note: If the above command returns a error due to Tekton Version mismatch, please run the below command to fix this:

`kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml`


2. Run `tkn hub install task flake8` to install the flake8 task from the Tekton Catalog.

3. Run `kubectl apply -f tasks.yaml` to apply all previous tasks to your cluster.

4. Check that you have all of the previous tasks installed:
`tkn task ls`

you should see the following output:

```
NAME               DESCRIPTION              AGE
cleanup            This task will clean...  2 minutes ago
git-clone          These Tasks are Git...   2 minutes ago
flake8             This task will run ...   1 minute ago
echo                                        46 seconds ago
nose                                        46 seconds ago
```

## Step 1: Check for ClusterTasks

Our previous pipeline currently has a placeholder for a `build` step that uses the `echo` task. Now it is time to replace it with a real image builder.

You search Tekton Hub for the word "build" and you see there is a task called `buildah` that will build images so you decide to use the `buildah` task in your pipeline to build your code.

Instead of installing it yourself, you first check the ClusterTasks in your cluster to see if it already exists. Luckily, the OpenShift environment you are using already has `buildah` installed as a ClusterTask. A ClusterTask is installed cluster-wide by an administrator and anyone can use it in their pipelines without having to install it themselves.

Check that the `buildah` task is installed as a ClusterTask using the Tekton CLI.

`tkn clustertask ls`

## Step 2: Add a Workspace to the Pipeline Task

Now update the pipeline to use the `buildah` task to build your code. 
In reading the documentation for the `buildah` task, you see that it requires a workspace named `source` to be passed in.

```
    - name: build
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
```

## Step 3: Reference the buildah Task in the Pipeline

Now, you need to reference the new buildah task that you want to use. In the previous steps, you simply changed the name of the reference to the task. But since the buildah task is a `ClusterTask`, you need to add the statement `kind: ClusterTask` under the name so that Tekton knows to look for a ClusterTask and not a regular Task.

```
      taskRef:
        name: buildah
        kind: ClusterTask
```

## Step 4: Update the Task Parameters

- The documentation for the buildah task details several parameters but only one of them is required. You need to use the `IMAGE` parameter to hold the name of the image you want to build.
Since you might want to reuse this pipeline to build different images, you will make it a variable parameter that can be passed in when the pipeline runs. To do this, you need to change it here and add a parameter to the Pipeline itself.

- Change the `message` parameter to `IMAGE` and specify the value of `$(params.build-image):`


- Now that you are passing in the `IMAGE` parameter to this task, you need to go back to the top of the `pipeline.yaml` file and add the parameter there so that it can be passed into the pipeline when it is run.

- Add a parameter named `build-image` to the existing list of parameters at the top of the pipeline under `spec.params`.

```
spec:
  params:
    - name: build-image
```

## Step 5: Apply Changes and Run the Pipeline

Now that you have made all of the changes to the pipeline, you can apply them to your cluster and run the pipeline.

1. Apply the changes to the pipeline by running `kubectl apply -f pipeline.yaml`

2. Next, make sure that the persistent volume claim for the workspace exists by applying it using `kubectl`: `kubectl apply -f pvc.yaml`

## Step 6: Start the Pipeline

When you start the pipeline, you need to pass in the build-image parameter, which is the name of the image to build.

Now, start the pipeline to see your new build task run. Use the Tekton CLI `pipeline start` command to run the pipeline, passing in the parameters `repo-url`, `branch`, and `build-image` using the `-p` option. Specify the workspace `pipeline-workspace` and volume claim `pipelinerun-pvc` using the `-w`  option:

```
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch=main \
    -p build-image=image-registry.openshift-image-registry.svc:5000/$SN_ICR_NAMESPACE/tekton-lab:latest \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```


## Step 7: Check the Pipeline Run Status

- You can check the status of the pipeline run using the Tekton CLI `tkn pipelinerun list` command:

`tkn pipelinerun list`

- you should see the following output:

```
NAME             STARTED         DURATION    STATUS
cd-pipeline-run  2 minutes ago   2 minutes   Succeeded
```

- You can check the logs of the last run with:

`tkn pipelinerun logs cd-pipeline-run`
