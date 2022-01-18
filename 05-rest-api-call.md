# About

This module shows how to run the DAG via a REST call, while impersonating a user managed service acccount

## 1. Prerequisites

Completion of prior modules.

## 2. Variables 

In Cloud Shell paste the below-
```


PROJECT_ID=composer-2-playground
PROJECT_NUMBER=508201578739 # Replace with yur project number

UMSA="agni-sa"
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
UMSA_KEY_FILE=~/umsa-key-file

ADMIN_FQ_UPN="admin@akhanolkar.altostrat.com" # Replace with your admin UPN


VPC_NM=composer-2-vnet
VPC_FQN=projects/$PROJECT_ID/global/networks/$VPC_NM
SUBNET_NM=composer-2-snet

REGION=us-central1

COMPOSER_ENV_NM=cc2-agni
USE_PUBLIC_IPS=true

DATAFLOW_SUBNET="https://www.googleapis.com/compute/v1/projects/$PROJECT_ID/regions/$LOCATION/subnetworks/$SUBNET_NM"

SRC_FILE_STAGING_BUCKET_PATH=gs://$PROJECT_ID-dag-input-files
BQ_DATASET_NM=average_weather_ds
BQ_TABLE_NM=average_weather

AIRFLOW_URI=`gcloud composer environments describe $COMPOSER_ENV_NM \
    --location $REGION \
    --format='value(config.airflowUri)'`
```

## 3. Review the code 

Navigate to the scripts directory in the git repo cloned in Cloud Shell-
```
cd ~/composer2-basic-orchestration/00-scripts/rest-call
```

Review the script, main.py, and check for where the following  -
```

```
## 4. Generate the UMSA key file
Note - this is for pure testing purposes.
Prefer Workload Identity Federation over UMSA keys that are clearly a security risk

```
rm $UMSA_KEY_FILE

gcloud iam service-accounts keys create $UMSA_KEY_FILE \
    --iam-account=$UMSA@$PROJECT_ID.iam.gserviceaccount.com 
```
