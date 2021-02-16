# Getting Started with OpenShift GitOps

## Install OpenShift GitOps 

Log into OpenShift Web Console as a cluster admin and navigate to the **Administrator** perspective and then **Operators** &rarr; **OperatorHub**. 

In the **OperatorHub**, search for *OpenShift GitOps* and follow the operator install flow to install it.

![OpenShift GitOps operator](images/gitops-01.png)

![OpenShift GitOps operator](images/gitops-02.png)

![OpenShift GitOps operator](images/gitops-03.png)

Once OpenShift GitOps is installed, an instance of [Argo CD](https://argo-cd.readthedocs.io/en/stable/) is automatically installed on the cluster in the `openshift-gitops` namespace and link to this instance is added to the application launcher in OpenShift Web Console.

![Application Launcher](images/gitops-04.png)

## Accessing Argo CD

Argo CD upon installation generates an initial admin password which is stored in a Kubernetes secret. In order to retrieve this password, run the following command to decrypt the admin password:

```
kubectl get secret argocd-cluster-cluster -n openshift-gitops -ojsonpath='{.data.admin\.password}' | base64 -d
```

Click on Argo CD from the OpenShift Web Console application launcher and then log into Argo CD with `admin` username and the password retrieved from the previous step.

![Argo CD](images/gitops-05.png)

![Argo CD](images/gitops-06.png)


## Configure OpenShift with GitOps

In current Git repository, the [cluster](cluster/) directory contains an OpenShift Web Console customization. Let's configure Argo CD to recursively sync the content of the [cluster](cluster/) directory to the OpenShift cluster. Initially, we can ask set the sync policy to manual in order to be able to review changes before rolling out configurations to the cluster. 

In the Argo CD dashboard, click on the **New App** button to add a new Argo CD application that syncs a Git repository containing cluster configurations with the OpenShift cluster.

Enter the following details and click on **Create**.

* Application Name: `cluster-configs`
* Project: `default`
* Sync Policy: `Manual`
* Repository URL: `https://github.com/siamaksade/openshift-gitops-getting-started`
* Revision: `HEAD`
* Path: `cluster`
* Destination: `https://kubernetes.default.svc`
* Namespace: `default`
* Directory Recurse: `checked`

![Argo CD - Create Application](images/gitops-07.png)

> Argo CD applications can get configured declaratively by creating `Application` resources. As an alternative to the Argo CD dashboard, you can create the **cluster-configs** application by importing the following file:
>  ```
>  oc create -f argo/cluster.yaml
>  ```
> Run the following to review the created application:
>  ```
>  oc get application -n openshift-gitops
>  ```

Looking at the Argo CD dashboard, you would notice that the **cluster-configs** Argo CD application is created by is out of sync, since we configured it with manual sync policy.

![Argo CD - Cluster Config](images/gitops-08.png)

Click on the **Sync** button on the **cluster-configs** application and then on **Synchronize** button after reviewing the changes that will be rolled out to the cluster.

Once the sync is completed successfully, you would see that Argo CD reports a the configurations to be currently in sync with the Git repository and healthy. You can click on the **cluster-configs** application to check the details of sync resources and their status on the cluster. 

![Argo CD - Cluster Config](images/gitops-09.png)

![Argo CD - Cluster Config](images/gitops-10.png)

Now go back to the OpenShift Web Console and click on the **Application Launcher**. You would see that a new link is added which points at the [Red Hat Developer Blog](https://developers.redhat.com/topics/kubernetes) as the result of rolling out the content of the [cluster](cluster/) directory to the cluster. If curious, take a look inside the [cluster](cluster/) to find out which config enables adding links to the OpenShift Web Console!

![Argo CD - Cluster Config](images/gitops-11.png)

Now that the configuration sync is in place, any changes in the Git repository will be automatically detect by Argo CD and would change the status of the **cluster-configs** to `OutOfSync` which implies a drift from the desired configuration. One can set the [sync policy to automated](https://argoproj.github.io/argo-cd/user-guide/auto_sync/) in order for Argo CD to automatically roll out changes form Git repository to the cluster. 

## Deploy Applications with GitOps

In addition to configuring OpenShift clusters, many teams use GitOps workflows for continuous delivery and deploying applications in multi-cluster Kubernetes environments.

The [app](app/) directory in the current Git repository contains the Kubernetes manifests using Kustomize for deploying the sample Spring PetClinic application. Let's configure Argo CD to automatically and recursively deploy any changes made to these manifests on the OpenShift cluster.

In the Argo CD dashboard, click on the **New App** button to add a new Argo CD application that syncs a Git repository containing cluster configurations with the OpenShift cluster.

Create a new Argo CD application by clicking on the **New App** button in the Argo CD dashboard and entering the following details.

* Application Name: `spring-petclinic`
* Project: `default`
* Sync Policy: `Automatic`
* Self-heal: `checked`
* Auto-create namespace: `checked`
* Repository URL: `https://github.com/siamaksade/openshift-gitops-getting-started`
* Revision: `HEAD`
* Path: `app`
* Destination: `https://kubernetes.default.svc`
* Namespace: `spring-petclinic`
* Directory Recurse: `checked`

> You can also create the Argo CD application by importing the following file:
>  ```
>  oc create -f argo/app.yaml
>  ```

As soon as the Argo CD application is created, a sync is started in order to rollout the Spring PetClinic manifests to the `spring-petclinic` namespace.
