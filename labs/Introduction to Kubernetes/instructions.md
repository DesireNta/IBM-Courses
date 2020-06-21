# Introduction to Kubernetes

## Outcomes

You learned the basics of Docker including images and containers in the last lab. You also pushed an image to IBM Cloud Registry. In this lab, you will
- work with `kubectl` CLI
- create your first pod
- create your first deployment

## Prerequisites

The complete environment to finish this lab is provided for you at this link <tbd>. 

# Step 1: Ensure environment is installed
1. Open the terminal in the provided environment by using the menu in the editor `Terminal -> New Terminal`.
   
   ![terminal](images/terminal.png)

2. Make sure the environment has been configured by running the following commands in the terminal.
   - Check docker is installed
        ```
        theia@theiadocker-ulidder:/home/project$ docker --version
        >> Docker version 18.09.7, build 2d0083d
        ```
   - Check ibmcloud CLI is installed
        ```
        theia@theiadocker-ulidder:/home/project$ ibmcloud --version
        >> ibmcloud version 1.0.0+908f90a-2020-03-30T06:37:22+00:00

2. Make sure kubectl is installed by running the following commands in the terminal. You should get an output that looks something like this. The versions might be different.
   ```
   $ kubectl version
   
      Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.2", GitCommit:"59603c6e503c87169aea6106f57b9f242f64df89", G
      itTreeState:"clean", BuildDate:"2020-01-18T23:30:10Z", GoVersion:"go1.13.5", Compiler:"gc", Platform:"linux/amd64"}
      Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.10+IKS", GitCommit:"a0052bd119c067cf48e8a19f0ab7d5a5e2ca0a1
      8", GitTreeState:"clean", BuildDate:"2020-05-20T20:48:06Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
   ```

# Step 2: Work with kubectl CLI
The kubectl CLI is already installed for you in this environment. Not only that, the environment created a free cluster for your use with one worker node. Let's look at some basic kubectl commands.

1. Get cluster information using the `kubectl cluster-info` command
   ```
   $ kubectl config get-clusters
   
      NAME
      labs-user-sandbox-prod-tor01/bp1hokjw0e5bk715r7f0
   ```
2. You also learned about pods in this module. Let's list all the pods currently in our cluster. If this is a new session for you, you will not see any pods.
      ```
      $ kubectl get pods
      
         No resources found in sn-labs-upkaribmwats namespace.
      ```

# Step 3: Create a pod
1. Let's create our first pod. We will use the `nginx` image on Docker Hub to create this pod. As explained in the video for this module, you can create a pod imperatively or declaratively. 

## Using the `run` command to create a pod
1. Let's look at the imperative way first where we tell the cluster what to do using the `kubectl run` command.
   ```
      $ kubectl run nginx --image nginx

         kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.

         deployment.apps/nginx created
   ```
   You will see a warning, but the pod has been created for you. 
2. Let's see the actual pod using the `kubectl get <resource>` command. This command prints out the details of the cluster resource in a table format. There are many paratmers you can use with this command. Let's look at some simple use cases.
   ```
   $ kubectl get pods
      NAME                     READY   STATUS    RESTARTS   AGE
      nginx-6db489d4b7-2nzd8   1/1     Running   0          2m55s
   ```
   Great, the previous command indeed created a pod for us. It is called `nginx-6db489d4b7-2nzd8`. Your pod will most likely have a different name.  

   You can also specify the wide option for the output that gets you some more details about the resource. 

   ```
   $ kubectl get pods -o wide
      NAME                     READY   STATUS    RESTARTS   AGE     IP             NODE            NOMINATED NODE   READINESS GATES
      nginx-6db489d4b7-2nzd8   1/1     Running   0          9m12s   172.30.69.74   10.114.85.172   <none>           <none>
   ```
3. Let's delete this pod by using the `kubectl delete <resource> <name>` command.
   ```
   $ kubectl delete pod nginx-6db489d4b7-2nzd8
   
      pod "nginx-6db489d4b7-2nzd8" deleted
   ```
4. Let's get all the pods another time to see if it was actually deleted.
   ```
      $ kubectl get pods

         NAME                     READY   STATUS    RESTARTS   AGE
         nginx-6db489d4b7-rstbf   1/1     Running   0          77s
   ```
   Wait a second! We deleted the pod in the previous step, so why is it still there? The reason is that the `kubectl run` command actually created a `deployment` for us with a `replica` of 1. Therefore, the cluster automatically created another pod for us when we deleted the original one. You will see the pod name is different as well. 
5. We can use the `kubectl describe <resource> <name> command to see the details for this pod.
   ```
   $ kubectl describe pod nginx-6db489d4b7-rstbf

   Name:         nginx-6db489d4b7-rstbf
   Namespace:    sn-labs-upkaribmwats
   Priority:     0
   Node:         10.114.85.172/10.114.85.172
   Start Time:   Thu, 18 Jun 2020 23:12:25 +0000
   Labels:       pod-template-hash=6db489d4b7
               run=nginx
   Annotations:  kubernetes.io/limit-ranger:
                  LimitRanger plugin set: cpu, ephemeral-storage, memory request for container nginx; cpu, ephemeral-storage, memor
   y limit for container ngi...
               kubernetes.io/psp: ibm-privileged-psp
   Status:       Running
   IP:           172.30.69.74
   IPs:
   IP:           172.30.69.74
   Controlled By:  ReplicaSet/nginx-6db489d4b7
   Containers:
   nginx:
      Container ID:   containerd://778b66a518452a8eb7209ab42ae2f6af326a8f686f780199368764700c38d6e8
      Image:          nginx
      Image ID:       docker.io/library/nginx@sha256:21f32f6c08406306d822a0e6e8b7dc81f53f336570e852e25fbe1e3e3d0d0133
      Port:           <none>
      Host Port:      <none>
      State:          Running
         Started:      Thu, 18 Jun 2020 23:12:27 +0000
      Ready:          True
      Restart Count:  0
      Limits:
         cpu:                500m
         ephemeral-storage:  3Gi
         memory:             512Mi
      Requests:
         cpu:                200m
         ephemeral-storage:  512Mi
         memory:             128Mi
      Environment:          <none>
      Mounts:
         /var/run/secrets/kubernetes.io/serviceaccount from default-token-n6v4s (ro)
   ...
   ...
   ```
   As you can see, the pod has a replicaset associated wit it.
