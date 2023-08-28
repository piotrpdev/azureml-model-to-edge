# AzureML Model to Edge

An experiment to deploy Azure ML models to RHDE for serving

## Approach

### Experiment 1

This trail focus on Azure Automated ML and the models and conda environments that are produced bu Azure ML. Here are two examples: [bike-rentals-auto-ml](models/bike-rentals-auto-ml/) and [tensorflow-housing](models/tensorflow-housing/):

```plaintext
bike-rentals-auto-ml/
├── conda.yaml
├── MLmodel
├── model.pkl
├── python_env.yaml
└── requirements.txt

tensorflow-housing/
├── conda.yaml
├── MLmodel
├── model.pkl
├── python_env.yaml
├── requirements.txt
└── tf2model/
    ├── saved_model.pb
    └── ...
```

## Prerequisites

- Trained AzureML model including MLflow environment
- OpenShift cluster with [OpenShift Pipelines Operator](https://docs.openshift.com/container-platform/4.13/cicd/pipelines/installing-pipelines.html) installed
- OpenShift project / namespace. E.g.  `oc new-project azureml-model-to-edge`
- A repository on [Quay.io](https://quay.io/)
- S3 bucket for storing the models
- A clone of this repository

## Deploy AzureML Container build pipeline

### Provide S3 credentials

After creating the S3 bucket and uploading the models, fill out `aws-env.yaml` and create the secret:

```bash
oc create -f aws-env.yaml
```

### Deploy and run the build pipeline

```bash
oc apply -k azureml-container-pipeline/
oc create -f azureml-container-pipeline/azureml-container-pipelinerun-tensorflow-housing.yaml 
```

## Deploy Test MLflow Container image pipeline

### Quay Repository and Robot Permissions

- Create a repository (tensorflow-housing), add a robot account to push images and set write Permissions for the robot on the repository.
- Download `build-secret.yml`
- Apply build-secret. E.g.:

```bash
oc apply -f <downloaddir>/rhoai-edge-build-secret.yml 
oc secret link pipeline rhoai-edge-build-pull-secret
```

### Deploy and run the test pipeline

```bash
oc apply -k test-mlflow-image-pipeline/
oc create -f test-mlflow-image-pipeline/test-mlflow-image-pipelinerun-tensorflow-housing.yaml
```
