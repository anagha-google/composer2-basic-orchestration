# About

This module covers code for Cloud Composer 2 DAG REST call in Python, a pattern typically used from say, an on-premise environment. <br>
Running this Python script as a User Managed Service Account (UMSA) can be achieved from outside of GCP, via GCP's Workload Identity Federation (recommended) or persisting UMSA keys in a Vault and fetching it at runtime and impersonating the UMSA. Due to complexity of setup, the module only covers code. Reach out to your GCP Customer Engineer for support. 

<br>
Note: Our recommendation is to go with Cloud Pub/Sub message triggered orchestration.  

## 1. Prerequisites

Completion of prior modules.

<hr><br>

## 2. Review the code 

Navigate to the scripts directory in the git repo cloned in Cloud Shell-
```
cd ~/composer2-basic-orchestration/00-scripts/rest-call
```
Review the script, main.py. <br>
Once again, running this Python script from outside GCP, as a User Managed Service Account (UMSA) can be achieved via GCP's Workload Identity Federation (recommended) or persisting UMSA keys in a Vault/Secrets store and fetching it at runtime and impersonating the UMSA. Due to complexity of setup, the module only covered code. 


<hr>
<br>
This concludes the module. Proceed to the next module.
