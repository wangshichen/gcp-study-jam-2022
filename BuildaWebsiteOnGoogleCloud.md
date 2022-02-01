# Topics of the Quest
- (Video) Hosting a Static Web App on Google Cloud using Cloud Storage
- Deploy Your Website on Cloud Run
- (Video) Hosting a Web App on Google Cloud using Compute Engine
- Hosting a Web App on Google Cloud Using Compute Engine
- Deploy, Scale, and Update Your Website on Google Kubernetes Engine
- Migrating a Monolithic Website to Microservices on Google Kubernetes Engine
- (Video) Case Study: Hosting Scalable web apps on Google Cloud
- Build a Website on Google Cloud: Challenge Lab

# Challenge Lab Notes
- Create your cluster in us-central1.
- Naming is normally team-resource.
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination.
- Use the n1-standard-1 machine type unless directed otherwise.

## step reference
### pre-requisite
- activate cloud shell terminal and type `gcloud auth list` for authorization.
- `gcloud config set compute/zone us-central1-a` set default region/zone.
- `gcloud services enable cloudbuild.googleapis.com` & `gcloud services enable container.googleapis.com` to enable necessary gcp api.

### Task 1 Download the monolith code and build your container
- `cd ~`
- `git clone https://github.com/googlecodelabs/monolith-to-microservices.git`
- `cd ~/monolith-to-microservices`
- `./setup.sh`
- `cd ~/monolith-to-microservices/monolith`
- `gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/[MONOLITH_IMAGE_IDENTIFIER]:1.0.0 .`


### Task 2 Create a kubernetes cluster and deploy the application
- `gcloud container clusters create [CLUSTER_IDENTIFIER] --num-nodes 3`
- `kubectl create deployment [MONOLITH_CONTAINER_IDENTIFIER] --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/[MONOLITH_IMAGE_IDENTIFIER]:1.0.0`
- `kubectl expose deployment [MONOLITH_CONTAINER_IDENTIFIER] --type=LoadBalancer --port 80 --target-port 8080`
- `kubectl get service`


### Task 3 Create a containerized version of your Microservices
- `cd ~/monolith-to-microservices/microservices/src/orders`
- `gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/[ORDER_IMAGE_IDENTIFIER]:1.0.0 .`
- `cd ~/monolith-to-microservices/microservices/src/products`
- `gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/[PRODUCT_IMAGE_IDENTIFIER]:1.0.0 .`


### Task 4 Deploy the new microservices
- `kubectl create deployment [ORDER_CONTAINER_IDENTIFIER] --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/[ORDER_IMAGE_IDENTIFIER]:1.0.0`
- `kubectl expose deployment [ORDER_CONTAINER_IDENTIFIER] --type=LoadBalancer --port 80 --target-port 8081`
- `kubectl create deployment [PRODUCT_CONTAINER_IDENTIFIER] --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/PRODUCT_IMAGE_IDENTIFIER:1.0.0`
- `kubectl expose deployment [PRODUCT_CONTAINER_IDENTIFIER] --type=LoadBalancer --port 80 --target-port 8082`
- `kubectl get service`
- `check http://[ORDERS_EXTERNAL_IP]/api/orders & http://[PRODUCT_EXTERNAL_IP]/api/products`


### Task 5 Configure the Frontend microservice
- `cd ~/monolith-to-microservices/react-app`
- edit `.env` file to replace the following content
```
REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders
REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products
```
- `npm run build`


### Task 6 Create a containerized version of the Frontend microservice
- `cd ~/monolith-to-microservices/microservices/src/frontend`
- `gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/[FRONTEND_IMAGE_IDENTIFIER]:1.0.0 .`


### Task 7 Deploy the Frontend microservice
- `kubectl create deployment [FRONTEND_CONTAINER_IDENTIFIER] --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/[FRONTEND_IMAGE_IDENTIFIER]:1.0.0`
- `kubectl expose deployment [FRONTEND_CONTAINER_IDENTIFIER] --type=LoadBalancer --port 80 --target-port 8080`
- `kubectl get service`
