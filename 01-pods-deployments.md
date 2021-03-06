# Pods and Deployments:

A **Pod** (*not container*) is the basic building-block/worker-unit in Kubernetes. Normally a pod is a part of a **Deployment**. 


## 1.1 Create a namespace

```
kubectl create namespace <name>
kubectl get pods -n <name>
```

Namespaces are the default way for kubernetes to separate resources. Namespaces do not share anything between them, which is important to know.

This is quite powerful. On a user level, it is also possible to limit namespaces and resources by users but it is a bit too involved for your first experience with Kubernetes. Therefore, please be aware that other people's namespaces are off limits for this workshop even if you do have access. ;)

Every time you run a kubectl command, you can opt to use the namespace flag (-n mynamespace) to execute the command on that namespace.

Similarly in the yaml files, you can specify namespace. Most errors you will get throughout the rest of the workshop will 99% be deploying into a namespace where someone already did that before you so ensure you are using your newly created namespace!

Kubernetes clusters come with a namespace called 'default', which should contain some pods (containers) deployed previously by Praqma.

The below commands do the same thing, because kubernetes commands will default to the default namespace:

```
kubectl get pods -n default
kubectl get pods
```

So let's try to use some of the Kubernetes objects, starting with a deployment.

## 1.2 Set your default namespace

You want to target your own namespace instead of default every time without the -n parameter, run:
```
kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
```

## 1.3 Creating pods using 'run' command:
We start by creating our first deployment. Normally people will run an nginx container/pod as first example o deployment. You can surely do that. But, we will run a different container image as our first exercise. The reason is that it will work as a multitool for testing and debugging throughout this course. Besides it too runs nginx! 


Here is the command to do it:

```
kubectl run multitool  --image=praqma/network-multitool
```

You should be able to see the following output:

```
$ kubectl run multitool --image=praqma/network-multitool
deployment "multitool" created
```

This command creates a deployment named multitool, starts a pod using this docker image (praqma/network-multitool), and makes that pod a member of that deployment. You don't need to confuse yourself with all these details at this stage. This is just extra (but vital) information. Just so you know what we are talking about, check the list of pods and deployments:

List of pods:
```
$ kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
multitool-3148954972-k8q06   1/1       Running   0          3m
```

List of deployments:
```
$ kubectl get deployments
NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
multitool   1         1         1            1           3m
```

There is actually also a replicaset, which is created as a result of the `run` command above, but that is not super important to know at this point. A ReplicaSet is something which deals with the number of copies of this pod. It will be covered in later exercise. It is shown below just for the sake of completeness.
```
$ kubectl get replicasets
NAME                   DESIRED   CURRENT   READY     AGE
multitool-3148954972   1         1         1         3m
```

Ok. The bottom line is that we wanted to have a pod running, and we have that. 


Lets setup another pod, a traditional nginx deployment, with a specific version - 1.7.9. 


Setup an nginx deployment with nginx:1.7.9
```
kubectl run nginx --image=nginx:1.7.9
```

You get another deployment and a replicaset as a result of above command, shown below, so you know what to expect:

```
$ kubectl get pods,deployments,replicasets
NAME                            READY     STATUS    RESTARTS   AGE
po/multitool-3148954972-k8q06   1/1       Running   0          25m
po/nginx-1480123054-xn5p8       1/1       Running   0          14s

NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/multitool   1         1         1            1           25m
deploy/nginx       1         1         1            1           14s

NAME                      DESIRED   CURRENT   READY     AGE
rs/multitool-3148954972   1         1         1         25m
rs/nginx-1480123054       1         1         1         14s
```

## 1.4 Deploying applications using declarative configuration files

You can also use the `nginx-simple-deployment.yaml` file to create the same nginx deployment. You can find the file in the `support-files` directory of this repo. However before you execute the command shown below, note that it will try to create a deployment with the name **nginx**. If you already have a deployment named **nginx** running, as done in the previous step, then you will need to delete that first.

Delete the existing deployment using the following command:
```
$ kubectl get deployments
NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
multitool   1         1         1            1           32m
nginx       1         1         1            1           7m

$ kubectl delete deployment nginx
deployment "nginx" deleted
```

Now you are ready to proceed with the example below:

```
$ kubectl create -f nginx-simple-deployment.yaml 
deployment "nginx" created
```


The contents of `nginx-simple-deployment.yaml` are as follows:
```
$ cat nginx-simple-deployment.yaml

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```


Verify that the deployment is created:
```
$ kubectl get deployments
NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
multitool   1         1         1            1           59m
nginx       1         1         1            1           36s
```


Check if the pods are running:
```
$ kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
multitool-3148954972-k8q06   1/1       Running   0          1h
nginx-431080787-9r0lx        1/1       Running   0          40s
```

## 1.5 Testing Kubernetes promise of resilience by deleting a pod

Before we move forward, lets see if we can delete a pod, and if it comes to life automatically:
```
$ kubectl delete pod nginx-431080787-9r0lx 
pod "nginx-431080787-9r0lx" deleted
```

As soon as we delete a pod, a new one is created, satisfying the desired state by the deployment, which is - it needs at least one pod running nginx. So we see that a **new** nginx pod is created (with a new ID):
```
$ kubectl get pods
NAME                         READY     STATUS              RESTARTS   AGE
multitool-3148954972-k8q06   1/1       Running             0          1h
nginx-431080787-tx5m7        0/1       ContainerCreating   0          5s

(after few more seconds)

$ kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
multitool-3148954972-k8q06   1/1       Running   0          1h
nginx-431080787-tx5m7        1/1       Running   0          12s
```
