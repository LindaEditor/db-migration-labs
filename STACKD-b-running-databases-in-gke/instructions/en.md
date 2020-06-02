# Running Databases in GKE

## Overview

In this lab, you will create a Kubernetes cluster and then deploy databases into it. You will see two ways to deploy the databases. First using your own configuration code and then using a Kubernetes package manager called Helm.

### Objectives

In this lab, you will learn how to perform the following tasks:

*   Create a GKE Cluster
*   Deploy MySQL onto the Cluster
*   Use Helm to deploy MySQL on the cluster

## Task 0. Lab Setup

In this task, you use Qwiklabs and perform initialization steps for your lab.

### Access Qwiklabs

![[/fragments/startqwiklab]]

After you complete the initial sign-in steps, the project dashboard appears.

![GCP Project Dashboard](img/gcpprojectdashboard.png)

Click __Select a project__, highlight your _GCP Project ID_, and click
__OPEN__ to select your project.

![[/fragments/cloudshell]]

## Task 1. Create a GKE Cluster

1.  In the Navigation menu ( ![Menu](img/menu.png) ), click on **Kubernetes Engine > Clusters**.

1.  Click on the **Create Cluster** button. Accept all the defaults and click the **Create** button at the bottom. It will take a couple minutes for the cluster to be ready. 

1.  When the cluster is ready, click on the **Connect** button. Notice the command for connecting to the cluster is specified. Click the **Run in Cloud Shell** button to open Cloud Shell with the command entered. Hit Enter to run the command.

1.  Now you are connected to the cluster and ready to deploy a program. Test the connection with the following kubectl command. This command should return a list of the three virtual machines that make up this cluster. 

```
kubectl get nodes
```

## Review

You just created a Kubernetes cluster, next you will configure and deploy MySQL to run in it.

## Task 2. Deploy MySQL onto the Cluster

1.  You need a root password for the database. You will store the password as a Kubernetes secret. Enter the following command to create the secret. The secret is a key-value pair. In this case the key is `ROOT_PASSWORD` and the value is `password`

```
kubectl create secret generic mysql-secrets --from-literal=ROOT_PASSWORD="password"
```

1.  Create a folder for the configuration files you will create and change to it.

```
mkdir mysql-gke
cd mysql-gke
```

1.  Now you will create the Kubernetes configuration files. In Cloud Shell, click on the **Open Editor** button. Make sure you click on the `mysql-gke` folder on the left. <p>Select **File > New File** and name the file `volume.yaml`.</P>

1.  Enter the following YAML code and save the file. This reserves 1 gigabyte of disk space for the MySQL database.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-data-disk
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

<aside><p><strong>Note </strong>the name `mysql-data-disk`. This name will be used in the next file.</p></aside>

1.  Create another new file called `deployment.yaml`, and paste the following code into it. This configures the MySQL database. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              subPath: "mysql"
              name: mysql-data
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: ROOT_PASSWORD
            - name: MYSQL_USER
              value: testuser
            - name: MYSQL_PASSWORD
              value: password
      volumes:
        - name: mysql-data
          persistentVolumeClaim:
            claimName: mysql-data-disk
```

<aside><p><strong>Note the following:</strong></p></aside>

*  On line 19, the MySQL docker image is specified.
*  Starting at line 26, an environment variable is created for the database root password using the secret you created earlier. There are also variables to create a test user with a simple password. 
*  In the last line, notice the disk space you allocated in the previous file is used. 

1.  The database needs a service so it can be accessed. Create a third file called `service.yaml` and enter the followng into it. 

```
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
```

<aside><p><strong>Note:</strong> This creates a service that provides access to the database from within the cluster, that forwards requests to the MySQL database.</p></aside>

1.  In Cloud Shell, click the **Open Terminal** button to return to the command line. Make sure you are in the right folder and type `ls` to verify you have your three YAML files. 

1.  Now enter the following kubectl commads to deploy your database.

```
kubectl apply -f volume.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

1.  Wait a minute for the resources to be created. Then enter the following command. ***You should see the pod running that has your database installed. If it is not running yet, wait a little while longer and try again.***

```
kubectl get pods
```

1. The database is only accessible from inside the cluster. At this point there are no client applications. So, you will just access the pod with the database and see if it is running. <p>From the last command, copy the name of the pod to the clipboard. It will begin with `mysql-deployment-` followed by a unique string. Enter the following command, ***but replace the pod name with your pod's name***.

```
kubectl exec -it mysql-deployment-76fdc44468-rfhbp /bin/bash
```

1.  Now you're at a bash prompt within the MySQL pod. Enter the following to log into MySQL, when prompted, enter the password `password`.

```
mysql -u root -p
```

1.  This should give you a mysql prompt. Run the following command.

```
show databases;
```

1.  Create a new database.

```
create database pets;
```

1.  See if your database was created by entering `show databases;` again.

1.  Type `exit` to exit MySQL, type `exit` again to return to the Cloud Shell command prompt.

1.  You can now remove everything that was created by entering the following commands.

```
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kubectl delete -f volume.yaml
```

## Review

You have deployed a MySQL database to a Kubernetes cluster using Kubernetes configuration files. Helm is a package manager for Kubernetes. It can make deploying databases and other applications easier on a Kubernetes cluster. You will use it next.


## Task 2. Use Helm to deploy MySQL on the cluster

1.  You should still be in Cloud Shell, connected to your Kubernetes cluster. Enter the following command to initialize Helm.

```
helm init
```

1.  To use Helm, a service account has to be created on the cluster that gives it administrative rights. Enter the following commands to add this service account with the required permissions.

```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

1.  Now enter the following command to update the Helm packages.

```
helm repo update 
```

1.  Now try installing MySQL using Helm.

```
helm install stable/mysql
```

1.  Read the output from the Helm install command and try connecting to your database using the instructions provided.

1.  Once you get connected to the database, exit to return to the Cloud Shell command prompt. 

1.  To see your Helm deployment enter the following command. 

```
helm ls
```

1.  Notice, the deployment has an auto-generated name like `intent-spaniel` or `smiling-penguin`. To delete the depoyment enter the following command ***but replace the deployment name shown with yours***.

```
helm delete intent-spaniel
```


<aside><p><strong>Congratulations!</strong> You have created a Kubernetes cluster and then deployed MySQL databass into it, first using your own configuration code, and then using Helm.</p></aside>


![[/fragments/endqwiklab]]


![[/fragments/copyright]]
