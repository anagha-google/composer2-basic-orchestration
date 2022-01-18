# About

This module includes all prerequisites for the orchesration lab-<br>
[1. Declare variables]()<br>
[2. Enable Google APIs](02-prerequisites.md#2-enable-google-apis)<br>
[3. Create a VPC & a subnet](02-prerequisites.md#3-create-a-vpc--a-subnet)<br>
[4. Create firewall rules](02-prerequisites.md#4-create-firewall-rules)<br>
[5. Implement organizational policies](02-prerequisites.md#5-implement-organizational-policies)<br>
[6. Create a User Managed Service Account](02-prerequisites.md#6-create-a-user-managed-service-account)<br>
[7. Grant general IAM permissions](02-prerequisites.md#7-grant-general-iam-permissions)<br>
[8. Grant IAM permissions specific to Cloud Composer](02-prerequisites.md#8-grant-iam-permissions-specific-to-cloud-composer)<br>
[9. Grant IAM Permissions specific to Cloud Functions](02-prerequisites.md#9-grant-iam-permissions-specific-to-cloud-functions)<br>
[10. Grant IAM Permissions specific to Cloud Dataflow](02-prerequisites.md#10-grant-iam-permissions-specific-to-cloud-dataflow)<br>
[11. Grant IAM Permissions specific to Cloud Storage](02-prerequisites.md#11-grant-iam-permissions-specific-to-cloud-storage)<br>
[12. Provision Clone Composer 2, impersonating the UMSA identity](02-prerequisites.md#12-provision-clone-composer-2-impersonating-the-umsa-identity)<br>
[13. Clone this hands-on-lab's git repo](02-prerequisites.md#13-clone-this-hands-on-labs-git-repo)<br>
...

## 1. Declare varibles 

We will use these throughout the lab. Run the below-
```
PROJECT_ID=composer-2-playground
PROJECT_NUMBER=508201578739 # Replace with yur project number

UMSA="agni-sa"
UMSA_FQN=$UMSA@$PROJECT_ID.iam.gserviceaccount.com
ADMIN_FQ_UPN="admin@akhanolkar.altostrat.com" # Replace with your admin UPN


VPC_NM=composer-2-vnet
VPC_FQN=projects/$PROJECT_ID/global/networks/$VPC_NM
SUBNET_NM=composer-2-snet

REGION=us-central1

COMPOSER_ENV_NM=cc2-agni
```


## 2. Enable Google APIs

From cloud shell, run the below-
```
gcloud services enable orgpolicy.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable containerregistry.googleapis.com
gcloud services enable composer.googleapis.com
gcloud services enable monitoring.googleapis.com 
gcloud services enable cloudtrace.googleapis.com 
gcloud services enable clouddebugger.googleapis.com 
gcloud services enable bigquery.googleapis.com 
gcloud services enable storage.googleapis.com
gcloud services enable cloudfunctions.googleapis.com
gcloud services enable pubsub.googleapis.com
gcloud services enable dataflow.googleapis.com
```

## 3. Create a VPC & a subnet

Launch cloud shell, change scope to the project you created (if required), and run the commands below to create the networking entities required for the hands on lab.


#### 3.1. Create a VPC


```
gcloud compute networks create $VPC_NM \
    --subnet-mode=custom \
    --bgp-routing-mode=regional \
    --mtu=1500
```

b) List VPCs with:
```
gcloud compute networks list
```

c) Describe your network with:
```
gcloud compute networks describe $VPC_NM
```

#### 3.2. Create a subnet for composer


```
gcloud compute networks subnets create $SUBNET_NM \
     --network=$VPC_NM \
     --range=10.0.0.0/24 \
     --region=$REGION \
     --enable-private-ip-google-access
```

## 4. Create firewall rules
a) Intra-VPC, allow all communication

```
gcloud compute firewall-rules create allow-all-intra-vpc --project=$PROJECT_ID --network=$VPC_FQN --description="Allows\ connection\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ custom\ protocols." --direction=INGRESS --priority=65534 --source-ranges=10.0.0.0/20 --action=ALLOW --rules=all
```

b) Allow SSH

```
gcloud compute firewall-rules create allow-all-ssh --project=$PROJECT_ID --network=$VPC_FQN --description="Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22." --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22
```

c) Create a firewall rule to allow yourself to ingress

Replace with your IP address below-
```
gcloud compute --project=$PROJECT_ID firewall-rules create allow-all-to-my-machine --direction=INGRESS --priority=1000 --network=$VPC_NM --action=ALLOW --rules=all --source-ranges=xx.xxx.xx.xx
```

## 5. Implement organizational policies

Applicable for Google Customer Engineers working in Argolis for this hands on lab.<br>
Modify/Apply/may not apply for your environment - check with your administrator.


### 5.1.  Create variables for use further in the rest of project in cloud shell
Covered in section 1.0

### 5.2. Relax require OS Login
```
rm os_login.yaml

cat > os_login.yaml << ENDOFFILE
name: projects/${PROJECT_ID}/policies/compute.requireOsLogin
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy os_login.yaml 

rm os_login.yaml
```

### 5.3. Disable Serial Port Logging

```

rm disableSerialPortLogging.yaml

cat > disableSerialPortLogging.yaml << ENDOFFILE
name: projects/${PROJECT_ID}/policies/compute.disableSerialPortLogging
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy disableSerialPortLogging.yaml 

rm disableSerialPortLogging.yaml

```
### 5.4. Disable Shielded VM requirement

```

shieldedVm.yaml 

cat > shieldedVm.yaml << ENDOFFILE
name: projects/$PROJECT_ID/policies/compute.requireShieldedVm
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy shieldedVm.yaml 

rm shieldedVm.yaml 

```

### 5.5. Disable VM can IP forward requirement

```
rm vmCanIpForward.yaml

cat > vmCanIpForward.yaml << ENDOFFILE
name: projects/$PROJECT_ID/policies/compute.vmCanIpForward
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy vmCanIpForward.yaml

rm vmCanIpForward.yaml

```

### 5.6. Enable VM external access 

```

rm vmExternalIpAccess.yaml

cat > vmExternalIpAccess.yaml << ENDOFFILE
name: projects/$PROJECT_ID/policies/compute.vmExternalIpAccess
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy vmExternalIpAccess.yaml

rm vmExternalIpAccess.yaml

```

### 5.7. Enable restrict VPC peering

```
rm restrictVpcPeering.yaml

cat > restrictVpcPeering.yaml << ENDOFFILE
name: projects/$PROJECT_ID/policies/compute.restrictVpcPeering
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy restrictVpcPeering.yaml

rm restrictVpcPeering.yaml

```


### 5.8. Configure ingress settings for Cloud Functions

```
rm gcf-ingress-settings.yaml

cat > gcf-ingress-settings.yaml << ENDOFFILE
name: projects/$PROJECT_NUMBER/policies/cloudfunctions.allowedIngressSettings
spec:
  etag: CO2D6o4GEKDk1wU=
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy gcf-ingress-settings.yaml

rm gcf-ingress-settings.yaml

```

### 5.9. Validation
To describe a particular constratint, run like the below describes the constraint for cloud function ingress setting for the author's project-
```
gcloud org-policies describe \
cloudfunctions.allowedIngressSettings --project=$PROJECT_ID
```

Author's output:
```
name: projects/xxxnn/policies/cloudfunctions.allowedIngressSettings
spec:
  etag: CPz46Y4GELiOlfQB
  rules:
  - values:
      allowedValues:
      - ALLOW_ALL
  updateTime: '2022-01-09T06:11:08.512051Z'
  
 ```

<hr style="border:12px solid gray"> </hr>
<br>

## 6. Create a Service Account

```
gcloud iam service-accounts create ${UMSA} \
    --description="User Managed Service Account for the Composer-2-Playground project" \
    --display-name=$UMSA 
```


<hr style="border:12px solid gray"> </hr>
<br>


## 7. Grant General IAM Permissions 

### 7.1. Permissions specific to UMSA

#### 7.1.a. Service Account User role for UMSA

```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=serviceAccount:${UMSA_FQN} \
    --role=roles/iam.serviceAccountUser   
```


#### 7.1.b. Service Account Token Creator role for UMSA

```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=serviceAccount:${UMSA_FQN} \
    --role=roles/iam.serviceAccountTokenCreator  
```

### 7.2. Permissions specific to UMSA

### 7.2.a. Permission for lab attendee to operate as the UMSA

```
gcloud iam service-accounts add-iam-policy-binding \
    ${UMSA_FQN} \
    --member="user:${ADMIN_FQ_UPN}" \
    --role="roles/iam.serviceAccountUser"
```

```
gcloud iam service-accounts add-iam-policy-binding \
    ${UMSA_FQN} \
    --member="user:${ADMIN_FQ_UPN}" \
    --role="roles/iam.serviceAccountTokenCreator"
```

<hr style="border:12px solid gray"> </hr>
<br>

## 8. Grant IAM Permissions specific to Cloud Composer

### 8.a. Cloud Composer Administrator role for UMSA

```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=serviceAccount:${UMSA_FQN} \
    --role=roles/composer.admin
```

### 8.b. Cloud Composer Worker role for UMSA

```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=serviceAccount:${UMSA_FQN} \
    --role=roles/composer.worker
```

### 8.c. Cloud Composer ServiceAgentV2Ext role for Composer Google Managed Service Agent Account (CGMSAA)

This account is visible in IAM on Cloud Console only when the "Include Google Provided Role Grants" check box is checked.
This service accounts gets auto-created in the project when the Google API for Composer is enabled.

```
CGMSAA_FQN=service-${PROJECT_NUMBER}@cloudcomposer-accounts.iam.gserviceaccount.com

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=serviceAccount:${CGMSAA_FQN} \
    --role roles/composer.ServiceAgentV2Ext
```


### 8.d. Permissions for operator to be able to change configuration of Composer 2 environment and such

```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=user:${ADMIN_FQ_UPN} \
    --role roles/composer.admin

```

### 8.e. Permissions for operator to be able to manage the Composer 2 GCS buckets and environments


```
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member=user:${ADMIN_FQ_UPN} \
    --role roles/composer.environmentAndStorageObjectViewer
```


<hr style="border:12px solid gray"> </hr>
<br>


## 9. Grant IAM Permissions specific to Cloud Functions

### 9.1. Permissions specific to UMSA

### 9.1.a. Permission for UMSA to operate as a GCF service agent

```
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member=serviceAccount:$UMSA_FQN --role=roles/cloudfunctions.serviceAgent
```

### 9.1.b. Permission for UMSA to operate as a GCF admin

```
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member=serviceAccount:$UMSA_FQN --role=roles/cloudfunctions.admin
```

<hr style="border:12px solid gray"> </hr>
<br>
<br>


## 10. Grant IAM Permissions specific to Cloud Dataflow

### 10.1. Permissions for UMSA to spawn Cloud Dataflow pipelines

a) Dataflow worker
```
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member=serviceAccount:$UMSA_FQN --role=roles/dataflow.worker
```

b) To Dataflow developer
```
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member=serviceAccount:$UMSA_FQN --role=roles/dataflow.worker
``` 

<hr style="border:12px solid gray"> </hr>
<br>

## 11. Grant IAM Permissions specific to Cloud Storage

### 11.1. Permissions for UMSA to read from GCS

a) ObjectViewer
```
gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN --role="roles/storage.objectViewer"
```

<hr style="border:12px solid gray"> </hr>
<br>


## 12. Provision Clone Composer 2, impersonating the UMSA identity

Takes abour 30 minutes..
```
gcloud composer environments create ${COMPOSER_ENV_NM} \
--location ${REGION} \
--labels env=dev,purpose=kicking-tires \
--network ${VPC_NM} \
--subnetwork ${SUBNET_NM} \
--image-version "composer-2.0.0-airflow-2.1.4" \
--service-account ${UMSA_FQN}
```

Once the environment is available (takes 30 minutes), browse through all the UIs of Cloud Composer as well as the Airflow UI.


## 13. Clone this hands-on-lab's git repo

In cloud shell, clone the repo-
```
git clone https://github.com/anagha-google/composer2-basic-orchestration.git
```

## 14. What's next?

In the next module, we will create a basic "Hello World" DAG to validate if our Composer environment is functional.

