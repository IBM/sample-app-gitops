# sample-app-gitops

For both simplicity and consistency, all tutorials and working demos are designed using a single unique sample application, [Robot Shop](https://github.com/instana/robot-shop), which is small enough to run on a developer’s laptop but still have enough complexity to be interesting and not just a “Hello World” example.

Another reason to choose Robot Shop as sample application is that it includes all required components installed and configured for native observability which provides automatic instrumentation for complete end to end tracing and visibility into time series metrics supported by [Instana](https://www.instana.com/). This closes the gap of application deployment in production environment using GitOps by verifying the system status via application metrics.

![image](https://www.instana.com/media/shop-front-1024x622.png)

# Deploying Application

## Description

This scenario is aimed to demonstrate how you can define the desired states for an application in a Git repository, then use GitOps tools to deploy the application to a target environment and keep the actual states and desired states in sync.

As an example, in this document, we will use Robot Shop as a sample application and use Argo CD to deploy it to a cluster.

![](images/architecture.png)

## Tools

* OpenShift
* OpenShift GitOps (Argo CD)
* Helm

## Instructions

### Setup GitOps Control Panel

To setup a GitOps control panel, let's install OpenShift GitOps Operator from Operator Hub in an OpenShift cluster. Log into an OpenShift cluster using your account, navigate to `Operators -> OperatorHub`, then search `Red Hat OpenShift GitOps`.

![](images/install-gitops-operator.png)

Open the tile and click `Install` button. After waiting for a while, you should have your OpenShift GitOps Operator ready for use. Open the menu on the top right side of your OpenShift Console, then choose `Cluster Argo CD`. This will bring you to the Argo CD login page.

![](images/goto-argo-cd.png) 

To login Argo CD from UI, you can choose the option "Log in via OpenShift" on the login page, so that you can keep using the same login credential that is used to authenticate OpenShift Console.

![](images/login-argo-cd.png)

For the first time that you login using this option, you will need to grant the permission that allows OpenShift GitOps to access your OpenShift account, just click the `Allow selected permissions` button.

![](images/grant-permission.png)

Alternatively, you can also use the default admin account provided by Argo CD itself to login. To get the password, navigate to OpenShift Console, click `Workloads -> Secrets`, select `openshift-gitops` project, find a secret called `openshift-gitops-cluster`, then copy the content in its data field with key `admin.password`.

![](images/argo-cd-password.png)

!!!TODO
    Instructions to deploy Argo CD on vanilla Kubernetes.

### About GitOps Repository

For a typical application deploying using GitOps, there are always at least two repositories:

* Application repository with the source code. In our case, that is the [Git repository](https://github.com/instana/robot-shop) to host source code for the sample application Robot Shop.
* Environment configuration repository that defines the desired state of the application. In our case, that is the [Git repository](https://github.com/cloud-pak-gitops/sample-app-gitops) to host all configuration needed to deploy our sample application.

### Prepare Environment

Before you can deploy the application using GitOps, you need to prepare the environment first which is one off work. For example, to configure Argo CD with custom settings, to prepare storage for your application persistence, and so on. All these work can also be completed using GitOps.

In our case, we defined some customized health checks for Kubernetes custom resources that are not supported in Argo CD by default. We also use [rook-ceph](https://rook.io/) as the storage provider to provision persistent volume automatically for our application. All these configuration are stored inside the environment configuration repository.

To simplify the scenario, we will apply these configuration to the cluster that runs the Argo CD instance, so that the application will also be co-located with Argo CD in the same cluster.


#### Configure Argo CD

Create an Argo Application to configure Argo CD by filling out the `NEW APP` form with values according to the table below:

| Field            | Value
|:-----------------|:------
| Application Name | argocd-app
| Project          | default
| Repository URL   | https://github.ibm.com/gitops-circus/robot-shop-gitops
| Revision         | HEAD
| Path             | config/services/argocd
| Cluster URL      | https://kubernetes.default.svc
| Namespace        | openshift-gitops

Click `SYNC` button after you finish creating the Application if you did not choose `Automatic` sync policy when define the Application.

#### Setup Storage

Create an Argo Application to setup storage by filling out the `NEW APP` form with values according to the table below:

| Field            | Value
|:-----------------|:------
| Application Name | rook-ceph-app
| Project          | default
| Repository URL   | https://github.ibm.com/gitops-circus/robot-shop-gitops
| Revision         | HEAD
| Path             | config/services/rook-ceph
| Cluster URL      | https://kubernetes.default.svc
| Namespace        | rook-ceph

Click `SYNC` button after you finish creating the Application if you did not choose `Automatic` sync policy when define the Application. Wait for a while till all Applications, including those child ones that are created automatically by the root one get all their health checks passed.

![](images/prepare-env.png)

### Deploy Application

Create an Argo Application to deploy the sample application by filling out the `NEW APP` form with values according to the table below:

| Field            | Value
|:-----------------|:------
| Application Name | sample-apps
| Project          | default
| Repository URL   | https://github.ibm.com/gitops-circus/robot-shop-gitops
| Revision         | HEAD
| Path             | config/apps/robot-shop
| Cluster URL      | https://kubernetes.default.svc
| Namespace        | n/a

Click `SYNC` button after you finish creating the Application if you did not choose `Automatic` sync policy when define the Application. Wait for a while till all Applications, including those child ones that are created automatically by the root one get all their health checks passed.

### Access Application

To access the application, you can go back to the OpenShift Console, and open the menu on the top right side of the page. There will be a new menu item added for the application that we created just now.

![](images/goto-sample-app.png)

Click that menu item will bring you to the application home page.

![](https://www.instana.com/media/shop-front-1024x622.png)

Congratulations! You have successfully deployed the sample application Robot Shop using GitOps.

### Uninstall Application

To uninstall the application, choose the Application named `robot-shop-app` from Argo CD `Applications` page, then click `DELETE` button. This will bring down the application. Wait for a while till the Application along with its child Applications are all completely deleted, then go to check the menu on the top right side of OpenShift Console, you will see the menu item for the application has been removed.
