# GKE Autopilot Tutorial
This is a Tutorial of Google Kubernetes Engine

<details>
<summary>Table of Contents</summary>
  
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Create GKE Cluster and other GCP resources](#create-gke-cluster-and-other-gcp-resources)
  - [Create VPC](#create-vpc)
  - [Create GKE Cluster](#create-gke-cluster)
  - [Create Artifact Registry (Docker Registry)](#create-artifact-registry-docker-registry)
- [Deploy the sample web application on kubernetes](#deploy-the-sample-web-application-on-kubernetes)
  - [Build and push the docker image](#build-and-push-the-docker-image)
    - [Build the Dockerfile](#build-the-dockerfile)
    - [Push to docker repogitry created in GCP](#push-to-docker-repogitry-created-in-gcp)
  - [Deploy the docker image on GKE](#deploy-the-docker-image-on-gke)
    - [Connect to GKE from Cloud Shell](#connect-to-gke-from-cloud-shell)
    - [Deploy the "Pod"](#deploy-the-pod)
    - [Deploy the "Deployment" and "Service"](#deploy-the-deployment-and-service)
    - [Reaource request](#reaource-request)
      - [CPU and Memory request](#cpu-and-memory-request)
      - [GPU reauest](#gpu-reauest)
      - [Spot Pods](#spot-pods)
    - [Horizontal Pod Autoscaling (HPA)](#horizontal-pod-autoscaling-hpa)
    - [CI/CD](#cicd)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
  
</details>

## 1. Create GKE Cluster and other GCP resources

[What's Kubernetes?](https://cloud.google.com/learn/what-is-kubernetes)

With the widespread adoption of containers among organizations, Kubernetes, the container-centric management software, has become the de facto standard to deploy and operate containerized applications.

[What's GKE?](https://cloud.google.com/kubernetes-engine?hl=en)

GKE (Google Kubernetes Engine)is the most scalable and fully automated Kubernetes service

---

### 1.1 Create VPC

<details>
<summary>VPC console Screen shot</summary>
<img width="1238" alt="image" src="https://user-images.githubusercontent.com/111631457/220551439-99f9fb4e-0e62-4621-ae8f-b7d3182cb647.png">
      
<img width="846" alt="image" src="https://user-images.githubusercontent.com/111631457/220551872-3ac1620d-195a-4fcd-80e0-422e446d640d.png">
</details>

---

### 1.2 Create GKE Cluster

<details>
<summary>GKE console Screen shot</summary>
<img width="956" alt="image" src="https://user-images.githubusercontent.com/111631457/220567036-0d471367-116a-4bdf-b8f3-9f1c9f2ac776.png">
      
<img width="658" alt="image" src="https://user-images.githubusercontent.com/111631457/220567363-2a8090da-4e87-4c47-88ba-14275a05ab7b.png">

<img width="895" alt="image" src="https://user-images.githubusercontent.com/111631457/220567457-6fa2a27d-fbed-47eb-86b2-377644af88af.png">
      
<img width="961" alt="image" src="https://user-images.githubusercontent.com/111631457/220567876-d0d71208-3e38-4093-bc8c-42bd4fd47a18.png">
</details>

---

### 1.3 Create Artifact Registry (Docker Registry)


<details>
<summary>Artifact Registry console Screen shot</summary>
<img width="1427" alt="image" src="https://user-images.githubusercontent.com/111631457/222051142-1033f513-dc16-4a7c-be92-20f6c226970b.png">
      
<img width="853" alt="image" src="https://user-images.githubusercontent.com/111631457/222051237-5bb8b711-eac1-4e34-9abc-9bb9b594aaaf.png">

<img width="562" alt="image" src="https://user-images.githubusercontent.com/111631457/222076746-d00ea348-2453-4dc4-8282-336bf51450fc.png">
</details>

## 2. Deploy the sample web application on kubernetes

In Cloud Shell

Clone this github repo

```
$ git clone https://github.com/khosino/gke-tutarial.git
$ cd gke-tutorial
$ ls -l
```

---

### 2.1 Build and push the docker image

Please check the Dockerfile and index.html! (Cloud Shell Editor is recommended)

<img width="1414" alt="image" src="https://user-images.githubusercontent.com/111631457/222078397-7e5082ca-3f83-4222-b0b6-8afb68b4e3a7.png">

---

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

---

#### Push to docker repogitry created in GCP

[This is the Doc how to push the image to Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling?hl=ja)

Need to set the tag like below. You can copy from Artifact Registry console in Google Cloud.

_{LOCATION}-docker.pkg.dev/{PROJECT-ID}/{REPOSITORY_NAME}/{IMAGE_NAME}:{VERSION_TAG}_


Change the image tag to push

```
$ docker tag test-web-image:latest asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image:latest
$ docker images

REPOSITORY                                                                           TAG       IMAGE ID       CREATED         SIZE
asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image   latest    2298801489d6   6 minutes ago   435MB
test-web-image                                                                       latest    2298801489d6   6 minutes ago   435MB
```

Execute the push command

```
$ docker push asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image
```

<img width="1238" alt="image" src="https://user-images.githubusercontent.com/111631457/222081450-fc9626a9-c5f0-4d0a-be4f-c7bb9e88f824.png">

**Check the result. It's pushed to Artifact Registry**

<img width="512" alt="image" src="https://user-images.githubusercontent.com/111631457/222081723-29afdd15-1b21-4689-b0aa-885bbc92d441.png">

---

### 2.2 Deploy the docker image on GKE

#### Connect to GKE from Cloud Shell

Select the CONNECT on the console and change the project info to yours.

<img width="512" alt="image" src="https://user-images.githubusercontent.com/111631457/222082678-af53214a-4fc4-4c16-b830-5cd7777d2361.png">

<img width="512" alt="image" src="https://user-images.githubusercontent.com/111631457/222082781-3c8b3a66-7000-4083-a67c-b8652d5d954e.png">


```
$ gcloud container clusters get-credentials autopilot-cluster-1 --region asia-northeast1 --project gke-tutorial-hclsj
```

Check `kubectl` command

```
## Command Cheet Sheet

# Node check
$ kubectl  get  nodes


# Pod check
kubectl  get  pods
kubectl  get  pods  -n  {namespace(e.g. default, kube-system)} 
kubectl  get  pods   -o  wide

# Apply the manifest of kubernetes 
kubectl  apply  -f  {YAML file path}
kubectl  apply  -f  {directly path including YAML}
```

```
$ kubectl get nodes
$ kubectl get pods
$ kubectl get pods -n kube-system
```

---

#### Deploy the "Pod"

Check the kubenetes manifest file in Pod directory and change the image path.

<img width="512" alt="image" src="https://user-images.githubusercontent.com/111631457/222085805-5c500abf-6083-4a79-9871-0a7bbe97e5a6.png">

```
$ cd manifest
$ kubectl apply -f pod.yaml
$ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-web   1/1     Running   0          2m52s
```

The pod is created. To check the detail log, use describe command.
However we cannot access to this pod because there is no service or ingress.

```
kubectl describe pods test-web
```

---

#### Deploy the "Deployment" and "Service"

Deploy the Deployment with 3 replicas.

```
$ kubectl apply -f deployment.yaml
$ kubectl get deployment
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
test-web-deployment   3/3     3            3           2m47s

$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
test-web-deployment-d7779f6f4-crvvh   1/1     Running   0          3m19s
test-web-deployment-d7779f6f4-mwnxs   1/1     Running   0          2m18s
test-web-deployment-d7779f6f4-vwwsd   1/1     Running   0          3m19s
```

Check the behavior when you try to delete a pod?

```
kubectl delete pods test-web-deployment-d7779f6f4-crvvh
```

Deploy the Service

```
$ kubectl apply -f service.yaml
$ kubectl get service
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE
kubernetes   ClusterIP      10.118.128.1     <none>         443/TCP        7d8h
test-web     LoadBalancer   10.118.130.151   34.85.xxx.xxx   80:30738/TCP   29m
```

Access to LoadBalancer EXTERNAL-IP (34.85.xxx.xxx)

<img width="323" alt="image" src="https://user-images.githubusercontent.com/111631457/222209955-e4dd8444-233a-402c-ba7b-be220de5d667.png">

The LoadBalancer is created automatically in Google Cloud.

<img width="512" alt="image" src="https://user-images.githubusercontent.com/111631457/222210376-d0fb33e9-7031-434a-b2e8-0693dd64b425.png">

---

#### Reaource request

In Autopilot, you request resources in your Pod specification. 
If you do not specify resource requests for some containers in a Pod, Autopilot applies default values. 

---

##### CPU and Memory request

check the current resources

[Resource requests in Autopilot](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-resource-requests)

```
$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
test-web-deployment-d7779f6f4-crvvh   1/1     Running   0          11h
test-web-deployment-d7779f6f4-mwnxs   1/1     Running   0          11h
test-web-deployment-d7779f6f4-vwwsd   1/1     Running   0          11h

$ kubectl describe pods test-web-deployment-d7779f6f4-crvvh
~~~
Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
~~~
# This is the default value in case you don't specify.
```

Change the manifest file
Open the comment "Resrouce Request CPU" in deployment.yaml

<details>
<summary>Manifest "development.yaml"</summary>
  
```  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-web-deployment
  labels:
    app: test-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-web
  template:
    metadata:
      labels:
        app: test-web
    spec:

      # ### Resource Request GPU
      # nodeSelector:
      #   cloud.google.com/gke-accelerator: "nvidia-tesla-t4"
      # ###

      # ### Best-effort Spot Pod
      # terminationGracePeriodSeconds: 25
      # affinity:
      #   nodeAffinity:
      #     requiredDuringSchedulingIgnoredDuringExecution:
      #       nodeSelectorTerms:
      #       - matchExpressions:
      #         - key: cloud.google.com/gke-spot
      #           operator: In
      #           values:
      #           - "true"
      # ###
      
      containers:
      - name: test-web
        image: asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image:latest

        ### Resource Request CPU
        resources:
          requests:
            cpu: 250m
            memory: 1Gi
        ###

        # ### Resource Request GPU
        # resources:
        #   limits:
        #     nvidia.com/gpu: 2
        #   requests:
        #     cpu: "36"
        #     memory: "36Gi"
        # ### 

        ports:
        - containerPort: 80
```

</details>

![image](https://user-images.githubusercontent.com/111631457/222332947-b79562ab-fb52-41b2-a16e-586f679dc3fb.png)

```
$ kubectl apply -f deployment.yaml
deployment.apps/test-web-deployment configured

$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
test-web-deployment-6587db96c8-5fsgb   1/1     Running       0          7m8s
test-web-deployment-6587db96c8-5l9wr   1/1     Terminating   0          5m31s
test-web-deployment-d7779f6f4-4cqqm    0/1     Pending       0          1s
test-web-deployment-d7779f6f4-kdj4h    1/1     Running       0          6s
test-web-deployment-d7779f6f4-nssz8    1/1     Running       0          2m29s
# Rolling update
# Always keep 3 pods available

$ kubectl describe pods test-web-deployment-d7779f6f4-kdj4h
~~~
Requests:
  cpu:                250m
  ephemeral-storage:  1Gi
  memory:             1Gi
~~~
```

---

##### GPU reauest

Check the quotas in the project

<img width="1209" alt="image" src="https://user-images.githubusercontent.com/111631457/222381756-b0fcf633-8e7c-4eac-8d24-81c9cccc9986.png">

<img width="1168" alt="image" src="https://user-images.githubusercontent.com/111631457/222381885-6c69a4d2-a80e-4658-8ecd-72d2dfca0fa0.png">

Open the comment "Resrouce Request GPU" and close "Resrouce Request CPU" in deployment.yaml

<details>
<summary>Manifest "development.yaml"</summary>
  
```  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-web-deployment
  labels:
    app: test-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-web
  template:
    metadata:
      labels:
        app: test-web
    spec:

      ### Resource Request GPU
      nodeSelector:
        cloud.google.com/gke-accelerator: "nvidia-tesla-t4"
      ###

      # ### Best-effort Spot Pod
      # terminationGracePeriodSeconds: 25
      # affinity:
      #   nodeAffinity:
      #     requiredDuringSchedulingIgnoredDuringExecution:
      #       nodeSelectorTerms:
      #       - matchExpressions:
      #         - key: cloud.google.com/gke-spot
      #           operator: In
      #           values:
      #           - "true"
      # ###
      
      containers:
      - name: test-web
        image: asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image:latest

        # ### Resource Request CPU
        # resources:
        #   requests:
        #     cpu: 250m
        #     memory: 1Gi
        # ###

        ### Resource Request GPU
        resources:
          limits:
            nvidia.com/gpu: 2
          requests:
            cpu: "36"
            memory: "36Gi"
        ### 

        ports:
        - containerPort: 80
```

</details>

![image](https://user-images.githubusercontent.com/111631457/222333215-27157ab9-fc7f-4ac3-8f29-21de8201379a.png)

```
$ kubectl apply -f deployment.yaml
$ kubectl get pods
$ kubectl describe pods test-web-deployment-6b67f6c455-2jlgr
~~~
Requests:
  cpu:                18
  ephemeral-storage:  1Gi
  memory:             18Gi
  nvidia.com/gpu:     1
~~~
Node-Selectors:  cloud.google.com/gke-accelerator=nvidia-tesla-t4
                 cloud.google.com/gke-accelerator-count=1
~~~
```

---

##### Spot Pods

[Spot Pods](https://cloud.google.com/kubernetes-engine/docs/how-to/autopilot-spot-pods#prefer-spot-pods) are Pods that run on nodes backed by Compute Engine Spot VMs. Spot Pods are priced lower than standard Autopilot Pods, but can be evicted by GKE whenever compute resources are required to run standard Pods.

[Requesting Spot Pods on a best-effort basis](https://cloud.google.com/kubernetes-engine/docs/how-to/autopilot-spot-pods#prefer-spot-pods)
When you request Spot Pods on a preferred basis, GKE schedules your Pods based on the following order:

1. Existing nodes that can run Spot Pods that have available allocatable capacity.
2. Existing standard nodes that have available allocatable capacity.
3. New nodes that can run Spot Pods, if the compute resources are available.
4. New standard nodes.

Deploy the best-effort basis Spot Pods with GPU

Open the comment "Best-effort Spot Pods" in development.yaml

<details>
<summary>Manifest "development.yaml"</summary>
  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-web-deployment
  labels:
    app: test-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-web
  template:
    metadata:
      labels:
        app: test-web
    spec:
    
      ### Resource Request GPU
      nodeSelector:
        cloud.google.com/gke-accelerator: "nvidia-tesla-t4"
      ###

      ### Best-effort Spot Pod
      terminationGracePeriodSeconds: 25
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: cloud.google.com/gke-spot
                operator: In
                values:
                - "true"
      ###
      
      containers:
      - name: test-web
        image: asia-northeast1-docker.pkg.dev/gke-tutorial-hclsj/gke-tutorial-repo/test-web-image:latest

        # ### Resource Request CPU
        # resources:
        #   requests:
        #     cpu: 250m
        #     memory: 1Gi
        # ###

        ### Resource Request GPU
        resources:
          limits:
            nvidia.com/gpu: 2
          requests:
            cpu: "36"
            memory: "36Gi"
        ### 

        ports:
        - containerPort: 80
```
  
</details>

```
$ kubectl apply -f deployment.yaml
$ kubectl get pods
$ kubectl describe pods 
~~~
Tolerations: cloud.google.com/gke-accelerator=nvidia-tesla-t4:NoSchedule
             cloud.google.com/gke-spot=true:NoSchedule
~~~
```

---

#### Horizontal Pod Autoscaling (HPA)

The Horizontal Pod Autoscaler changes the shape of your Kubernetes workload by automatically increasing or decreasing the number of Pods in response to the workload's CPU or memory consumption, or in response to custom metrics reported from within Kubernetes or external metrics from sources outside of your cluster.

This example creates HorizontalPodAutoscaler object to autoscale the Deployment when CPU utilization surpasses 50%, and ensures that there is always a minimum of 3 replica and a maximum of 10 replicas.

<img width="314" alt="image" src="https://user-images.githubusercontent.com/111631457/222517532-1f45fda8-55d4-4740-98e8-2eab009f6851.png">

```
$ kubectl apply -f hpa.yaml
$ kubectl get hpa
NAME           REFERENCE                        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
test-web-hpa   Deployment/test-web-deployment   19%/50%   3         10        3          41m
```

If the load exceeds 50%, Replica Pods are created as follows

<img width="670" alt="image" src="https://user-images.githubusercontent.com/111631457/222523278-0e1239b8-ecd9-430c-897a-e4437543c930.png">

[Ref.]
For a simple load, log in to the container and execute a lot of "yes" commands.

```
$ kubectl exec -it test-web-deployment-5bd8b78bfc-675bj /bin/bash

$ yes > dev/null &
(repeat it)
```


---

#### CI/CD

Continuous integration and continuous delivery (CI/CD) are essential processes to deliver software quickly and reliably. CI/CD helps to automate the build, test, and deployment process, which can save time and reduce errors.

In your case, for example, you can simply write the source code for the chi chi learning to be used in Ray Cluster and upload it to git, which will automatically "build the container", "push to Docker Registory", "deploy to GKE", etc.
In other words, you can do everything in this tutorial automatically.



