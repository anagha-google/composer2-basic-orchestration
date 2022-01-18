# About

In this module, we will just review how to set the schedule for DAG execution based on time, within the DAG code itself.

## Scheduling in the DAG code

In the code fragment below, the last like sets the schedule. Apache Airflow will execute the DAG at the time interval set. Backfilling can be completed as well by setting DAG start date to a prior date.

#### 1. Basic schedule with time interval of every 15 minutes
```
with models.DAG(
        'hello_world_dag',
        'catchup=True',
        default_args=default_args,
        schedule_interval=datetime.timedelta(minutes=15)) as dag:
```

#### 2. Backfill
Start date, along with "catchup" boolean will backfill.

```
YESTERDAY = datetime.datetime.now() - datetime.timedelta(days=1)

default_args = {
    'owner': ...
    'start_date': YESTERDAY,
}
```

#### 3. Complete "Hello World" DAG

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
This concludes the module
