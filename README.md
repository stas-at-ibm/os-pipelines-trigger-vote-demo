# Openshift Pipeline Trigger Demo

## Prerequisites

- [OpenShift Local](https://developers.redhat.com/products/openshift-local/getting-started) (4.11+)
- Openshift Red Hat Pipeline Operator
- [Tekton CLI](https://github.com/tektoncd/cli)
- Gitea Github Service

## Pipeline Diagram

![pipeline-diagram](docs/pipeline-diagram.png)

## Prepare Project

```bash
oc new-project vote-app
```

Every resource should be created in the `vote-app` project.

## Init Source Repository

This step is optional and can be done by other means. For the pipeline triggers to work you will need:

- a Gitea user named `gitea`
- a vote-ui gitea repository
- a vote-api gitea repository

If you followed the guide to setup Gitea entirely you should have the user `gitea`. Cloning the repo can be done via a task run.

```bash
oc create -f templates/clone-vote-app-task-run.yaml
```

Follow the logs.

```bash
tkn tr logs -f -a $(tkn tr ls -n vote-app | awk 'NR==2{print $1}')
```

You should see the task run.

```bash
tkn taskrun ls
NAME                   STARTED   DURATION   STATUS
clone-vote-app-qvnd7   now       15s        Succeeded
```

## Quick Setup

If you want to setup all resources in on step then use the command below and then go to the [Webhook](#webhook) step. If you want to setup every resource step by step then ignore the command below and start from [Create PVC](#create-pvc).

```bash
oc create -k kustomization.yaml
```

Delete.

```bash
oc delete -k kustomization.yaml
```

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

## Trigger Templates

Trigger template generates Tekton resources in response to events from the event listener.

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

Trigger binding binds attributes from the event payload with parameters in the trigger template. We will use two attributes from the event payload:

- `after`: commit hash of the latest pushed commit
- `repository.clone_url`: Git repo clone url

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

## Event Listener

Event listener is an inteface which recieves external events. When an event is recieved it will trigger the creation of Tekton resources like pipelines depending on how they are defined in the trigger template. Webhooks from Gitea will send their events to this resource.

```bash
oc create -f templates/event-listeners.yaml
```

You should see the event listeners.

```bash
tkn el ls
NAME                     AGE   URL                                                                AVAILABLE
gitea-webhook-vote-api   now   http://el-gitea-webhook-vote-api.vote-api.svc.cluster.local:8080   True
gitea-webhook-vote-ui    now   http://el-gitea-webhook-vote-ui.vote-api.svc.cluster.local:8080    True
```

## Webhook

You can create the webhook manually via the Gitea UI or use the webhook task run. The task run assumes that the Gitea server and k8 service are running in the `vote-app` project/namespace.

```bash
oc create -f templates/gitea-webhook-task-run.yaml
```

View task run logs.

```bash
tkn tr logs -f -a $(tkn tr ls | awk 'NR==2{print $1}')
```

## Triggers in Action

If you have `watch` installed open a new terminal and watch the pipelines run.

```bash
watch tkn pr ls
```

Clone and edit the source and push the changes.

```bash
git clone http://gitea-vote-app.apps-crc.testing/gitea/pipelines-vote-ui.git
git clone http://gitea-vote-app.apps-crc.testing/gitea/pipelines-vote-api.git
```

Make a change, commit and push the changes. A pipeline run should be triggered.

```bash
Every 2.0s: tkn pr  ls

NAME                                 STARTED      DURATION   STATUS
build-and-deploy-vote-ui-app-bzq5v   now          1m41s      Succeeded
```

You can also checkout the pipeline run in the Openshift console.

## Resources

- [Red Hat Tektop Tutorial](https://redhat-scholars.github.io/tekton-tutorial/tekton-tutorial/index.html)
- [Getting started with Openshift Pipelines](https://developers.redhat.com/learning/learn:openshift:develop-gitops/resource/resources:getting-started-openshift-pipelines)
