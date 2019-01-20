



# Practical Container Orchestration 
---
# Helm lab
---

![helmlogo2](images/helmlogo2.png)

---

## Table of Contents

- [Task 1: Helm Setup](#task-1--helm-setup)
  * [1. Connect to the Kubernetes Cluster](#1-connect-to-the-kubernetes-cluster)
  * [2. What is the helm tool](#2-what-is-the-helm-tool)
  * [3. Download the Helm client](#3-download-the-helm-client)
    + [MacOS](#macos)
    + [Windows](#windows)
  * [4. Configure a RBAC role](#4-configure-a-rbac-role)
  * [5. Initialize Helm & Tiller](#5-initialize-helm---tiller)
  * [6. Add a helm repo](#6-add-a-helm-repo)
  * [7. Access to your private registry](#7-access-to-your-private-registry)
- [Task 2: Installing a simple application](#task-2--installing-a-simple-application)
  * [1. Getting a new helm repo](#1-getting-a-new-helm-repo)
  * [2. View the available packages](#2-view-the-available-packages)
  * [3. Install a package](#3-install-a-package)
  * [4. List the package](#4-list-the-package)
  * [5. Delete the package](#5-delete-the-package)
- [Task 3: Understand the Kubernetes manifests](#task-3--understand-the-kubernetes-manifests)
  * [1. Build a new docker image](#1-build-a-new-docker-image)
  * [2. View a kubernetes manifest](#2-view-a-kubernetes-manifest)
- [Task 4: Define a Helm chart](#task-4--define-a-helm-chart)
  * [1. Initialize an empty chart directory](#1-initialize-an-empty-chart-directory)
  * [2. Look at the chart directory content.](#2-look-at-the-chart-directory-content)
  * [3. Check the chart](#3-check-the-chart)
- [Task 5: Using Helm](#task-5--using-helm)
  * [1. Create a new namespace](#1-create-a-new-namespace)
  * [2. Install the chart to the training namespace](#2-install-the-chart-to-the-training-namespace)
  * [3. List the releases](#3-list-the-releases)
  * [4. List the deployments](#4-list-the-deployments)
  * [5. List the services](#5-list-the-services)
  * [6. List the pods](#6-list-the-pods)
  * [7. Upgrade](#7-upgrade)
- [Congratulations](#congratulations)

# Helm Lab

Helm helps you manage Kubernetes applications — Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application.

During this lab, we are going to install a helm client and configure it. Then we are going to install several sample charts to see how it works.

Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste madness.


# Task 1: Helm Setup


## 1. Connect to the Kubernetes Cluster

Be sure that you are connected to **mycluster**. 

Use the kubectl command to check that you are connected:

`kubectl get nodes`


If you get an error, go back to the **PrepareLab** and check the instructions to **gain access** to the cluster.


## 2. What is the helm tool

Helm is a client/server application : **Helm** client and **Tiller** server. 
Before we can run any chart with helm, we should proceed to some installation and configuration. 

## 3. Download the Helm client


### MacOS
```console
https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-darwin-amd64.tar.gz
```

`tar -zxvf helm-v2.9.1-darwin-amd64.tar.gz`

```
chmod +x helm

mv ./helm /usr/local/bin/helm
```


### Windows
```console 
https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-windows-amd64.zip
```
Unzip that file.

Move the executable file **helm.exe** to a directory that is available in your path like "C:\Program Files\IBM\Cloud\bin\"

## 4. Configure a RBAC role

> IMPORTANT : To maintain cluster security, create a service account for Tiller in the kube-system namespace and a Kubernetes RBAC cluster role binding for the tiller-deploy pod.

In your preferred editor, create the following file and save it as :

`rbac-config.yaml`

The cluster-admin cluster role is created by default in Kubernetes clusters, so you don’t need define it explicitly.   

```console
apiVersion: v1
kind: ServiceAccount
metadata:
 name: tiller
 namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: tiller
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: ClusterRole
 name: cluster-admin
subjects:
 - kind: ServiceAccount
   name: tiller
   namespace: kube-system
```

Create the service account and cluster role binding.

`kubectl create -f rbac-config.yaml`

Results:

```console
$ kubectl create -f rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding "tiller" created
```

## 5. Initialize Helm & Tiller

Initialize Helm and install tiller with the service account that you created.

> IMPORTANT : this operation should done once.

`helm init --service-account tiller`

Results:

```console 
$ helm init --service-account tiller
$HELM_HOME has been configured at /Users/phil/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```
Verify that the tiller-deploy pod has a Status of Running in your cluster.

`kubectl get pods -n kube-system -l app=helm`

Results:

```console
$ kubectl get pods -n kube-system -l app=helm
NAME                           READY     STATUS    RESTARTS   AGE
tiller-deploy-75f5797b-sxmrq   1/1       Running   0          4m
```


## 6. Add a helm repo

Add the IBM Cloud Helm repository to your Helm instance.

`helm repo add ibm  https://registry.bluemix.net/helm/ibm` 

Results:

```console
$ helm repo add ibm  https://registry.bluemix.net/helm/ibm
"ibm" has been added to your repositories
```

List the Helm charts that are currently available in the IBM Cloud repository.

`helm search ibm`

Check the helm version:

`helm version --short`

Results:
```console
$ helm version --short
Client: v2.9.1+g20adb27
Server: v2.9.1+g20adb27
```

> **Important**: helm client and tiller server should have the same version.


## 7. Access to your private registry

For the next exercise, we need to get access to  your private registry. To do so,  login to the private registry:

`ibmcloud cr login`

Results:

```console
$ ibmcloud cr login 
Logging in to 'registry.eu-gb.bluemix.net'...
Logged in to 'registry.eu-gb.bluemix.net'.

OK
```


# Task 2: Installing a simple application


## 1. Getting a new helm repo

Add a Helm repository. To add the Kubernetes Incubator repository, run the following command:

```console
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
```

## 2. View the available packages

`helm search -l`

You should see a very long list of packages.

## 3. Install a package

In this command, package_name is the name for the package, and package_in_repo is the name of the available package to install. For example, to install the WordPress package, run the following command:

`helm install --name=my-wordpress stable/wordpress`

Results:

```console
$ helm install --name=my-wordpress stable/wordpress      
NAME:   my-wordpress
LAST DEPLOYED: Mon May 21 00:52:04 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME                    TYPE    DATA  AGE
my-wordpress-mariadb    Opaque  2     1s
my-wordpress-wordpress  Opaque  2     1s

==> v1/ConfigMap
NAME                  DATA  AGE
my-wordpress-mariadb  1     1s

==> v1/PersistentVolumeClaim
NAME                    STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
my-wordpress-mariadb    Pending  1s
my-wordpress-wordpress  Pending  1s

==> v1/Service
NAME                    TYPE          CLUSTER-IP      EXTERNAL-IP  PORT(S)                     AGE
my-wordpress-mariadb    ClusterIP     172.21.109.195  <none>       3306/TCP                    1s
my-wordpress-wordpress  LoadBalancer  172.21.229.183  <pending>    80:32743/TCP,443:32275/TCP  1s

==> v1beta1/Deployment
NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
my-wordpress-mariadb    1        1        1           0          1s
my-wordpress-wordpress  1        1        1           0          1s

==> v1/Pod(related)
NAME                                     READY  STATUS   RESTARTS  AGE
my-wordpress-mariadb-696c878dd-9mqzt     0/1    Pending  0         1s
my-wordpress-wordpress-6b57cb6c85-m27m6  0/1    Pending  0         1s

```


## 4. List the package

`helm list`

```
 NAME                REVISION    UPDATED                     STATUS      CHART              NAMESPACE
 my-wordpress        1           Wed Jun 28 22:15:13 2017    DEPLOYED    wordpress-0.6.5    default
```

 You can also look at the Kubernetes Dashboard to see if the deployment was successful (DEPLOYED).
 Even if the deployment was successful, you still have to check that your application is running fine (in that case, a volume claim is requested and the application is not running fine).

## 5. Delete the package

`helm delete my-wordpress --purge`

```
$ helm delete my-wordpress --purge
release "my-wordpress" deleted
```

Generally an helm chart is managing many pods, deployments, secrets, volumes and services. So deleting the chart is a quick way to clean up your work.

# Task 3: Understand the Kubernetes manifests



## 1. Build a new docker image

Move to the Lab 2 directory: 

`cd "container-service-getting-started-wt/Lab 2"`

Then build the container:		

`docker build -t registry.eu-gb.bluemix.net/<namespace>/
hello-world:2 .`

`docker images registry.eu-gb.bluemix.net/<namespace>/hello-world:2`

Result:

```console
$ docker images registry.eu-gb.bluemix.net/imgreg/hello-world:2
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
registry.eu-gb.bluemix.net/imgreg/hello-world   2                   e8fea84576b2        5 weeks ago         74.1MB

```

## 2. View a kubernetes manifest

Open the **healthcheck.yml** file

`nano healthcheck.yml`
or 
`notepad healthcheck.yml`

Inspect the **healthcheck.yml** file:

- Indicates that the symbol `---` indicates a delimiter for a new resource
- Note that there are 2 resources being deployed here, a deployment and a service.
- The metadata section defines the attributes of the resource (name, annotation, label etc), while the spec specifies the detail of the resource

Review the deployment's manifest

```console
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: hw-demo-deployment
spec:
  replicas: 3
  template:
    metadata:
      name: pod-liveness-http
      labels:
        run: hw-demo-health
        test: hello-world-demo
    spec:
      containers:
        - name: hw-demo-container
          image: "registry.ng.bluemix.net/<namespace>/hello-world:2"
          imagePullPolicy: Always
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```

Inspect that file :

- Look into the deployment and see that the spec contains a ReplicaSet (see the `replicas` term) and that the ReplicaSet includes a Pod
- The Pod is defined as a template that contains the similar structure of metadata and specification
- The specification of the pod includes an array of containers that refers to an image.
- The livenessProbe and readinessProbe are checks that the kubernetes system would perform to check pod's health.

Review the **service's manifest**

```console
apiVersion: v1
kind: Service
metadata:
  name: hw-demo-service
  labels:
    run: hw-demo-health
spec:
  type: NodePort
  selector:
    run: hw-demo-health
  ports:
   - protocol: TCP
     port: 8080
     nodePort: 30072
```

- The services defines the accessibility of a pod. This service is of type NodePort, which exposes an internal Port (8080) into an externally accessible nodePort through the proxy node (here port 30072)
- How does a service know which pod are associated with it?  From the selector(s) that would select all pods with the same label(s) to be load balanced.

# Task 4: Define a Helm chart

Now that you have understood the structure of a kubernetes manifest file, you can start working with helm chart. Basically a helm chart is a collection of manifest files that are deployed as a group. The deployment includes the ability to perform variable substitution based on a configuration file.

## 1. Initialize an empty chart directory

`cd`

`helm create hellonginx`
​        

## 2. Look at the chart directory content.

`cd hellonginx`

Inspect the directory tree:

![create tree](./images/treehelm.png)


`nano values.yaml` or `notepad values.yaml`

Look at **values.yaml** and **modify it**. Prepare to deploy **1** replicas of the nginx image. Replace the service section and choose a port (like 30073 for instance) with the following code:

```console
  name: hellonginx-service
  type: NodePort
  externalPort: 80  
  internalPort: 80  
  nodePort: 30073
```





The main content for **values.yaml** is as follows:

```console
# Default values for hellonginx.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent

service:
  name: hellonginx-service
  type: NodePort
  externalPort: 80  
  internalPort: 80  
  nodePort: 30073

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  path: /
  hosts:
    - chart-example.local
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #  cpu: 100m
  #  memory: 128Mi
  # requests:
  #  cpu: 100m
  #  memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}
```

Review **deployment template** : 

`nano ./templates/deployment.yaml` or  `notepad ./templates/deployment.yaml`

Don't change anything.

```console
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ template "hellonginx.fullname" . }}
  labels:
    app: {{ template "hellonginx.name" . }}
    chart: {{ template "hellonginx.chart" . }}
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ template "hellonginx.name" . }}
      release: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ template "hellonginx.name" . }}
        release: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
{{ toYaml .Values.resources | indent 12 }}
    {{- with .Values.nodeSelector }}
      nodeSelector:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.affinity }}
      affinity:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.tolerations }}
      tolerations:
{{ toYaml . | indent 8 }}
    {{- end }}
```





Review the **service template**: 

`nano ./templates/service.yaml` or  `notepad ./templates/service.yaml`

Change the **-port section** with the following code:

```console
    - port: {{ .Values.service.externalPort }}
      targetPort: {{ .Values.service.internalPort }}
      protocol: TCP
      nodePort: {{ .Values.service.nodePort }}
      name: {{ .Values.service.name }}
```

So the service should look as follows:

```console
apiVersion: v1
kind: Service
metadata:
  name: {{ template "hellonginx.fullname" . }}
  labels:
    app: {{ template "hellonginx.name" . }}
    chart: {{ template "hellonginx.chart" . }}
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.externalPort }}
      targetPort: {{ .Values.service.internalPort }}
      protocol: TCP
      nodePort: {{ .Values.service.nodePort }}
      name: {{ .Values.service.name }}
  selector:
    app: {{ template "hellonginx.name" . }}
    release: {{ .Release.Name }}
```
## 3. Check the chart

Go back to the hellonginx path and check the validity of the helm chart.

`helm lint`

![modify chart](images/lint.png)

# Task 5: Using Helm

The helm chart that we created in the previous section that has been verified can now be deployed.

## 1. Create a new namespace

We are now going to define a new namespace in the mycluster.
This namespace can be seen as a **virtual partition** in the cluster. 

`kubectl create namespace training`

Results :

```console
$ kubectl create namespace training
namespace "training" created
```


## 2. Install the chart to the training namespace

Type the following command and don't forget the dot at the end.
First check you are still in the **hellonginx** directory.

`helm install --name hellonginx --namespace training .`

Results:
```console
$ helm install --name hellonginx --namespace training .
NAME:   hellonginx
LAST DEPLOYED: Sun Jan 20 23:12:11 2019
NAMESPACE: training
STATUS: DEPLOYED

RESOURCES:
==> v1beta2/Deployment
NAME        DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
hellonginx  1        1        1           0          1s

==> v1/Pod(related)
NAME                         READY  STATUS             RESTARTS  AGE
hellonginx-76d95f9f78-px552  0/1    ContainerCreating  0         1s

==> v1/Service
NAME        TYPE      CLUSTER-IP      EXTERNAL-IP  PORT(S)       AGE
hellonginx  NodePort  172.21.212.184  <none>       80:30073/TCP  1s


NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace training -o jsonpath="{.spec.ports[0].nodePort}" services hellonginx)
  export NODE_IP=$(kubectl get nodes --namespace training -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT

```
At the end of the output of that command, you will see 3 lines that you can normally use to get the URL to access your application.

We are going to use another method :

`ibmcloud cs workers mycluster`

Results :

```console
$ ic cs workers mycluster
OK

ID                                                 Public IP         Private IP      Machine Type   State    Status   Zone    Version   
kube-mil01-paad8927a922ea4f5686b36896357a889d-w1   159.122.181.117   10.144.187.96   free           normal   Ready    mil01   1.9.7_1512  
```
So combine the public IP of your worker node (159.122.181.117) and the port 30073 (defined in the service) to get the URL:

`http://159.122.181.117:30073`

Try this url and get the nginx hello:

![Welcome Nginx](./images/nginx.png)


## 3. List the releases 

`helm list`

Results:

```console
$ helm list

NAME      	REVISION	UPDATED                 	STATUS  	CHART           	NAMESPACE
hellonginx	1       	Tue May 22 10:46:32 2018	DEPLOYED	hellonginx-0.1.0	training 
```


## 4. List the deployments

`kubectl get deployments --namespace=training`

```console
$ kubectl get deployments --namespace=training
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hellonginx   1         1         1            1           3m
```

## 5. List the services

`kubectl get services --namespace=training`

**Results**
```console
$ kubectl get services --namespace=training

NAME         TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
hellonginx   NodePort   172.21.132.143   <none>        80:30073/TCP   23m
```

Locate the line port 80:300073.  

## 6. List the pods

`kubectl get pods --namespace=training`

**Results**
```console
$ kubectl get pods --namespace=training
NAME                          READY     STATUS    RESTARTS   AGE
hellonginx-76d95f9f78-px552   1/1       Running   0          4m
```

## 7. Upgrade 

We now want to change the number of replicas to 5:

Open the **values.yaml**

Change the **replicaCount: 1** to **replicaCount: 5**

Save the file and type the following command :

`helm  upgrade hellonginx .`

**Results**

```console
$ helm  upgrade hellonginx .
Release "hellonginx" has been upgraded. Happy Helming!
LAST DEPLOYED: Tue May 22 11:13:43 2018
NAMESPACE: training
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME        TYPE      CLUSTER-IP      EXTERNAL-IP  PORT(S)       AGE
hellonginx  NodePort  172.21.132.143  <none>       80:30073/TCP  27m

==> v1beta2/Deployment
NAME        DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
hellonginx  5        5        5           3          27m

==> v1/Pod(related)
NAME                         READY  STATUS             RESTARTS  AGE
hellonginx-6bcd9f4578-6rckz  0/1    ContainerCreating  0         1s
hellonginx-6bcd9f4578-7ps7l  1/1    Running            0         27m
hellonginx-6bcd9f4578-fg4dt  0/1    ContainerCreating  0         1s
hellonginx-6bcd9f4578-mzmtm  1/1    Running            0         27m
hellonginx-6bcd9f4578-vnx5g  1/1    Running            0         27m


```


# Congratulations

You successfully created and managed charts to deploy applications on the IBM Cloud. 

---
# End of the lab
---
# Practical Container Orchestration 
---