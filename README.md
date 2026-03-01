# GCP-Workload-Identity-Federation-Lab-Service-Account-Impersonation-Cloud-Storage-Permission
GCP Workload Identity Federation Lab | Service Account Impersonation + Cloud Storage Permission

How to use Workload Identity Federation in Google Cloud

How service account impersonation works

Difference between Storage Object Admin vs Storage Viewer

Why gcloud storage ls fails but bucket command works

How to fix storage.buckets.list permission error

STEP 1 — Create Target SA (Bucket Access)
1.1 create SA

gcloud iam service-accounts create workload-target-sa \
  --display-name="Target SA for GCS"
  
  1.2 - grant SA as project level only (optional)
  
  gcloud storage buckets add-iam-policy-binding gs://demo-bucket-wif01-winppk-app01 \
  --member="serviceAccount:workload-target-sa@winppk-app01.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"
  
  1.2 - Project-level role binding
  
  gcloud projects add-iam-policy-binding winppk-app01 \
  --member="serviceAccount:workload-target-sa@winppk-app01.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"
  
  STEP 2 - Create bucket (admin once):
    2.1 - create bucket
    
  gcloud storage buckets create gs://demo-bucket-wif01-winppk-app01 \
  --location=us-central1
  
  2.2 - Create Token creator SA
  
  gcloud iam service-accounts create token-creator-sa \
  --display-name="Token Creator SA"
  
  2.3 - Allow it to impersonate workload-target-sa:(grant SA)
  
  gcloud iam service-accounts add-iam-policy-binding \
  workload-target-sa@winppk-app01.iam.gserviceaccount.com \
  --member="serviceAccount:token-creator-sa@winppk-app01.iam.gserviceaccount.com" \
  --role="roles/iam.serviceAccountTokenCreator"
  
  STEP 3 — Create VM Using token-creator-sa
  3.1 create VM attach token-creator-sa
  
  gcloud compute instances create wif-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --service-account=token-creator-sa@winppk-app01.iam.gserviceaccount.com \
  --scopes=https://www.googleapis.com/auth/cloud-platform
  
  STEP 4 — Login to VM and Enable Impersonation
  4.1 Verify current identity
  
  gcloud auth list
  
4.2 Set impersonation ---- important

  gcloud config set auth/impersonate_service_account \
  workload-target-sa@winppk-app01.iam.gserviceaccount.com
  
  STEP 5 — Test Bucket Access
  5.1 upload sample.txt file to bucket
  
  echo "Impersonation Success" > sample.txt
  gcloud storage cp sample.txt gs://demo-bucket-wif01-winppk-app01/sample.txt
  
  gcloud storage ls gs://demo-bucket-wif01-winppk-app01
