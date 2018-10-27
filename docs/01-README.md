####  Setting up a local Image Registry

Instead of pushing/pulling images from the official docker registry, docker hub, we will set up a local repository as many teams do so for security purposes.

##### Steps

- From the root directory of the cloned repository, set up the cluster registry by creating the .yaml manifest files:

``` 
    kubectl create -f docker-registry/volume.yaml
    kubectl create -f docker-registry/registry-deployment.yaml
    kubectl create -f docker-registry/registry-services.yaml
```


- check resources created:

```
kube-ci-cd $ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                    STORAGECLASS   REASON    AGE
pvc-9a18371b-d98e-11e8-904a-0800277bacb9   2Gi        RWO            Delete           Bound       default/registry-claim   standard                 27s
registry                                   2Gi        RWO            Retain           Available                                                     27s
kube-ci-cd $ kubectl get pvc
NAME             STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
registry-claim   Bound     pvc-9a18371b-d98e-11e8-904a-0800277bacb9   2Gi        RWO            standard       31s

kube-ci-cd $ kubectl get deployments
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
registry   1         1         1            1           2m
kube-ci-cd $ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          1d
registry     NodePort    10.106.206.43   <none>        5000:30400/TCP   2m
registy-ui   NodePort    10.101.30.89    <none>        8080:31295/TCP   2m

```


The above will take sometime to finish creating the local image registry (several minutes), you can watch the status of the deployment by running the below command:
```
   kubectl rollout status deployments/registry
```


You can access the registry through the IP address of your minikube node and appending the registry-ui nodePort as below:

```
    minikube service registy-ui
```

#### Building the Docker Images

- In this step, we will build the docker image for the simple node application:

```
   kube-ci-cd $ docker build -t 127.0.0.1:30400/mysimple-app:latest -f applications/Dockerfile .
```

- AT this point, attempting to push the above image to the local image registry will fail because docker client can only push to  HTTP on localhost. Next, is to apply Docker container that listens on 127.0.0.1:30400 and forwards to our cluster:

```
   docker build -t socat-registry -f proxy-container/Dockerfile proxy-container/
```

- Check images built:
```
   kube-ci-cd $ docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
socat-registry                 latest              d4d7911c8e5b        8 minutes ago       9.99MB
127.0.0.1:30400/mysimple-app   latest              e450bafda250        18 minutes ago      537MB
alpine                         latest              196d12cf6ab1        6 weeks ago         4.41MB
risingstack/alpine
```


#### Push Images to Local Registry

- To push the simple node app image to the local registry, we need to start the proxy container first:
```
kube-ci-cd $ docker run -d -e "REG_IP=`minikube ip`" -e "REG_PORT=30400" --name socat-registry -p 30400:5000 socat-registry
f5ac890c6ca470dcab4c8b98013649b612e403fa599c9837a55069fc2e494091
```

- Next is to push the node app image:
```