6. In order to delete all the resources we have created so far, we need to delete the deployment itself using the `kubectl delete <resource> <name>` command. First, let's get the name of the deployment.
   ```
      $ kubectl get deploy
         NAME    READY   UP-TO-DATE   AVAILABLE   AGE
         nginx   1/1     1            1           32m
   ```
   We can now delete the deployment resource as follows:
   ```
      $ kubectl delete deploy nginx
         deployment.apps "nginx" deleted  
   ```
7. If we list all the pods now, we will not see any.
   ```
      $ kubectl get pods
         No resources found in sn-labs-upkaribmwats namespace.
   ```
   Since we delete the deployment, the replicaset and the pods were deleted as well.

## Using the `create` command to create a pod
1. Another way to create the pod is by using the `create` command. This command creates a resource from a file in either JSON or YAML format. We have provided this file as `nginx-create.yaml` in the current directory. Notice that this command is also imperative. It is explicitly telling kubectl what to create. The file specifies the container name to be deployed in the pod.
   ```
      $ kubectl create -f nginx-create.yaml 
         pod/nginx created
   ```
2. You can again get the pod by using the get command
   ```
      $ kubectl get pods
         NAME    READY   STATUS    RESTARTS   AGE
         nginx   1/1     Running   0          5s
   ```
   In this case, kubectl does not create a deploy for us because the yaml file explicitly asked for a kind of `Pod`.
3. Let's delete the pod as before by first getting the name.
   ```
      $ kubectl get pods
         NAME    READY   STATUS    RESTARTS   AGE
         nginx   1/1     Running   0          63s
   ```
   and then using the 'kubectl delete <resource> <name>` command.
   ```
      $ kubectl delete pod nginx
         pod "nginx" deleted
   ```

## Using the `apply` command to declare a state
1. The previous two ways to create a pod were imperative. We explicitly told kubectl what to do. While the imperative commands are easy to understand and execute in our simple example, they are not ideal for a production environment. These commands do not provide an audit trails and therefore are hard to manage when working in a team. With a declarative approach, we can simply define a desired state of our cluster and let the underlying platform take the best actions needed to get to that state. Let's take a look at an example.
2. We have provided a sample nginx-apply.yaml file in this lab. Open that file in the editor. Notice the following:
   1. we would like to get a `Deployment`
   ```
      kind: Deployment
   ```
   2. we would like three pods
   ```
      replicas: 3
   ```
   3. we want the pods to be created with the nginx container
   ```
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
   ```
   You can ignore the rest for now. We will get to a lot of these concepts in the next lab.
3. Let's give this desired state to kubectl and let it go it's magic
   ```
   $ kubectl create -f nginx-apply.yaml
      deployment.apps/nginx created
   ```
4. Let's see our deploy using the `get` command.
   ```
      $ kubectl get deploy
         NAME    READY   UP-TO-DATE   AVAILABLE   AGE
         nginx   3/3     3            3           18s
   ```
5. Let's see how many pods were created
   ```
      $ kubectl get pods
         
         NAME                    READY   STATUS    RESTARTS   AGE
         nginx-dd6b5d745-8g4ck   1/1     Running   0          17s
         nginx-dd6b5d745-frks2   1/1     Running   0          17s
         nginx-dd6b5d745-zrtb6   1/1     Running   0          17s
   ```
We did not tell kubectl how to create the deployment. We simply gave it a desired state. If we were to delete a pod now, a new one will be created in place to keep the replicas at 3. Let's try it out.
6. Let's delete the pod ending in `8g4ck`
   ```
      $ kubectl delete pod nginx-dd6b5d745-8g4ck
         
         pod "nginx-dd6b5d745-8g4ck" deleted

   ```
7. If we run the `get pods` command immediately after, you will see a new container is being created. If you don't see this status, it is possible that the pod was already created for you.
   ```
   $ kubectl get pods
      
      NAME                    READY   STATUS              RESTARTS   AGE
      nginx-dd6b5d745-frks2   1/1     Running             0          3m7s
      nginx-dd6b5d745-xqmbf   0/1     ContainerCreating   0          11s
      nginx-dd6b5d745-zrtb6   1/1     Running 
   ```
8. Let's see the status again
   ```
      $ kubectl get pods

         NAME                    READY   STATUS    RESTARTS   AGE
         nginx-dd6b5d745-frks2   1/1     Running   0          3m50s
         nginx-dd6b5d745-xqmbf   1/1     Running   0          54s
         nginx-dd6b5d745-zrtb6   1/1     Running   0 
   ```
9.  One of the pod names should be different than what we had before.


This concludes the guided exercise. When done, close the browser tab.