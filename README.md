# Project Report

## Project Goal

## What is Vertical Pod AutoScaler (VPA)?

## Tech Stack for this Demo
- RedHat OpenShift Cluster Platform (OCP)
  -  Note: VPA is provided by Kubernetes, so you can work with VPA with any other cloud platform, but we have used OCP and hence the instructions are provided for that here.
- Resource Consumer application
  -  Resource Consumer is an application tool to to generate the CPU/Memory workload inside a container.
  -  It starts the HTTP server when deployed, listens to requests at default port 8080 and handles those reqests.
  -  We use this application to test the vertical pod autoscaling by sending various HTTP load requests to resource consumer. 
 

## Steps to setup VPA in your namespace

### Prerequisites
We are assuming that you already have the following:
- a RedHat OpenShift Cluster Platform (OCP) and namespace ready to work on. 
- admin access in your cluster (This is required to install VPA operator in your cluster)

### Install VPA operator in your cluster 
1. In OCP, click on "Operators" -> "Operator Hub"
2. Choose " VerticalPodAutoscaler" and click Install
3. On the Install Operator page, ensure that the Operator recommended namespace option is selected. This installs the Operator in the mandatory openshift-vertical-pod-autoscaler namespace, which is automatically created if it does not exist.
4. Click Install.
5. Verify the installation by listing the VPA Operator components:
  1. Go to Workloads → Pods.
  2. Select the openshift-vertical-pod-autoscaler project from the drop-down menu and verify that there are four pods running.
  3. Navigate to Workloads → Deployments to verify that there are four deployments running.
6. [Optional] Verify the installation in the OCP CLI using the following command:
`$ oc get all -n openshift-vertical-pod-autoscaler`



### Inital Setup in your namespace

#### Create a deployment
1. Go to "Workloads" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Deployments" on the left side navigation bar.
3. Click on "Create Deployments" button which will open the yaml file. 
4. The yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-depl.yaml).
6. [Optional] you can change the name of the deployment, containers and app.
7. Click on "Create"

#### Create a service
1. Go to "Networking" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Services" on the left side navigation bar.
3. Click on "Create Service" button which will open the yaml file.
4. The above yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-service.yaml).
6. Verify the "app" entry matches the deployment of app you created.
7. [Optional] you can change the name of the service.
8. Click on "Create"

#### Create a route 
1. Go to "Networking" in your namespace under "Administrator" tab in moc/smaug.
2. Click on "Routes" on the left side navigation bar.
3. Click on "Create Route" button which will open the yaml file.
4. The above yaml file will have preloaded entries, delete all of them.
5. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-route.yaml).
6. Verify the "name" entry of the service under spec: matches the name of the service you created.
7. [optional] you can change the name of the service.
8. Click on "Create"

NOTE: We use this route to send requests to our resource consumer application deployed. 

### Create VPA Custom Resource in your namespace 
1. In Administrator tab, click on "Home" -> "API Explorer".
2. Type "VerticalPodAutoscaler" in filter by kind text box on the top right.
3. Click on "VerticalPodAutoscaler". (any version (v1/v1beta) is fine).
4. Click on "Instance" -> "Create VerticalPodAutoscaler" button, which will open the yaml file.
5. The above yaml file will have preloaded entries, delete all of them.
6. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-vpa.yaml)
7. Change the following entries in the pasted yaml file:
  1. Change the "namespace" under metadata, to your namespace   
  2. Change the "updateMode" to Initial. `updateMode: Initial` (since we are testing this one first)
9. Click on "Create"

### Create VPA Controller
1. In Administrator tab, click on "Home" -> "API Explorer".
2. Type "VerticalPodAutoscalerController" in filter by kind text box on the top right.
3. Click on "VerticalPodAutoscalerController".
4. Click on "Instance" -> "Create VerticalPodAutoscalerController" button, which will open the yaml file.
5. The above yaml file will have preloaded entries, delete all of them.
6. Now, [copy and paste this exact yaml entries](https://github.com/TheGreymanShow/vertical-pod-autoscaler-operator/blob/main/install/rc-vpa-controller.yaml)
7. Change the following entries in the pasted yaml file:
  1. Change the "namespace" under metadata, to your namespace   
9. Click on "Create"

## How to see VPA recommendation? 
You can see VPA recommendations through ocp command line in terminal or in your OCP web console. We will list both the ways here

### Through OCP web console
1. In Administrator tab, click on "Home" -> "API Explorer".
2. Type "VerticalPodAutoscalerController" in filter by kind text box on the top right.
3. Click on "VerticalPodAutoscalerController"
4. Select the instance you created. 
5. In the yaml file, your should see recommendations like below

TODO: Attach a photo of recommendations here. 

## How to check VPA changing the pod's metrics in Initial mode?

### Change the workload
For the recommendations to change, we need to change the workload. We can do this by the below steps.

1. Copy the route you created for your deployment. 
2. Copy and paste the following command in your terminal. 

`curl --data "millicores=500&durationSec=300" <your-route>/ConsumeCPU`

The above commands sends a request to increase the CPU load to 500 millicores for 300 seconds (5 minutes). Now you need to wait for 5 minutes for the request to finish.

### Check the recommendations
Now check the recommendations, it should be changed from the initial one. 
If not, increase the `durationSec=1800` (30 minutes) and try again. 
Once the recommendations changes, move to the next step.

### Delete the pod
- Delete the pod of your deployment, the pod will be automatically recreated since the resource policy is rolling-update. 
- The newly created pod will have the request value equal to the 'target' provided by the vpa recommendations.  

## How to check VPA changing the pod's metrics in Auto mode?

