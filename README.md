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
