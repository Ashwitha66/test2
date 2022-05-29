# Cloud-Based-Movie-Recommender-System
A recommender system is used to recommend items to a user based on the judgement whether they would prefer the item or not. This is done by predicting the future preferences of the user on the basis of past preferences and the preferences of similar users. The goal of this project is to create such a recommender system for movies using collaborative filtering approach. For an interactive experience for online users, I  created a data application using open source frameworks - Streamlit. The application is hosted as a Docker container on GCP and deployed on Kubernetes cluster using GKE. A load balancer service (deployed as Ingress service on GKE) distributes server workloads across multiple computing resources so the app can handle multiple call requests without timing out. In addition, GKE provides a stable IP address that external users can use to access the app. When the cluster receives a request, the load balancer routes it to one of the Pods in the Service, which then returns an instance of the Streamlit app. Our primary goal is to implement the recommender system for movies. My stretch goal is to extend this system to work with other datasets such as music, ecommerce, etc. Moreover, in addition to the offline recommendation algorithm, we are working towards integrating an online recommender algorithm as well.

## Architecture
![Arch_Image](https://github.com/AmitProspeed/Cloud-Based-Movie-Recommender-System/blob/main/images/Architecture.png)

## Interaction of Different Software and Hardware Components:

The initial collaborative filtering based recommender application is deployed on an open source interactive framework called Streamlit for enhanced user interactions. The Streamlit application is then built into a docker image and hosted as a docker container on GCP. The docker container is then deployed on a Kubernetes cluster using GKE. A kubernetes load balancer service on GKE helps expose the application to external users by providing an external IP and handle multiple server requests. When the cluster receives a request, the load balancer routes it to one of the Pods in the Service, which then returns an instance of the Streamlit app which runs the collaborative filtering recommendation algorithm. The application can be scaled up by adding additional pods. We can also delete the service and container cluster to avoid incurring unwanted charges. The computed recommendations for an user are updated in Redis database running in a separate pod.


## Collaborative Filtering (Training):
Our approach is to use collaborative filtering for providing recommendations. Iam  using [MovieLens dataset](https://grouplens.org/datasets/movielens/25m/) for training our recommendation system. The dataset consists of user ratings for movies. For making a recommendation for a user (“active user”), the first step is to find similarity to other users. This can be done via techniques such as Euclidean distance, Pearson correlation or Cosine similarity. The next step is to create a weighted matrix to find the users whose ratings are more similar to the active user. The recommendation matrix is then generated by normalizing the aggregated weighted matrix.
![Collaborative_filtering_Image](https://github.com/AmitProspeed/Cloud-Based-Movie-Recommender-System/blob/main/images/Collaborative_filtering.png)


## Collaborative Filtering (Evaluation):
For evaluation, Iam using a metric called hit rate ratio(HR@n). To measure hit rate, we first generate top N recommendations for all users in the dataset. If the generated list contains something that users rated, it corresponds to one hit. Hit rate = hits / (number of users)

![evaluation_Image](https://github.com/AmitProspeed/Cloud-Based-Movie-Recommender-System/blob/main/images/evaluation.png)

### Instructions to setup (In Local):- 

-> Setup streamlit environment

1. Inside project folder, python3 -m pip install --user virtualenv

2. python3 -m pip install --user virtualenv

3. source env/bin/activate

4. pip install streamlit

5. pip install redis

6. pip install flask

7. pip install jsonpickle

    -> Setup gcloud redis/rabbitmq

    1. gcloud config set compute/zone us-central1-b
    2. gcloud config set project southern-surge-289519
    3. gcloud container clusters create --preemptible mykube
    4. ./deploy-local-dev.sh
    5. (Later delete cluster if needed) gcloud container clusters delete mykube

8. cd rest/ python3 recommendation_rest_server.py

9. streamlit run filename.py

Local URL: http://localhost:8501
Network URL: http://10.0.0.33:8501

7. Finally, deactivate if required


-> Setup kubernetes in local (if needed)

1. curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
2. chmod +x ./kubectl
3. sudo mv ./kubectl /usr/local/bin/kubectl


### Deployment on GKE (GCP)

-> Dataset - Download and unzip dataset from https://grouplens.org/datasets/movielens/25m/ in rest/dataset

-> docker build -t gcr.io/southern-surge-289519/recc-rest -f /home/amro9884/Cloud-Based-Movie-Recommender-System/rest/Dockerfile-rest .

-> docker push gcr.io/southern-surge-289519/recc-rest:latest

-> kubectl apply -f rest-deployment.yaml

-> kubectl apply -f rest-service.yaml

-> docker build -t gcr.io/southern-surge-289519/recc-app -f /home/amro9884/Cloud-Based-Movie-Recommender-System/Dockerfile-app .

-> docker push gcr.io/southern-surge-289519/recc-app:latest

-> kubectl apply -f app-deployment.yaml

-> kubectl apply -f app-ingress-backendconfig.yaml

-> (to delete backend config) - kubectl delete backendconfig app-ingress-backendconfig

-> kubectl apply -f app-service.yaml

Ingress - gcloud container clusters update mykube --update-addons=HttpLoadBalancing=ENABLED

-> kubectl apply -f app-ingress.yaml






### Screenshots

![Arch_Image](https://github.com/AmitProspeed/Cloud-Based-Movie-Recommender-System/blob/main/images/screen1.png)

![Arch_Image](https://github.com/AmitProspeed/Cloud-Based-Movie-Recommender-System/blob/main/images/screen2.png)

![Arch_Image](https://github.com/AmitProspeed/Cloud-Based-Movie-Recommender-System/blob/main/images/screen3.png)

![Arch_Image](https://github.com/AmitProspeed/Cloud-Based-Movie-Recommender-System/blob/main/images/screen4.png)

![Arch_Image](https://github.com/AmitProspeed/Cloud-Based-Movie-Recommender-System/blob/main/images/screen5.png)