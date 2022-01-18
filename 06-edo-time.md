# About

In this module, we will just review how to set the schedule for DAG execution based on time, within the DAG code itself.

## 1. Scheduling in the DAG code

In the code fragment below, the last like sets the schedule. Apache Airflow will execute the DAG at the time interval set. Catchup can be completed as well by setting DAG start date to a prior date.

#### 1. Basic schedule with time interval of every 15 minutes
```
with models.DAG(
        'hello_world_dag',
        'catchup=True',
        default_args=default_args,
        schedule_interval=datetime.timedelta(minutes=15)) as dag:
```

#### 2. Catchup
Start date, along with ["catchup"](https://airflow.apache.org/docs/apache-airflow/stable/dag-run.html#catchup) boolean will complete the catch up DAG materialization.

```
YESTERDAY = datetime.datetime.now() - datetime.timedelta(days=1)

default_args = {
    'owner': ...
    'start_date': YESTERDAY,
}
```

#### 3. Complete "Hello World" DAG

Navigate in Cloud Shell to -
```
cd ~/composer2-basic-orchestration/00-scripts/hello-world-dag/4-dag-time-orchestrated
```

Review the script "hello-world-dag-scheduled.py" 

```
# Docs: https://cloud.google.com/composer/docs/composer-2/quickstart

import datetime

from airflow import models
from airflow.operators import bash

# If you are running Airflow in more than one time zone
# see https://airflow.apache.org/docs/apache-airflow/stable/timezone.html
# for best practices
YESTERDAY = datetime.datetime.now() - datetime.timedelta(days=1)

default_args = {
    'owner': 'Composer Sample',
    'depends_on_past': False,
    'email': [''],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': datetime.timedelta(minutes=5),
    'start_date': YESTERDAY,
}

with models.DAG(
        'hello_world_dag_scheduled',
        'catchup=True',
        default_args=default_args,
        schedule_interval=datetime.timedelta(minutes=15)) as dag:

    # Print the dag_run id from the Airflow logs
    print_dag_run_conf = bash.BashOperator(
        task_id='print_dag_run_conf', bash_command='echo "Hello World! I am a time scheduled DAG, and the current DAG run ID = " {{ dag_run.id }}'
    )
```
<br>
<hr>

## 2. Deploy the DAG

```
# Navigate to the DAG directory
cd ~/composer2-basic-orchestration/00-scripts/hello-world-dag/4-dag-time-orchestrated


# Declare variables
PROJECT_ID=composer-2-playground
UMSA="agni-sa"
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
COMPOSER_ENV_NM=cc2-agni
LOCATION=us-central1

# Deploy DAG

gcloud composer environments storage dags import \
--environment $COMPOSER_ENV_NM  --location $LOCATION \
--source hello-world-dag-scheduled.py \
--impersonate-service-account $UMSA_FQN

```

## 3. Review the DAg execution in Airflow UI

You should see the DAG start executing right away, and run periodically.

This concludes the module
