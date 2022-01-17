# About

This module covers creating a "Hello World" DAG, and executing the same, manually, as the user managed service account created in the provisioning prerequisite module.<br>

## Duration 
~ 30 minutes or less

## Skill Level
Low

## Goal
To test if the Cloud Composer 2 environment is successfully provisioned.

## Prerequisites
[provisioning](../02-prerequisites.md)<br>

## References
[GCP Documentation Reference](https://cloud.google.com/composer/docs/composer-2/quickstart)<br>

## 1. Variables for the lab module

In cloud shell, declare the below-
```
PROJECT_ID=composer-2-playground
UMSA="agni-sa"
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
COMPOSER_ENV_NM=cc2-agni
LOCATION=us-central1
```

## 2. Review the DAG script in the git repo cloned 

a) In cloud shell, navigate to the directory where the script is located
```
cd ~/composer2-basic-orchestration/00-scripts/hello-world-dag
```

b) Review the "Hello World DAG" Python script available [here](../01-hello-world-dag/00-scripts/1-dag-base/hello-world-dag.py)
<br>
The DAG merely displays the DAG run ID

## 3. Deploy the "Hello World" DAG to Cloud Composer 2

Run the below command to deploy the DAG

```
gcloud composer environments storage dags import \
--environment $COMPOSER_ENV_NM  --location $LOCATION \
--source hello-world-dag.py 
```

This will copy the DAG Python script to the Cloud Composer GCS DAG bucket, and will get imported and execute immediately (as per the code).

## 4. Switch to the Cloud Composer Airflow Web UI and execute the DAG and check results

The deployment automatically launches a DAG.
Navigate to the DAG run and go to logs, you shuld see something like this-
```
INFO - Hello World!, DAG run ID =  2
```


## 5. What's next?

We'll deploy a more realistic DAG that involves reading files from GCS, transforming via a Cloud Dataflow pipeline and loading into BigQuery, with the goal to run it as a user managed service account
