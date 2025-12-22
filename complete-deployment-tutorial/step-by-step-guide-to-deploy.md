# Deploying n8n on Azure Kubernetes Service

This guide shows you how to self-host n8n on Azure. It uses n8n with Postgres as a database backend using Kubernetes to manage the necessary resources.

### Prerequisites

You need [The Azurecommand line tool](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

### Hostingoptions

Azure offers several ways suitable for hosting n8n, including Azure Container Instances (optimized for running containers), Linux Virtual Machines, and Azure Kubernetes Service (containers running with Kubernetes).

This guide uses the **Azure Kubernetes Service (AKS)** as the hosting option. Using Kubernetes requires some additional complexity and configuration but is the best method for scaling n8n as demand changes.

The steps in this guide use a mix of the Azure UI and command line tool, but you can use to accomplish most tasks.

## Steps to Deploy n8n to AKS:

This part tells you the straightforward step to deploy your n8n to AKS cluster. Whereas, the next part gives you more detail about, what you are actually doing.

### Step 1: Deploy Azure Kubernetes Service and Connect to it

From [the Azure portal](https://portal.azure.com/) select **Kubernetes services**.

From the Kubernetes services page, select **Create** > **Create a Kubernetes cluster**.

You can select any of the configuration options that suit your needs, then select **Create** when done.

After creating an AKS cluster. You must connect to that server. You can find the connection details for a cluster instance by opening its details page and then the **Connect** button. The resulting code snippets shows the steps to paste and run into a terminal to change your local Kubernetes settings to use the new cluster.

### Step 2: Clone Configuration GitHub repository

After you have successfully done the first step, then you have to clone my GitHub repository. This repository contain all the deployment files in the yaml format including the n8n, and Persistent volume claim deployments.

Kubernetes and n8n require a series of configuration files. You can also clone these from [this repository](https://github.com/n8n-io/n8n-hosting). The following steps tell you which file configures what and what you need to change.

Clone therepository with the following command:

``` git clone https://github.com/afraz-hassan/Deploying-n8n-on-Azure-Kubernetes-Service.git   ```

And change directory:

````cd Deploying-n8n-on-Azure-Kubernetes-Service\complete-deployment-tutorial\kube-menifest````

**Feel Completely free to edit the yaml files in the cloned repository according to your requirements.**

### Step 3: Configure Postgres

Its time to deploy our Postgres server in the Azure. You can either do it in Azure CLI, in declarative yaml format, or by visiting azure portal through GUI. Easiest way is to do it through Azure Portal.

Simply, go to your azure portal, search for postgres and select **Postgres Flexible Server.**

Click on create and select any of the configuration options that suit your needs, then select **Create** when done.

Click on the connect option in the postgres server in azure portal. You will get the credentials like username, host, database name and others. Keep a note of them.

### Step 4: Review all the configuration in cloned github repo

Then, review all the naming configurations and all the other settings in the yaml files in all the yaml files from the cloned github repository.

Edit the postgres environment variable like username, host, database name, password and other, in the configmap.yaml in the cloned github repository.

Feel free to edit the environment variable, naming conventions and resource quotas accordingto your requirements.

But there are few things which are sensitive (such as docker image etc). If you change them then the successful deployment may be compromised.

**Step 5: Apply the changes and verify**

Send all the manifests to the cluster with the following command:

```kubectl apply -f .```

**Namespace error**

You may seean error message about not finding an "n8n" namespace as that resources isn't ready yet. You can run the same command again, or apply the namespace manifest first with the following command:

```kubectl apply -f namespace.yaml```

### Step 6. Setup DNS (Optional)

n8ntypically operates on a subdomain. Create a DNS record with your provider forthe subdomain and point it to the IP address of the n8n service. Find the IP address of the n8n service from the Services & ingresses menu item of the cluster you want to use under the External IP column. You need to add the n8n port, "5678" to the URL.

**OR Access it through IP directly**

Find the IP address of the n8n service from the Services & ingresses menu item of the cluster you want to use under the External IP column. Click on that IP in front of n8n, it will take you to new tab in your browser and open your n8n dashboard.

**N8N Cookies Error**

Maybe, when you open your n8n, you may see a secure cookies error if you are routing through http. It is recommended to shift to https for secure routing, But if you want to use http then go to configmaps.yaml file and add the below environment variable into it and run

```Kubectl apply -f .```

[N8N\_SECURE\_COOKIE=false](https://www.google.com/search?q=N8N_SECURE_COOKIE=false&rlz=1C1UEAD_enPK1089PK1089&oq=n8n+cookies+error&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIICAEQABgWGB4yCggCEAAYgAQYogTSAQg2ODc5ajBqN6gCALACAA&sourceid=chrome&ie=UTF-8&ved=2ahUKEwj2odvbh8SRAxWqTkEAHVJ0MqsQgK4QegQIARAE) 

## MoreInformation About the Cloned GitHub Repository:

**Configure volume for persistent storage**

To maintain data between pod restarts, the Postgres deployment needs a persistent volume.The default storage class is suitable for this purpose and is defined inthe postgres-claim0-persistentvolumeclaim.yaml manifest.

**Specializedstorage classes**

If you have specialised or higher requirements for storage classes, [read more onthe options Azure offers in the documentation](https://learn.microsoft.com/en-us/azure/aks/concepts-storage#storage-classes).

**Postgres environment variables**

Postgresneeds some environment variables set to pass to the application running in the containers.

The example postgres-secret.yaml file contains placeholders you need to replace with your own values. Postgres will use these details when creating the database.

**Configure n8n**

**Create avolume for file storage**

While not essential for running n8n, using persistent volumes is required for:

*   Using nodes that interact with files, such as the binary data node.
    

*   If you want to persist [manual n8n encryption keys](https://docs.n8n.io/hosting/configuration/environment-variables/deployment/) between restarts. This saves a file containing the key into file storage during startup.
    

The n8n-claim0-persistentvolumeclaim.yaml manifest creates this, and the n8n Deployment mounts that claim inthe volumes section of the n8n-deployment.yaml manifest.

…

volumes:

  - name: n8n-claim0

    persistentVolumeClaim:

      claimName: n8n-claim0

…

**Podresources**

[Kubernetes letsyou](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) optionally specify the minimum resources application containers need and the limits theycan run to. The example YAML files cloned above contain the following inthe resources section of the n8n-deployment.yaml file:

…

resources:

  requests:

    memory: "250Mi"

  limits:

    memory: "500Mi"

…   

This definesa minimum of 250mb per container, a maximum of 500mb, and lets Kubernetes handle CPU. You can change these values to match your own needs. As a guide, here are the resources values for the n8n cloud offerings:

*   **Start**: 320mb RAM, 10 millicore CPU burstable
    

*   **Pro (10k executions)**: 640mb RAM, 20 millicore CPU burstable
    

*   **Pro (50k executions)**: 1280mb RAM, 80 millicore CPU burstable
    

**Optional: Environment variables**

You canconfigure n8n settings and behaviors using environment variables.

Createan n8n-secret.yaml file. Refer to [Environment variables](https://docs.n8n.io/hosting/configuration/environment-variables/) for n8n environment variables details.

**Deployments**

The two deployment manifests (n8n-deployment.yaml and postgres-deployment.yaml) define the n8n andPostgres applications to Kubernetes.

The manifests define the following:

*   Send the environment variables defined to each application pod
    

*   Define the container image to use
    

*   Set resource consumption limits with the resources object
    

*   The volumes defined earlier and volumeMounts to define the path in the container to mount volumes.
    

*   Scaling and restart policies. The example manifests define one instance of each pod. You should change this to meet your needs.
    

**Services**

The two service manifests (postgres-service.yaml and n8n-service.yaml) exposethe services to the outside world using the Kubernetes load balancer using ports 5432 and 5678 respectively.

**Send toKubernetes cluster**

Send all the manifests to the cluster with the following command:

```kubectl apply -f .```

**Namespaceerror**

You may seean error message about not finding an "n8n" namespace as thatresources isn't ready yet. You can run the same command again, or apply thenamespace manifest first with the following command:

```kubectl apply -f namespace.yaml```

**Static IPaddresses with AKS**

[Read this tutorial](https://learn.microsoft.com/en-us/azure/aks/static-ip) for more details on how to usea static IP address with AKS.

**Delete resources**

If you are doingjust for testing or practice, then remove the resources created by themanifests with the following command:

```kubectl delete -f .```


OfficialDocumentation: [https://docs.n8n.io/hosting/installation/server-setups/azure/](https://docs.n8n.io/hosting/installation/server-setups/azure/)
