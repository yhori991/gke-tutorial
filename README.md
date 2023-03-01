# GKE Tutorial
This is a Tutorial of Google Kubernetes Engine

1. [Create GKE Cluster & other GCP resources](https://github.com/khosino/gke-tutarial#create-gke-cluster)
2. Deploy sample web application on GKE
3. 

## Create GKE Cluster and other GCP resources

[What's Kubernetes?](https://cloud.google.com/learn/what-is-kubernetes)

With the widespread adoption of containers among organizations, Kubernetes, the container-centric management software, has become the de facto standard to deploy and operate containerized applications.

[What's GKE?](https://cloud.google.com/kubernetes-engine?hl=en)

GKE (Google Kubernetes Engine)is the most scalable and fully automated Kubernetes service

### Create VPC
<img width="1238" alt="image" src="https://user-images.githubusercontent.com/111631457/220551439-99f9fb4e-0e62-4621-ae8f-b7d3182cb647.png">
<img width="846" alt="image" src="https://user-images.githubusercontent.com/111631457/220551872-3ac1620d-195a-4fcd-80e0-422e446d640d.png">
Click Create

VPC Network is created.

### Create GKE Cluster
<img width="956" alt="image" src="https://user-images.githubusercontent.com/111631457/220567036-0d471367-116a-4bdf-b8f3-9f1c9f2ac776.png">
<img width="658" alt="image" src="https://user-images.githubusercontent.com/111631457/220567363-2a8090da-4e87-4c47-88ba-14275a05ab7b.png">

<img width="895" alt="image" src="https://user-images.githubusercontent.com/111631457/220567457-6fa2a27d-fbed-47eb-86b2-377644af88af.png">
<img width="961" alt="image" src="https://user-images.githubusercontent.com/111631457/220567876-d0d71208-3e38-4093-bc8c-42bd4fd47a18.png">
Click `Create`

GKE Cluster is created.

### Create Artifact Registry (Docker Registry)

<img width="1427" alt="image" src="https://user-images.githubusercontent.com/111631457/222051142-1033f513-dc16-4a7c-be92-20f6c226970b.png">
<img width="853" alt="image" src="https://user-images.githubusercontent.com/111631457/222051237-5bb8b711-eac1-4e34-9abc-9bb9b594aaaf.png">
Click `Create`

Repogitry is created

<img width="562" alt="image" src="https://user-images.githubusercontent.com/111631457/222076746-d00ea348-2453-4dc4-8282-336bf51450fc.png">

## Deploy sample python web application on kubernetes

In Cloud Shell

Clone this github repo

```
$ git clone https://github.com/khosino/gke-tutarial.git
$ cd gke-tutorial
$ ls -l
```

### Build and push the docker image

Please check the Dockerfile and index.html! (Cloud Shell Editor is recommended)

<img width="1414" alt="image" src="https://user-images.githubusercontent.com/111631457/222078397-7e5082ca-3f83-4222-b0b6-8afb68b4e3a7.png">

#### Build the Dockerfile

--tag option means naming the image.

```
$ cd docker
$ docker build . --tag test-web-image
```

<img width="786" alt="image" src="https://user-images.githubusercontent.com/111631457/222078834-310c280d-3349-4599-a70b-2de5157fcb21.png">

Check the result image.

```
$ docker images

REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
test-web-image   latest    2298801489d6   42 seconds ago   435MB
```

#### Push to docker repogitry created in GCP

[This is the Doc how to push the image to Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling?hl=ja)

Need to set the tag like below. You can copy from Artifact Registry console in Google Cloud.

_{LOCATION}-docker.pkg.dev/{PROJECT-ID}/{REPOSITORY_NAME}/{IMAGE_NAME}:{VERSION_TAG}_


**Change the image tag to push**

```
$ docker tag test-web-image:latest asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image:latest
$ docker images

REPOSITORY                                                                           TAG       IMAGE ID       CREATED         SIZE
asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image   latest    2298801489d6   6 minutes ago   435MB
test-web-image                                                                       latest    2298801489d6   6 minutes ago   435MB
```

**Execute the push command**

```
$ docker push asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image
```

<img width="1238" alt="image" src="https://user-images.githubusercontent.com/111631457/222081450-fc9626a9-c5f0-4d0a-be4f-c7bb9e88f824.png">

**Check the result. It's pushed to Artifact Registry**

<img width="1021" alt="image" src="https://user-images.githubusercontent.com/111631457/222081723-29afdd15-1b21-4689-b0aa-885bbc92d441.png">

### Deploy the docker image on GKE

#### Connect to GKE from Cloud Shell

Select the CONNECT on the console and change the project info to yours.

<img width="757" alt="image" src="https://user-images.githubusercontent.com/111631457/222082678-af53214a-4fc4-4c16-b830-5cd7777d2361.png">

<img width="1015" alt="image" src="https://user-images.githubusercontent.com/111631457/222082781-3c8b3a66-7000-4083-a67c-b8652d5d954e.png">


```
$ gcloud container clusters get-credentials autopilot-cluster-1 --region asia-northeast1 --project gke-tutorial-hclsj
```

Check 'kubectl'

```
## Command Cheet Sheet

# Node check
$ kubectl  get  nodes


# Pod check
kubectl  get  pods
kubectl  get  pods  -n  {namespace(e.g. default, kube-system)} 
kubectl  get  pods   -o  wide

# Kubernetes の マニフェスト を適用
kubectl  apply  -f  {YAML file path}
kubectl  apply  -f  {directly path including YAML}
```

```
$ kubectl get nodes
$ kubectl get pods
$ kubectl get pods -n kube-system
```

#### Deploy Pod

Check the kubenetes manifest file in Pod directory and change the image path.

<img width="1254" alt="image" src="https://user-images.githubusercontent.com/111631457/222085805-5c500abf-6083-4a79-9871-0a7bbe97e5a6.png">

```
$ cd manifest
$ kubectl apply -f pod.yaml
```
Compute Default SAの権限を変える






