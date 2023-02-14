# Openshift Pipeline Trigger Demo

## Prerequisites

- Openshift 4.11+
- Openshift Red Hat Pipeline Operator
- Tekton CLI
- Gitea Github Service

## Pipeline Diagram

![pipeline-diagram](docs/pipeline-diagram.png)

## Prepare Project

```bash
oc new-project vote-app
```

Every resource should be created in the `vote-app` project.

## Create PVC

Used to share data between pipeline tasks.

```bash
oc apply -f templates/source-pvc.yaml
```

## Create Tasks

The pipeline will use two custom tasks and two build in tasks. The custom tasks are `apply-manifests` and `update-deployment`.

```bash
oc create -f templates/tasks.yaml
```

You should see the tasks.

```bash
tkn tasks ls
NAME                   DESCRIPTION   AGE
apply-manifests                      now
update-deployment                    now
```

## Create Pipeline

```bash
oc create -f templates/pipeline.yaml
```

You should see the pipeline.

```bash
tkn pipeline ls
NAME               AGE   LAST RUN   STARTED   DURATION   STATUS
build-and-deploy   now   -          -         -          -
```

## Init Source Repository

This step is optional and can be done by other means. For the all following steps and the pipeline triggers you will need:

- a Gitea user named `gitea`
- a vote-ui gitea repository
- a vote-api gitea repository

This steps can be executed automatically via:

```bash
oc create -f templates/gitea-init-task-run.yaml
```

Follow the logs.

```bash
tkn tr logs -f -a $(tkn tr ls -n vote-app | awk 'NR==2{print $1}')
```

You should see the task run.

```bash
tkn taskrun ls
NAME               STARTED   DURATION   STATUS
init-gitea-qvnd7   now       15s        Succeeded
```

## Trigger Templates

TriggerTemplate is responsible to create the Tekton Resources when it receives the Event from the EventListener.

```bash
oc create -f templates/trigger-templates.yaml
```

You should see the templates.

```bash
tkn tt ls
NAME                               AGE
vote-api-trigger-template   now
vote-ui-trigger-template    now
```

## Trigger Binding

TriggerBinding is responsible to bind the Event payload with Template Parameters. We will use two attributes from the event payload:

- `after`: the commit hash after our push is merged into the master
- `repository.clone_url`: The Git repo clone url

Values can be retrieved using JSONPath expressions.

```bash
oc create -f templates/trigger-binding.yaml
```

You should see the trigger binding.

```bash
tkn tb ls
NAME                             AGE
gitea-vote-app-trigger-binding   now
```
