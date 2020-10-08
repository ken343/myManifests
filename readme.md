gcloud auth list

gcloud config list project

gcloud config set compute/zone us-central1-a

[Preparation]
gcloud compute networks create lb-network --subnet-mode=custom

gcloud compute networks subnets create backend-subnet \
    --network=lb-network \
    --range=10.1.2.0/24 \
    --region=us-central1

gcloud compute networks subnets create proxy-only-subnet \
  --purpose=INTERNAL_HTTPS_LOAD_BALANCER \
  --role=ACTIVE \
  --region=us-central1 \
  --network=lb-network \
  --range=10.129.0.0/23

[Firewall]
gcloud compute firewall-rules create fw-allow-ssh \
    --network=lb-network \
    --action=allow \
    --direction=ingress \
    --target-tags=allow-ssh \
    --rules=tcp:22

gcloud compute firewall-rules create fw-allow-health-check \
    --network=lb-network \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=load-balanced-backend \
    --rules=tcp

gcloud compute firewall-rules create fw-allow-proxies \
  --network=lb-network \
  --action=allow \
  --direction=ingress \
  --source-ranges=10.129.0.0/23 \
  --target-tags=load-balanced-backend \
  --rules=tcp:80,tcp:443,tcp:8080

[Create Cluster]
	
gcloud container clusters create my-cluster \
--release-channel=rapid \
--enable-ip-alias \
--zone=us-central1-a \
--network=lb-network
--subnetwork=backend-subnet

[Get Credentials for kubectl]
gcloud container clusters get-credentials [CLUSTER-NAME]

[Deployments-Services-Ingresses]
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0

kubectl expose deployment hello-server --type=LoadBalancer --port 8080

[Test]
kubectl get pods
kubectl get service
kubectl get ingress

gcloud compute instances create l7-ilb-client-us-central1-a \
--image-family=debian-9 \
--image-project=debian-cloud \
--network=lb-network \
--subnet=backend-subnet \
--zone=us-central1-a \
--tags=allow-ssh

  # SSH in to the VM
  gcloud compute ssh l7-ilb-client-us-central1-a \
      --zone=us-central1-a

  # Use curl to access the internal application VIP
  curl 10.128.0.58 #Whatever the ingress ip is.
  hostname-server-6696cf5fc8-z4788

*curl/wget check hello service http:http://34.72.190.154:8080/ #For external lb test

[Delete Cluster]
gcloud container clusters delete [CLUSTER-NAME]
