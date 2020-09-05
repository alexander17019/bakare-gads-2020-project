## Deploying To Kubernetes Version 1.6


## Objectives 
   *Scaling and management of containers in production using deployments via kubernetes 
   
   *Rolling update using deployments
   
   *Canary deployments
   
   *Blue-green deployments
   
   *Using the web User Interface 
   
   
## Step one 
Setting up the work environment 

1) login into the google cloud platform ,after logining in ,You would be prompted intp the google cloud console. 
We need to make sure the following APIS are enabled in cloud console 

* Kubernetes Engine API
* Google Container Registry API
* Go to navigation menu , click APIS & services
  Scroll down and confirm that your APIS are enabled. If any is missing ,click ENABLE API AND SERVICES at the top and search for the API(S) missing and enable it for the project.
  
  
##  
 Activate Google Cloud Shell
 * In the console ,on the top right toolbar,click open cloud shell button close to the search button. An enivronment will pop decribed as cloud shell with the continue prompt just below ,click continue and the cloud shell environmemt is initiated.
 
 * The project should be set to your current project_ID BUT if for some reason it is not associated to your project_ID. Use this command to associate it to your project 
  * Gcloud config set project {PROJECT_ID}
  
##
Define your zone as a project default zone. This way you do not need to specify --zone parameter in gcloud commands.
 
*gcloud config set compute/zone us-central1-a

## 
Get the sample code for creating and running containers and deployments:

*git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git

##
Start your Kubernetes cluster with 5 nodes.

*cd orchestrate-with-kubernetes/kubernetes

##
In Cloud Shell, run the following command to start a Kubernetes cluster called bootcamp that runs 5 nodes.

*gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"


## OBJECTIVES 
 
 Learn about deployments
 
 Step one
 Run the explain command in kubectl to tell you about the deployment object.
 
 *kubectl explain deployment
 
 Step two
 Run the command with the --recursive option to see all of the fields.
 
 *kubectl explain deployment --recursive
 
 Step three
 Use the explain command as you go through the lab to help you understand the structure of a deployment object and understand what the individual fields do.
 
 *kubectl explain deployment.metadata.name
 
 
 Create a deployment 
 
 Step one
 Examine the deployment configuration file.
 
 *cat deployments/auth.yaml
 
 Step two 
 Create the deployment object using kubectl create
 
 *kubectl create -f deployments/auth.yaml
 
 Step three
 Verify that it was created
 
 *kubectl get deployments
 
 Step four 
 kubernetes creates a ReplicaSet for the deployment
 
 *kubectl get replicasets
 
 Step five 
 To view the pods created for your deployment. A single pod was created when a ReplicaSet was created
 
 *kubectl get pods
 
 Step six
 With your pod running, it's time to put it behind a service. Use the kubectl create command to create the auth service.
 
 *kubectl create -f services/auth.yaml
 
 Step seven
 Do the same to create and expose the hello and frontend deployments.
 
 *kubectl create -f deployments/hello.yaml
 *kubectl create -f services/hello.yaml 
 
 Step eight
 Create a ConfigMap and secret for the frontend 
 
 *kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf

 *kubectl create secret generic tls-certs --from-file tls/

 *kubectl create -f deployments/frontend.yaml
 
 *kubectl create -f services/frontend.yaml
 
 Step nine
 Interact wiht the frontend service and record itd external ip address, do run this command a few times for the external address to be populated to the external ip section of the frontend service and curl the service.
 
 *kubectl get services frontend
 
 *curl -ks http://<External-IP>
 After the curl of the service you will get a hello response. Use the output templating feature of kubectl to run curl as a one-line command.
 *curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
 
 
 Scale the deployment : Update the spec.replicas field to scale the deployment
 
 Step one 
 Run the kubectl explain command to see an explanation of the field 
 
 *kubectl explain deployment.spec.replicas
 
 Step two
 You can update the replicas field most easily using the kubectl scale command.
 
 *kubectl scale deployment hello --replicas=5
 
 Step three
 verify the five pods are running
 
 *kubectl get pods | grep hello- | wc -l
 
 Step four
 Scale back the nods back to three
 
 *kubectl scale deployment hello --replicas=3
 
 Step five
 verify the correct number of pods
 
 *kubectl get pods | grep hello- | wc -l
 
 ## Objective two : Rolling updates
 
 Step one
 Run the following command to update the deployment 
 
 Step two
 Change the image in containers section with the following  ,then save and exit
 
 *containers:
- name: hello
  image: kelseyhightower/hello:2.0.0
  
Note: The editor uses vi commands
1. Use the arrow keys to hover 
2. Type r to replace or go to the section you want to change and type i for insert to write
3.After you are done writing ,type :wq! to quit the file 

Step three
You can see the new ReplicaSet that Kubernetes creates.

*kubectl get replicaset

Step four 
view the new entry in the rollout history

*kubectl rollout history deployment/hello

Step five 
Pause a rollout update if you detect a problem with a rollout

*kubectl rollout pause deployment/hello

Step six
Verify the current state of the rollout

*kubectl rollout status deployment/hello

Step seven 
verify the pods 

*kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

Step eight 
Use the resume command to continue the rollout

*kubectl rollout resume deployment/hello

Step nine 
Run the status command to verify the rollout is complete

*kubectl rollout status deployment/hello

Step ten 
Rollback an update if a bug occurs in the new version but be warned the users will experience some issues . so the rollback is useful to rectify such issues by taking the application to a point or version it was working perfectly .

*kubectl rollout undo deployment/hello

Step eleven
Verify the rollback

*kubectl rollout history deployment/hello

Step twelve
Verify all pods rolled back to previous verions 

*kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

##Objective three : CANARY DEPLOYMENTS

Step one
Examine a file that create a canary deployment for your new version

*cat deployments/hello-canary.yaml

Step two
Create the canary deployment

*kubectl create -f deployments/hello-canary.yaml

Step three
Verify the canary deployment is created,you have the hello and hello canary deployments

*kubectl get deployments

Step four 
Verify both hello versions being served by requests and run the command several times so ensure the first version takes most of the requests.

*curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

Step five 
Delete it and the services as follows

*kubectl delete deployment hello-canary


Objective Four: Create Blue-green deployment
Update the service to use the blue deployment 

*kubectl apply -f services/hello-blue.yaml

Step one 
create the green deployment

*kubectl create -f deployments/hello-green.yaml

Step two 
Verify the blue deployment(1.0.0) is still being used to accept request 

*curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

Step three
Update service to use the green deployment 

*kubectl apply -f services/hello-green.yaml

Step four 
Verify the green deployment is being used

*curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

Step five 
if there is a bug issue ,you can rollback to older version,While the green deployment is still running, simply update the service to the old (blue) deployment.

*kubectl apply -f services/hello-blue.yaml

Step six
Verify the blue deployment is being used 

*curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version


##Objective five : Using the web user interface
 
 Step one 
 In Cloud Console, click Navigation menu and select Kubernetes Engine.

Select a resource in the tab on the left, for example Services & Ingress.





 
 
 
 
 
 
 



 
 
  

   