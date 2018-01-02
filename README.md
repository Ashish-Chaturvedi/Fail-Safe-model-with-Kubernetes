# Fail-Safe-model-with-Kubernetes
Fail- safe setup of tools with Kubernetes



FAIL-SAFE MODEL FOR DEVOPS PIPELINE

PREFACE:
DevOps pipeline will always be available and it will auto-heal in case of a crash. 
All the data will be preserved with rollback from 2-5 hours to just 1-2 seconds.
It has moved beyond SDLC. It’s “NoOps” i.e. no operations team is needed for its operations.
System is able to come up by itself in case of a failure.

SETUP: 
(Kubernetes Cluster)
CASE:  Google Cloud Computing (GCP):

1.	Set compute zone:
$ gcloud config set compute/zone asia-south1-a

2.	Create Kubernetes cluster (1node):
$ gcloud container clusters create x404-devops --num-nodes=1

3.	Create a node-pool for larger storage:
$ gcloud container node-pools create larger-pool --cluster x404-devops --machine-type=n1-standard-4 --num-nodes=1

4.	List the available node-pools:
$ gcloud container node-pools list --cluster x404-devops

5.	Get Kubernetes Nodes that fall under default-pool(small compute size):
$ kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool

6.	“Cardon” the particular node that falls under default-pool:
$ kubectl cordon gke-x404-devops-default-pool-425700fb-3tft

7.	(Optional) Drain current pods to new pool(only if any pod exist):
$ kubectl drain --force gke-x404-devops-default-pool-425700fb-3tft

8.	(Optional, if we have more than one node) Label the kube minion:
$ kubectl label nodes gke-x404-devops-larger-pool-425700fb-3sdf env=devops-dev
Components used for setup:
•	Pods
•	Dynamic Volumes
•	Dynamic Volume Claims
•	Services 

Tools:
•	Jenkins
•	Sonar
•	Nexus3

SETUP:
(Directories)
1.	SSH into the Kubernetes minion and create directories for storage:
$ mkdir /opt/jenkins
$ mkdir /opt/sonar
$ mkdir /opt/nexus

2.	Give Permissions for access to dynamic volume
$ chmod 777 –R /opt/jenkins and for other directories as well


SETUP:
(Containers)
CASE:  JENKINS
1.	Create template for Dynamic Volume (Do care about storage for particular component) 
$ vi per_jenkins_volume.yaml

	Content:
kind: PersistentVolume
apiVersion: v1
metadata:
  name: per-jenkins-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    				path: "/opt/jenkins"


 

	
2.	vi per_jenkins_claim.yaml

Content:
 			kind: PersistentVolumeClaim
			apiVersion: v1
			metadata:
  name: per-jenkins-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      			  storage: 3Gi

 

3.	vi create_jenkins_pod.yaml

Content:	
kind: Pod
apiVersion: v1
metadata:
  name: jenkins
  labels:
    app: jenkins
spec:
  volumes:
    - name: jenkins-storage
      persistentVolumeClaim:
       claimName: per-jenkins-claim
  containers:
    - name: jenkins-container
      image: mindtreestackdevops/jenkins:v1
      ports:
        - containerPort: 8080
          name: "jenkins-server"
      volumeMounts:
        - mountPath: "/var/jenkins_home/"
          name: jenkins-storage
  nodeSelector:
      env: devops-dev

 

4.	Execute following commands to create pods:
$ kubectl create <template_name>.yaml

5.	Expose pod to external access
$ kubectl expose pod <pod_name> --port=8080/9000/8081 --type=LoadBalancer –name=jenkins/sonar/nexus
