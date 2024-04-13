# Introduction to Containers with Docker, Kubernetes &amp; OpenShift
> From IBM on Coursera
## Peer-graded Assignment: Build and Deploy a Simple Guestbook
Results available under [screenshots/assignment](https://github.com/j5py/containers/tree/main/screenshots/assignment) (file names imposed by the instructions)
### Build the Guestbook app
Clone the git repository that contains the artifacts needed for this lab
```Shell
[ ! -d 'guestbook' ] && git clone https://github.com/ibm-developer-skills-network/guestbook
```
Change to the directory for this lab
```Shell
cd guestbook/v1/guestbook
```
Complete the Dockerfile ( [multi-stage builds](https://docs.docker.com/build/building/multi-stage/) ) with the necessary commands to build and push your image
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
## Optional: Deploy Guestbook App from the OpenShift Internal Registry
Results available under [screenshots/optional](https://github.com/j5py/containers/tree/main/screenshots/optional)
### Deploy
Create an ImageStream that points to your image in IBM Cloud Container Registry
```Shell
oc tag us.icr.io/$MY_NAMESPACE/guestbook:v1 guestbook:v1 --reference-policy=local --scheduled
```
1. Click on **Open OpenShift Console** under **Skills Network Toolbox**
2. From **Developer** click **Add**, then  **Container image**
3. Switch to **Image stream tag from internal registry**, then select your project
4. Keep all the default values and hit **Create**
5. From **Topology** click the `guestbook` Deployment, this should take you to the **Resources**
    - Do not delete the `opensh.console` Deployment in **Topology** as this is essential for the OpenShift Console to function properly
### Update
Let’s update the Guestbook app and see how OpenShift’s image streams can help us update our apps with ease
```Shell
docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```
```Shell
oc import-image guestbook:v1 --from=us.icr.io/$MY_NAMESPACE/guestbook:v1 --confirm
```
Switch to **Administrator** > **Builds** > **ImageStreams** > `guestbook` > **History**
> View the guestbook in the browser again, OpenShift imported the new version of our image, and since the Deployment points to the image stream, it began running this new version as well
### Storage
**Topology** > `guestbook-app` > **Actions** > **Delete application**
```Shell
cd ../../v2
```
To familiarize yourself with the Deployment configuration for the Redis master
> Redis is an open source, in-memory data structure store, used as a database, cache and message broker
```Shell
cat redis-master-deployment.yaml
```
Create the Redis master Deployment
```Shell
oc apply -f redis-master-deployment.yaml && oc get deployments
```
List Pods to see the Pod created by the Deployment.
```Shell
oc get pods
```
to familiarize yourself with the Service configuration for the Redis master
```Shell
cat redis-master-service.yaml
```
Create the Redis master Service
```Shell
oc apply -f redis-master-service.yaml
```
**Developer** > **Topology** > `redis-master` > **resources**

To familiarize yourself with the Deployment configuration for the Redis slave
```Shell
cat redis-slave-deployment.yaml
```
Create the Redis slave Deployment
```Shell
oc apply -f redis-slave-deployment.yaml && oc get deployments
```
List Pods to see the Pod created by the Deployment
```Shell
oc get pods
```
To familiarize yourself with the Service configuration for the Redis slave
```Shell
cat redis-slave-service.yaml
```
Create the Redis slave Service
```Shell
oc apply -f redis-slave-service.yaml
```
**Developer** > **Topology** > `redis-slave` > **resources**

Now it’s time to deploy the second version of the Guestbook app, which will leverage Redis for persistent storage
1. From **Developer** click **Add**, then **Import from Git**
    - `https://github.com/ibm-developer-skills-network/guestbook`
2. **Show advanced Git options**
    - **Context dir** > `/v2/guestbook`
    - **Target port** > `3000`

**Developer** > **Topology** > `guestbook` > **Routes**
```Shell
oc status
```