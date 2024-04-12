# Introduction to Containers with Docker, Kubernetes &amp; OpenShift
> From IBM on Coursera
## Peer-graded Assignment: Build and Deploy a Simple Guestbook 
### Build the Guestbook app
Clone the git repository that contains the artifacts needed for this lab
```Shell
[ ! -d 'guestbook' ] && git clone https://github.com/ibm-developer-skills-network/guestbook
```
Change to the directory for this lab
```Shell
cd guestbook/v1/guestbook
```
Complete the Dockerfile with the necessary Docker commands to build and push your image
- The FROM instruction initializes a new build stage and specifies the base image that subsequent instructions will build upon
- The COPY command enables us to copy files to our image
- The ADD command is used to copy files/directories into a Docker image
- The RUN instruction executes commands
- The EXPOSE instruction exposes a particular port with a specified protocol inside a Docker container
- The CMD instruction provides a default for executing a container, or in other words, an executable that should run in your container

Export your namespace as an environment variable so that it can be used in subsequent commands
```Shell
export MY_NAMESPACE=sn-labs-$USERNAME
```
Build the Guestbook app
```Shell
docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1
```
Push the image to IBM Cloud Container Registry
```Shell
docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```
Verify that the image was pushed successfully
```Shell
ibmcloud cr images
```
To check your namespace
```Shell
ibmcloud cr namespaces
```
Apply the Deployment
```Shell
kubectl apply -f deployment.yml
```
To view your app, open a new terminal
```Shell
kubectl port-forward deployment.apps/guestbook 3000:3000
```
Launch your app on port 3000 (Skills Network)
### Autoscale the Guestbook app using Horizontal Pod Autoscaler
Autoscale the Deployment
```Shell
kubectl autoscale deployment guestbook --cpu-percent=5 --min=1 --max=10
```
You can check the current status of the newly-made Horizontal Pod Autoscaler
```Shell
kubectl get hpa guestbook
```
To generate load on the app and observe the autoscaling, open another new terminal
```Shell
kubectl run -i --tty load-generator --rm --image=busybox:1.36.0 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- <your-app-url>; done"
```
To observe the replicas increase in accordance with the autoscaling
```Shell
kubectl get hpa guestbook --watch
```
### Perform rolling updates and rollbacks on the Guestbook app
To build and push your updated app image
```Shell
docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```
Apply other changes
```Shell
kubectl apply -f deployment.yml
```
Open a new terminal and run the port-forward command again to start the app
```Shell
kubectl port-forward deployment.apps/guestbook 3000:3000
```
To see the history of Deployment rollouts
```Shell
kubectl rollout history deployment/guestbook
```
To see the details of Revision of the Deployment rollout
```Shell
kubectl rollout history deployments guestbook --revision=2
```
To get the ReplicaSets and observe the Deployment which is being used now
```Shell
kubectl get rs
```
To undo the Deploymnent
```Shell
kubectl rollout undo deployment/guestbook --to-revision=1 && kubectl get rs
```