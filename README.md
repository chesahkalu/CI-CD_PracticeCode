# CI/CD CD_files

This repository contains series of codes and documentation to implement and CD_files CI/CD using Tekton Pipelines.

## Contents

* 1: [CI with GitHub Actions](.github/workflows/README.md)
    - Create a GitHub workflow to run your CI pipeline
    - Add events to trigger the workflow
    - Add a job to the workflow
    - Add a job runner to the job
    - Add a container to the job runner

* 2: [Build an empty Pipeline with Tekton](CD_files/01_base_pipeline/README.md)
    - Create a base pipeline and task to echo a message.
    - Apply Tekton Tasks
    - Apply Tekton Pipeline
    - Run the pipeline
    - Apply parameters to the task and pipeline.
    - Apply additional parameters to a pipeline to clone a Git repository.

* 3: [Adding GitHub Triggers in Tekton](CD_files/02_add_git_trigger/README.md)
    - create a Tekton Trigger to cause a pipeline run from external events like changes made to a repo in GitHub.
    - Create an EventListener, a TriggerBinding and a TriggerTemplate
    - State how to trigger a deployment when changes are made to github
    
* 4: [Use Tekton CD Catalog](CD_files/03_use_tekton_catalog/README.md)
    - The Tekton community provides a wide selection of tasks and pipelines that you can use in your CI/CD pipelines, so that you do not have to write all of them yourself. Many common tasks can be found at hub.tekton.dev . Here we:
    - Use the Tekton CD Catalog to install the git-clone task
    - Describe the parameters required to use the git-clone task
    - Use the git-clone task in a Tekton pipeline to clone your Git repository
    - Use the Tekton CD Catalog to install the git-clone task

* 5: [Integrate Unit Test Automation](CD_files/04_unit_test_automation/README.md)
    - Use the Tekton CD catalog to install the flake8 task
    - Describe the parameters required to use the flake8 task
    - Use the flake8 task in a Tekton pipeline to lint your code
    - Create a test task from scratch and use it in your pipeline
