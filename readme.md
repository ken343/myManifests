Steps to setting up a simple GKE Cluster

gcloud auth list

gcloud config list project

gcloud config set compute/zone us-central1-a

gcloud container clusters create [CLUSTER-NAME]

gcloud container clusters get-credentials [CLUSTER-NAME]

kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0

kubectl expose deployment hello-server --type=LoadBalancer --port 8080

kubectl get service

*curl/wget check hello service http:http://XX.XX.XX.XX:8080/

gcloud container clusters delete [CLUSTER-NAME]
