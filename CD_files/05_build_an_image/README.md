# Building an Image

In this guide, a Docker image is built and pushed to a Image registry as part of the CI/CD pipeline. Using a ClusterTask from the Tekton Catalog called `Buildah`.

## Overview

It is imperativee to establish that your enviroment has all the necessary previous steps completed.

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