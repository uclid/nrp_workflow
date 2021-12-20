# PRP Kubernetes cluster usage workflow

## Background
The Cloud Computing Lab at the Department of Computer and Information Sciences, University of Delaware will be using computing resources provided by the National Research Platform (NRP) for conducting custom large-scale experiments on various research problems. As such, this document provides an overview of the technologies we need to know and learn in order to use those resources. I have created this document in a tutorial style so that all the necessary steps to successfully deploy and run your custom code in the NRP clusters are clearly established.

This document will also serve as our collective guide to deploying diverse applications, services, batch jobs, etc. to the NRP clusters. As we continue using these resources, all the contributors are expected to update the document with relevant information so that future members of the lab or other readers have access to precise instructions.

## People
* Namespace administrator (maintainer): Dixit Bhatta
* Namespace users (current): Ahmad Almansoor, Erfan Farhangi Maleki, Weibin Ma
* Namespace users (former): Sahar Nilipour Tabatabaee, Selim Emre Altıngöz
* Advisor (PI): Dr. Lena Mashayekhy

## Getting Started
We will get started with the initial information on the Nautilus cluster deployed under Pacific Research Platform (PRP). Please visit this link to read more about [PRP]( http://prp.ucsd.edu:8080/prp)

A namespace for our lab must have been already created in the PRP Kubernetes cluster. Check with the namespace administrator to get the namespace.

Please check your access to the PRP platform (subset of NRP) by logging** in at [Nautilus](https://nautilus.optiputer.net/).\
Note**: *choose University of Delaware as account type when logging in, be careful not to choose Google.*

After logging in, make sure you can see the namespace provided by your namespace admin under the "Namespaces overview" tab. 

The namespace admin can add you as a user of the namespace after you have logged in successfully in that system. Let the admin know as soon as you have successfully logged in.

## Technical Requirements
There are a couple of technologies you “must” be familiar with before running your custom code or application in the Nautilus cluster.
  
### Docker

Docker is a set of platform as a service products that uses OS-level virtualization to deliver software in packages called containers. A free video tutorial series on Docker can be found at this [link](https://www.youtube.com/playlist?list=PLhW3qG5bs-L99pQsZ74f-LC-tOEsBp2rK).

Here are the main ideas about Docker that you must know if you want to simply read through the topics over the internet:
* What is Docker?
* Docker Images
* Dockerfile
* Docker Containers
* Docker Hub

Also, here is the complete Docker [cheatsheet](https://github.com/wsargent/docker-cheat-sheet).
  
### Kubernetes

Kubernetes is an open-source container-orchestration system for automating application deployment, scaling, and management. You can follow this [tutorial](https://www.tutorialspoint.com/kubernetes/index.htm) upto “Deployments” to have sufficient knowledge to use the Nautilus cluster. 

Here are the main ideas about Kubernetes that you must know if you want to simply read through the topics over the internet:
* What is Kubernetes?
* Kubernetes Architecture
* Kubernetes Pods
* Kubernetes Deployments
* Kubernetes Services
* Kubernetes Jobs

Also, you can find the complete Kubernetes (kubectl) cheatsheet [here](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).
  
## The Workflow

### A basic walkthrough

Now that you have a basic understanding of all of the technologies described above. Please follow this tutorial [K8s intro — prpworkshop](http://prp.ucsd.edu:8080/prpworkshop/materials/k8s-1/k8s) in your local system. Please note that we are not doing anything in the Nautilus cluster yet.

The tutorial above assumes you are working in a linux-based system and trying to deploy a “service” to your local cluster. It is a simple “Hello World!” web service. However, it is important to understand that these steps can be performed in any machine, OS or environment. You just have to learn and perform equivalent steps for your machine, OS or environment.

Please pay attention to the workflow of this tutorial:

```
Step 0: Install Docker, minikube and kubectl
Step 1: Write your program and compile the code if necessary
Step 2: Build a docker image of the (compiled) code
Step 3: Run the image using kubectl (it will create both a pod and deployment**)
Note**: kubectl run is now deprecated. See full example below for a proper process
Step 4: Expose the deployment and start the service
Step 5: Cleanup
```

We build upon this workflow and use the example below to find a more detailed and precise workflow for our needs. Since almost all of us in our lab use CPLEX, I have used an example where a CPLEX model written in Java is run in the Nautilus cluster.

## Example: Running a CPLEX model (Java) in the Nautilus cluster

First, we have to ensure Docker and kubectl commands are installed and working properly in our system. Run some basic commands to double check if the installations have any issues.

After that, you need to have Linux (64-bit)** version “bin” of CPLEX installation file. You can download this by logging in or creating an account at [IBM Academic Initiative](https://content-eu-7.content-cms.com/b73a5759-c6a6-4033-ab6b-d9d4f9a6d65b/dxsites/151914d1-03d2-48fe-97d9-d21166848e65/academic/topic/data-science) website.

*Please note the Linux version is needed since we will be deploying the containers to run in a Linux environment. If any warnings are given about creating a Linux container later in this example, just ignore them and proceed.*

Now it’s time to structure your project hierarchy. Create a main project directory, we will name it “docker” for this example. Create two directories inside it, one for the CPLEX installation and another for your source code. Conventionally, Java source directories are called “src”. So, we will have the following directory structure and files.
  
```  
docker
--cplex
    cplex_studio1210.linux-x86-64.bin
    Dockerfile
    response.properties
--src
    HelloCplex.java
    cplex.jar    ---->this can be found inside your local cplex installation
Dockerfile
pod.yml
```

We have Dockerfile and response.properties inside “cplex” which will help us in running commands to create a cplex image. They will help bundle cplex libraries with our source code. The contents of the files are below. Please check your cplex file version to make changes as necessary.
  
**Dockerfile** (used to install cplex within the container)

```
FROM openjdk

COPY cplex_studio1210.linux-x86-64.bin /cplex/cplex_studio1210.linux-x86-64.bin
COPY response.properties /cplex/response.properties

RUN chmod u+x /cplex/cplex_studio1210.linux-x86-64.bin
RUN /cplex/cplex_studio1210.linux-x86-64.bin -f /cplex/response.properties
RUN rm -rf /cplex
```

**response.properties** (used to provide options during CPLEX installation)

```
#https://www.ibm.com/support/knowledgecenter/SSSA5P_12.8.0/ilog.odms.studio.help/Optimization_Studio/topics/td_silent_install.html
#https://www.ibm.com/support/knowledgecenter/SSSA5P_12.8.0/ilog.odms.studio.help/Optimization_Studio/topics/COS_installing.html
INSTALLER_UI=silent
LICENSE_ACCEPTED=TRUE
```
Now, we create a Docker image of cplex using the command inside the “cplex” directory.
`docker image build -t cplex:12.10 .`


Verify that the image is correctly created using,
`docker images`

Now, change the directory to “docker”. The Dockerfile we have there has the following content. HelloCplex.java can be found in the repository.

**Dockerfile**

```
FROM cplex:12.10

COPY src/ /app/

WORKDIR /app

RUN javac -cp cplex.jar HelloCplex.java

ENTRYPOINT ["java","-Djava.library.path=/opt/ibm/ILOG/CPLEX_Studio1210/cplex/bin/x86-64_linux","-cp",".:/opt/ibm/ILOG/CPLEX_Studio1210/cplex/lib/cplex.jar","HelloCplex"]
```

As you can observe, it will compile and run the java file. So, run the command to create the docker image.\
`docker build -t cplex_test .`

*Please note that your code must not have any errors and you must have checked that it works before creating the image.*

Verify the created image\
`docker images`

Now, we need to run this image before we publish it to Docker Hub. This is to ensure it is running well in a containerized environment with all dependencies.\
`docker run -it --rm --name cplex_test_cont cplex_test`

The run should terminate with the expected results. You should be able to see the output in your terminal.

When we are sure the image runs well, we can then publish it to Docker Hub. You must have a Docker Hub account before you can publish. So, go to [Docker Hub](https://hub.docker.com/) to create an account.

You will need to create a repository for that particular image inside your Docker Hub namespace. Although you can have 1 free private repository, it is preferable to use a public repository unless your code or runs have any sensitive information. It simplifies later configurations. When you are ready, login to the Docker Hub from terminal, create a tag and push the image using commands below:\
`docker login`

`docker tag 0420d97bcc7c your_username/cplex-java-test:v1`   
(*replace the image ID - 0420d97bcc7c is Image ID for cplex_test, and replace the docker username*)

`docker push your_username/cplex-java-test:v1`

Let the push finish** and you should be ready to run the containerized image contents in the Nautilus cluster.

Note**: *Sometimes the push takes a long time and stays stuck. You can verify if push was successful by refreshing your repository in Docker Hub. It should have the image with the relevant tag created as a linux-64 image. You can then exit from the stuck terminal without errors by pressing Ctrl+C.*

To begin working in the Nautilus cluster, check out this Quick Start guide to [Nautilus](http://ucsd-prp.gitlab.io/userdocs/start/quickstart/).

You should run the following command to find your current context\
`kubectl config get-contexts`

This will list your contexts and the current one is highlighted with an asterisk. Make sure this is “nautilus” to deploy the pod to Nautilus and not the local cluster (minikube).

```
CURRENT   NAME       CLUSTER    AUTHINFO                                      NAMESPACE
*         nautilus   nautilus   http://cilogon.org/serverA/users/<user-id>    provided-namespace
```

Now, create a configuration file pod.yml (in the current directory) to describe how we want to deploy our image. The file looks like below:

```
apiVersion: v1
kind: Pod
metadata:
  name: cplex-test-pod
spec:
  containers:
  - name: cplex-test-cont
    image: your-username/cplex-java-test:v1
```

To describe briefly, our pod will be named “cplex-test-pod” and the corresponding enclosed container will be named “cplex-test-cont”. Our image will be pulled from the public Docker Hub image that we published earlier “your-username/cplex-java-test:v1”.

We can now run this pod by using command:\
`kubectl create -f pod.yml`

And now our pod is successfully deployed. We can check this by running command:\
`kubectl get pods`

Note: *Depending on how soon you check this, the status of the pod will show if the container is being created, the pod has completed its run, or a “crash loop backoff” status. Note that if you see the crash loop backoff status, it is not actually an error. It only means the code in the pod has completed its run and Kubernetes’ tries to restart it thinking it terminated. This happens frequently if our code completes really fast (like our small example).*

To see the output of our code (anything we printed in the code), we can run the following command after the run is completed or crash loop backoff status is there.\
`kubectl logs cplex-test-pod`

If the log is too long to view in the command line, you can save the log in a text file at your local working directory. Just use the command below.\
`kubectl logs cplex-test-pod > my_log.txt`

Now that you have your results, you can note them and then you should clean up by deleting your pod. This will free up the resources for your labmates and other users of Nautilus.\
`kubectl delete pod cplex-test-pod`

**Limitations of a pod**
A limitation of the pod is that it has default computation resources assigned to it. Even if we request resources by specifying in the YAML file, it has a limit. For PRP, you may get an error message like this:

```
Error from server: error when creating "pod.yml": admission webhook "pod.nautilus.optiputer.net" denied the request: PODs without controllers are limited to 2 cores and 12 GB of RAM.
```

**Deployments**
We need a “deployment” to request more resources. An example deployment YAML named “deploy.yml” is below:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-dep
  labels:
    k8s-app: test-dep
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: test-dep
  template:
    metadata: 
      labels:
        k8s-app: test-dep
    spec:
      containers:
      - name: cplex-test-cont
        image: your_username/cplex-java-test:v2
        resources:
           limits:
             memory: "32Gi"
             cpu: 4
           requests:
             memory: "16Gi"
             cpu: 2
```

You can create and delete the deployment similar to a pod.
```
kubectl create -f deploy.yml
kubectl delete deployment test-dep
```

To view the output, you can get the pod associated with the deployment using “get pods” command and get the log accordingly.

And finally, verify none of your pods or deployments or services are running, if you are done using them. Take care not to bother with other pods/deployments/services in the same namespace (that you did not create).\
`kubectl get pods/deployments/services`

## Workflow Summary
Hence, our workflow looks like below:

```
Step 0: Make sure Docker and kubectl are working\
Step 1: Collect needed packages, write your program and compile the code if necessary
Step 2: Test your program by running in the IDE/terminal
    (Optional: Only needed the first time)
Step 3: Create Dockerfile(s) with image creation details
Step 4: Build a docker image of the (compiled) code
Step 5: Run the image in Docker to see if it is behaving correctly
    (Optional: Only needed the first time)
Step 6: Publish the docker image to your Docker Hub
    (Make the image public for easy deployment, unless you need confidentiality)
Step 7: Switch the context to “nautilus” if not already
Step 8: Create a pod.yml or deply.yml configuration file with necessary details
Step 9: Apply the configuration using kubectl
Step 10: Check the statuses and logs as needed
Step 11: Clean up
```
