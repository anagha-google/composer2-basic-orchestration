# Orchestration Basics with Cloud Composer 2

The repository contains instructions, code, configuration & input files for basic hands on experience with Cloud Composer 2 and orchestrating DAGs impersonating a User Managed Service Account. The DAG is embarrassigly basic to maintain focus on orchestration and is based off of a sample in the public Google Cloud docs.<br>

This is a community contribution. Please open an issue for any bugs you run into.

## 1. Options for triggering DAGs

![Options](09-images/00-Options.png)

<br><br>

## 2. Considerations

![Recommended](09-images/02-Considerations.png)

<br><br>

![Considerations](09-images/01-Recommended.png)


## 3. Lab modules

The following are the lab modules-
[1. Prerequisites - provisioning, configuring, securing](02-prerequisites.md) <BR>
[2. Validation of environment with a "Hello World" DAG](03-hello-world-dag.md) <BR>
[3. Creation of a simple data integration pipeline](04-data-integration-dag.md) <BR>
[4. Cloud Pub/Sub message event based Cloud Composer 2 DAG execution](05-edo-pubsub-event.md) <BR>
[5. Cloud storage event based Cloud Composer 2 DAG execution](06-edo-gcs-event.md) <BR>
[6. Time event based Cloud Composer 2 DAG execution](07-edo-time.md) <BR>
[7. Code for direct REST API call to launch Cloud Composer 2 DAG](08-rest-api-call.md) <BR>

## 4. Cleanup

Delete resources created after completing the lab.
  
## 5. References

## 6. Credits

Author: Anagha Khanolkar, Google Cloud<br>
Testing: Jay O' Leary, Dr. Thomas Abraham - Google Cloud
