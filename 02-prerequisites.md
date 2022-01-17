# About

This module includes all prerequisites for the orchesration lab-
1. Service account creation
2. IAM permissions
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

## 3. Create a VPC, a subnet, firewall rules

Launch cloud shell, change scope to the project you created (if required), and run the commands below to create the networking entities required for the hands on lab.


#### 3.1. Create a VPC

a) Create the network
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

a) Create subnet for Composer2
```
gcloud compute networks subnets create $SUBNET_NM \
     --network=$VPC_NM \
     --range=10.0.0.0/24 \
     --region=$REGION \
     --enable-private-ip-google-access
```

#### 3.3. Create firewall rules
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

Applicable for Google Customer Engineers working in Argolis; Modify/Apply/may not apply for your environment - check with your administrator.

a) Create variables for use further in the rest of project in cloud shell<br>
Covered in section 1.0

b) Relax require OS Login
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

c) Disable Serial Port Logging

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

d) Disable Shielded VM requirement

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

e) Disable VM can IP forward requirement

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

f) Enable VM external access 

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

g) Enable restrict VPC peering

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


h) Configure ingress settings for Cloud Functions

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

i) Validation<br>
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


## 7. Grant IAM Permissions 

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


## 8. Permissions specific to Cloud Composer

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


## 9. Permissions specific to Cloud Functions

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


## 10. Permissions specific to Cloud Dataflow

### 10.1. Permissions for UMSA to spawn Cloud Dataflow pipelines

a) Dataflow worker
```
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member=serviceAccount:$UMSA_FQN --role=roles/dataflow.worker
```

b) To Dataflow developer
```
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member=serviceAccount:$UMSA_FQN --role=roles/dataflow.worker
``` 

## 11. Permissions specific to Cloud Storage

### 11.1. Permissions for UMSA to read from GCS

a) ObjectViewer
```
gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$UMSA_FQN --role="roles/storage.objectViewer"
```
