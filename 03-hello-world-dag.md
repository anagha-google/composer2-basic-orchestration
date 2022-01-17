# About

This module covers creating a "Hello World" DAG, and executing the same, manually.<br>

## Duration 
~ 30 minutes or less

## Skill Level
Low

## Goal
The attendee should be able to create a very basic DAG successfully.

## Prerequisites
[GCP setup](../00-provisioning/provisioning.md)<br>

## References
[GCP Documentation Reference](https://cloud.google.com/composer/docs/composer-2/quickstart)<br>

## 2. Review the DAG script

a) From cloud shell, navigate to the directory where the script is located
```
cd ~/e2e-demo-indra/03-Cloud-Composer2/01-hello-world-dag/00-scripts/1-dag-base/
```

b) Review the "Hello World DAG" Python script available [here](../01-hello-world-dag/00-scripts/1-dag-base/hello-world-dag.py)
<br>
The DAG merely displays the DAG run ID

## 3. Deploy the DAG to Cloud Composer 2

Run the below command to deploy the DAG

```
PROJECT_ID=composer-2-playground
UMSA="agni-sa"
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
COMPOSER_ENV_NM=agni
LOCATION=us-central1

gcloud composer environments storage dags import \
--environment $COMPOSER_ENV_NM  --location $LOCATION \
--source hello-world-dag.py 
```
## 4. Switch to the Cloud Composer Aiflow Web UI and execute the DAG and check results


