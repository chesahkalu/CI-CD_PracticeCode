# Setting Up GitHub Triggers in Tekton

This guide walks through the process of setting up GitHub triggers in Tekton, enabling automatic pipeline execution in response to GitHub events like `push` or `pull_request`.

## Overview

GitHub triggers in Tekton are used to automate the execution of pipelines in response to events in a GitHub repository. This is achieved using Tekton Triggers, which consist of `EventListeners`, `TriggerBindings`, and `TriggerTemplates`.

## Prerequisites

- A Kubernetes cluster
- Tekton Pipelines and Tekton Triggers installed on the cluster

## Understanding Tekton Triggers

- **EventListeners**: Wait for events and process them when received.
- **TriggerBindings**: Extract information from events and pass them to TriggerTemplates.
- **TriggerTemplates**: Define what to do with extracted information, typically creating PipelineRuns.

## Applying Tekton Triggers

1. **Apply the EventListener**:

   This listens for GitHub webhook events.

    `kubectl apply -f eventlistener.yaml`
    
    Check the EventListener:
    
    `kubectl get eventlisteners` or `tkn trigger ls`

2. **Apply the TriggerBinding**:
    
    This extracts data from the GitHub event.
    
    `kubectl apply -f triggerbinding.yaml`
    
    Check the TriggerBinding:
    
    `kubectl get triggerbindings`

3. **Apply the TriggerTemplate**:
    
    This uses data from the TriggerBinding to create PipelineRuns.
    
    `kubectl apply -f triggertemplate.yaml`
    
    Check the TriggerTemplate:
    
    `kubectl get triggertemplates`

## Creating a GitHub webhook

- To trigger the pipeline on GitHub events:

1. Go to your GitHub repository settings.
2. Navigate to Webhooks and add a new webhook.
3. Set the Payload URL to your EventListener's exposed URL.
4. Choose the events you want to trigger the pipeline (e.g., push, pull request).
5. Save the webhook.

## Test the setup

- Make a push or create a pull request to your GitHub repository.
- The EventListener should trigger the pipeline based on the GitHub event.
- Check the Pipeline logs:
    
    `tkn pipelinerun logs -f`