
# Least privileged Cluster Roles
The following sections provide cluster role definitions for a few example use cases and expected results.

## Use Cases:

In this repo, we illustrate how Kubernetes RBAC can be used to control access to the Kubernetes API server using least privileged cluster roles.

Let us say that your SRE team has four groups of users/service accounts who need cluster level access with the following permissions.


1. cluster-admin-group: This group needs all access to the cluster.
2. cicd-group:  Members of this group can do everything in the cluster, except reading or writing secrets. (So e.g. user in group can change role bindings for all namespaces within the cluster)
3. ops-group: Members of this group can do everything in the cluster, except changing role bindings and reading/writing secrets.
4. qa-group: This admin has read-only access to everything in the cluster except the permissions to read the secrets.

### get-creds-role
1. Create IAM role to get cluster credentials:
Note: This IAM role is required at the minimum for every IAM user / Service Account that needs to connect to the GKE cluster. This role has only container.clusters.get permission which allows the user/service account to invoke “gcloud container clusters get-credentials” command which populates kubeconfig file.
```
gcloud iam roles create get-creds-role --project <project-name> --file get-cred-role.yaml
```

### cluster-admin Role

Use the predefined cluster-admin role.

#### Tests

1. Create a gcloud service account for testing purposes:
```
export PROJECT_ID=my-project-1
export SA_NAME=sa-admin-user
export SERVICE_ACCOUNT_EMAIL=$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com
gcloud iam service-accounts create $SA_NAME --display-name " Admin User Account"
```

2. Assign it IAM get credentials role and api_user role.
```
gcloud projects add-iam-policy-binding my-project-1 --member='serviceAccount:sa-admin-user@my-project-1.iam.gserviceaccount.com' --role='projects/my-project-1/roles/get-creds-role'
kubectl apply -f cluster-admin-role-binding.yaml
```

3. Populate kubeconfig with user token
```
gcloud container clusters get-credentials sample-cluster --zone us-central1-a
```

4. Test the permissions
```
kubectl auth can-i create pods --all-namespaces
yes
kubectl auth can-i create clusterrolebindings
yes
kubectl auth can-i create secrets
yes
kubectl auth can-i update secrets
yes
kubectl auth can-i create networkpolicies --all-namespaces
yes
```

### cicd  Role

Create CICI User Role.
Note - Resoources and API groups in this role include all resources and API groups as in output of (kubectl api-resources -o wide) except Secrets.
```
kubectl apply -f cicd-user-cluster-role.yaml
```
#### Tests

1. Create a gcloud service account for testing purposes:
```
export PROJECT_ID=my-project-1
export SA_NAME=sa-cicd-user
export SERVICE_ACCOUNT_EMAIL=$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com
gcloud iam service-accounts create $SA_NAME --display-name " CICD User Account"
```

2. Assign it IAM get credentials role and api_user role.
```
gcloud projects add-iam-policy-binding my-project-1 --member='serviceAccount:sa-cicd-user@my-project-1.iam.gserviceaccount.com' --role='projects/my-project-1/roles/get-creds-role'
kubectl apply -f cicd-user-cluster-role-binding.yaml
```

3. Populate kubeconfig with user token
```
gcloud container clusters get-credentials sample-cluster --zone us-central1-a
```

4. Test the permissions
```
kubectl auth can-i create pods --all-namespaces
yes
kubectl auth can-i create clusterrolebindings
yes
kubectl auth can-i create secrets
no
kubectl auth can-i update secrets
no
kubectl auth can-i create networkpolicies --all-namespaces
yes
kubectl auth can-i get secrets
no
```

### Ops  Role

Create Ops User Role.
Note - Resoources and API groups in this role include all resources , verbs and API groups as in output of (kubectl api-resources -o wide) except the following:
1. No action on Secrets.
2. Read access on clusterrolebindings and rolebindings.
```
kubectl apply -f ops_user_role.yaml
```
#### Tests

1. Create a gcloud service account for testing purposes:
export PROJECT_ID=my-project-1
export SA_NAME=sa-ops-user
export SERVICE_ACCOUNT_EMAIL=$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com
gcloud iam service-accounts create $SA_NAME --display-name "Cluster Ops User Account"


2. Assign it IAM get credentials role and ops_user role.
gcloud projects add-iam-policy-binding my-project-1 --member='serviceAccount:sa-ops-user@my-project-1.iam.gserviceaccount.com' --role='projects/my-project-1/roles/get-creds-role'
kubectl apply -f ops-user-cluster-role-binding.yaml


3. Populate kubeconfig with user token
```
gcloud container clusters get-credentials sample-cluster --zone us-central1-a
```

4. Test the permissions
```
kubectl auth can-i create pods --all-namespaces
yes
kubectl auth can-i create clusterrolebindings
no
kubectl auth can-i create secrets
no
kubectl auth can-i update secrets
no
kubectl auth can-i create networkpolicies --all-namespaces
yes
kubectl auth can-i get secrets
no
kubectl auth can-i create rolebindings
no
kubectl auth can-i get rolebindings
yes
kubectl auth can-i get clusterrolebindings
yes
```

### QA User Role

Create QA User Role.
Note - Resoources and API groups in this role include read access on all resources , verbs and API groups as in output of (kubectl api-resources -o wide) except Secrets.
```
kubectl apply -f qa-user-cluster-role.yaml
```
#### Tests

1. Create a gcloud service account for testing purposes:
export PROJECT_ID=my-project-1
export SA_NAME=sa-qa-user
export SERVICE_ACCOUNT_EMAIL=$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com
gcloud iam service-accounts create $SA_NAME --display-name "Cluster QA User Account"
g


2. Assign it IAM get credentials role and qa-user-cluster-role role.
```
gcloud projects add-iam-policy-binding my-project-1 --member='serviceAccount:sa-qa-user@my-project-1.iam.gserviceaccount.com' --role='projects/my-project-1/roles/aget-creds-role'
kubectl apply -f qa-user-cluster-role-binding.yaml
```

3. Populate kubeconfig with user token
```
gcloud container clusters get-credentials sample-cluster --zone us-central1-a
```

4. Test the permissions
```
kubectl auth can-i create pods --all-namespaces
no
kubectl auth can-i create clusterrolebindings
no
kubectl auth can-i create secrets
no
kubectl auth can-i create networkpolicies --all-namespaces
no
kubectl auth can-i create rolebindings
no
kubectl auth can-i get secrets
no
kubectl auth can-i get pods --all-namespaces
yes
kubectl auth can-i get rolebindings
yes
kubectl auth can-i get clusterrolebindings
yes
```
