[**Kubernetes Dashboard**]

Repository to store files and configuration needed to deploy the Kubernetes Dashboard with public access and custom Certificates.

## Introduction

Official installation notes can be found here: https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui

## Notes

- By default, k8s dashboard installation can be accesses only from `localhost` and using `kubectl proxy`.
  - To allow dashboard access from outside the cluster, the attribute `type: NodePort` is added to `kubernetes-dashboard` Service. Also, a fixed port is configured with `nodePort: 30001` attribute.
- During the dashboard deployment, a self-signed certificate is automatically generated and web browsers like Chrome are blocking the connection.
  - To avoid that, a self-signed certificate is manually generated and it will be added to client keychain or certificate manager.
- At last, following the official installation guide, a dedicated admin user must be created
  - The file `AdminUser.yml` is added in order to generate new user and permissions.

## Installation

To install and deploy the kubernetes dashboard, the following steps must be executed:
1. Download the last deployment file

Download the official deployment file by executing:
`wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml`

2. Edit the downloaded file

Once the deployment file is available, we need to comment: The Namespace creation and the kubernetes-dashboard secret; and also adapt the kubernetes-dashboard Service to include the type: NodePort and nodePort: 30001 attributes.

The file included in this git repository already has these modifications.

3. Creation of Namespace

Create manually the namespace by executing `kubectl create namespace kubernetes-dashboard`

4. Generate the certificates

In order to create the self-signed certificates, we must execute...
```
mkdir certs
cd certs
openssl genrsa -out dashboard.key 2048
openssl rsa -in dashboard.key -out dashboard.key
openssl req -sha256 -new -key dashboard.key -out dashboard.csr -subj '/CN=localhost'
openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard.key --from-file=dashboard.crt -n kubernetes-dashboard
```

5. Deploy the Kubernetes Dashboard

Once the previous configuration is done, we can deploy the rest of the components by executing `kubectl apply -f recommended.yml`

6. Add Admin User

To create a dedicated admin user execute `kubectl apply -f AdminUser.yml`

7. Accessing the Kubernetes Dashboard

Once, the previos steps has been executed correctly, the UI will be available at https://<IP>:30001

8. Configuring the self-signed certificates

In order to avoid our browser to block the connection, the generated certificate must be configured. To do that, you must copy the dashboard.crt file locally and add it to your Keychain (Mac) or any other certificate manager. Once is has been added, you must trust any use of it.

9. Get Token

Once we have been able to access the Kubernetes Dashboard UI, it will ask for a token. To get the generated token for our admin user you must execute
```
kubectl get secrets -n kubernetes-dashboard
kubectl describe secret <SECRET FROM YOUR ADMIN USER> -n kubernetes-dashboard 
```

## References

This deployment guide has been based on https://medium.com/@sondnpt00343/deploying-a-publicly-accessible-kubernetes-dashboard-v2-0-0-betax-8e39680d4067

