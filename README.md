# Orchestration Basics with Cloud Composer 2

The repository contains instructions, code, configuration & input files for basic hands on experience with Cloud Composer 2 and orchestrating DAGs impersonating a User Managed Service Account. The DAG is embarrassigly basic to maintain focus on provisioning and orchestration and is based off of a sample in the public Google Cloud docs.<br>

This is a community contribution and our goal was get a functionally adequate lab rolled out (versus excellent and automated) that demystifies provisioning, and event driven orchestration. Please open an issue for any bugs you run into. <br><br>

Note for Google Customer Engineers: This lab covers setup in *Argolis*.<br>

<hr>

## 1. Options for triggering DAGs

![Options](09-images/00-Options.png)

<hr>

## 2. Considerations

![Recommended](09-images/02-Considerations.png)

<hr>

## 3. Recommendation

![Considerations](09-images/01-Recommended.png)

<hr>

## 4. Lab modules

The following are the lab modules-<br>
[1. Prerequisites - provisioning, configuring, securing](02-prerequisites.md) <BR>
[2. Validation of environment with a "Hello World" DAG](03-hello-world-dag.md) <BR>
[3. Creation of a simple data integration pipeline](04-data-integration-dag.md) <BR>
[4. Cloud Pub/Sub message event based Cloud Composer 2 DAG execution](05-edo-pubsub-event.md) <BR>
[5. Cloud storage event based Cloud Composer 2 DAG execution](06-edo-gcs-event.md) <BR>
[6. Time event based Cloud Composer 2 DAG execution](07-edo-time.md) <BR>
[7. Code for direct REST API call to launch Cloud Composer 2 DAG](08-rest-api-call.md) <BR>

  <hr>
  
## 5. Cleanup

Delete resources created after completing the lab.
  
  <hr>
  
## 6. GCP Documentation References

[Cloud Composer 2 Guide](https://cloud.google.com/composer/docs/composer-2/quickstart)<BR>
[Triggering Cloud Composer 2 DAGs with Cloud Functions](https://cloud.google.com/composer/docs/composer-2/triggering-with-gcf)<BR>
[Cloud Composer 2 - Dataflow Template Operator](https://cloud.google.com/composer/docs/how-to/using/using-dataflow-template-operator)
  
  <hr>

## 7. Credits

Author: Anagha Khanolkar, Google Cloud<br>
Testing: Jay O' Leary, Dr. Thomas Abraham - Google Cloud
