# Integrate Unit Test Automation

This guide provides step-by-step instructions on integrating unit test automation into the CI/CD pipeline.

## Overview

Unit tests are automated tests that validate the behavior of a single unit of code. They are typically written by developers and run during the build process to ensure that the code behaves as expected. Unit tests are a key component of CI/CD pipelines, as they help to ensure that code changes do not break existing functionality.

## Prerequisites

- Kubernetes cluster
- Tekton Pipelines and Tekton Triggers installed on the cluster

## Step 1: Add cleanup Task and Init Task

- In the tasks.yml file, a cleanup task is adde. This task is designed to clean up a given workspace by removing all files within it. It's particularly useful in continuous integration pipelines where cleaning up temporary files or directories is necessary after a job is completed.
- When a task that causes a compilation of the Python code, it leaves behind .pyc files that are owned by the specific user. For consecutive pipeline runs, the git-clone task tries to empty the directory but needs privileges to remove these files and this cleanup task takes care of that.
- Init the task in the pipeline.yaml file

## Step 2: Add lint Task

- In the tasks.yml file, a lint task is added. This task is designed to run a linter on a Python project. It's particularly useful in continuous integration pipelines where linting is necessary to ensure code quality.

- First, install the flake8 linter from Tekton Hub:

    `tkn hub install task flake8`

This will install the flake8 task in your Kubernetes namespace.

- Next, add the lint task to your pipeline.yaml file, Including the source workspace for the lint task:

    ``` - name: lint
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: flake8
    ```
- Next, Add and modify parameters for flake8:

    In reading the documentation for the flake8 task, you see that it accepts an optional image parameter that allows you to specify your own container image. Since you are developing in a Python 3.9-slim container, you want to use python:3.9-slim as the image.

    The flake8 task also allows you to specify arguments to pass to flake8 using the args parameter. These arguments are specified as a list of strings where each string is a parameter passed to flake8. For example, the arguments `--count --statistics` would be specified as: `["--count", "--statistics"]`.



    ```
    - name: lint
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: flake8
      params:
      - name: image
        value: "python:3.9-slim"
      - name: args
        value: ["--count","--max-complexity=10","--max-line-length=127","--statistics"]
      runAfter:
        - clone
    ```

- Apply the changes to the kluster:

    `kubectl apply -f pipeline.yaml`

- Run the pipeline to test the lint task:

    ```tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog```

## Step 3: Add tests Task

There is no task for running unit tests in the Tekton Catalog, we create a custom task to run unit tests using the unit test framework
called nosetests by adding a new task called nose that uses the shared workspace for the pipeline and runs nosetests in a python:3.9-slim

- First creat a new task called nose in the tasks.yml file:

- Next, include the workspace that has the code to test. Since flake8 uses the name `source`, we use that for consistency.

- Next, create a parameter called `args` just like the `flake8` task has, giving it a `description`:, make the `type`: a string, and a `default`: with the verbose flag “-v” as the default.

- Next, specify the steps, which is only one with name `nosetests`. Have it run in a `python:3.9-slim image`.
Specify `workingDir` as the path to the workspace you defined `(i.e., $(workspaces.source.path))`.

- Next, Include a bash script to install the Python requirements and run the nosetests.

    ```---
    apiVersion: tekton.dev/v1beta1
    kind: Task
    metadata:
    name: nose
    spec:
    workspaces:
        - name: source
    params:
        - name: args
        description: Arguments to pass to nose
        type: string
        default: "-v"
    steps:
        - name: nosetests
        image: python:3.9-slim
        workingDir: $(workspaces.source.path)
        script: |
            #!/bin/bash
            set -e
            python -m pip install --upgrade pip wheel
            pip install -r requirements.txt
            nosetests $(params.args)
            ```
- Apply the changes to the kluster:

    `kubectl apply -f tasks.yaml`

## Step 4: Modify the Pipeline to Use Nose Tasks

```    - name: tests
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: nose
      params:
      - name: args
        value: "-v --with-spec --spec-color"
      runAfter:
        - lint
```
