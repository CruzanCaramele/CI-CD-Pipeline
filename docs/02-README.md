####  Part 2 - Running Jenkins in a Pod

In this part, we’ll set up continuous delivery our simple application by running Jenkins in a pod using Kubernetes. A pipeline will be created using a Jenkins 2.0 Pipeline script that automates building our simple app image, pushing it to the registry, and deploying it in Kubernetes.


#### Buildin a Jenkins Pipeline

- Build the Jenkins image to use in the Kubernetes cluster:
```
   docker build -t 127.0.0.1:30400/jenkins:latest -f applications/jenkins/Dockerfile applications/jenkins/
```

- Push Jenkins image to local registry:
```
   docker push 127.0.0.1:30400/jenkins:latest
```

- Deploy Jenkins, which we’ll use to create our automated CI/CD pipeline. It will take the pod a minute or two to roll out:
```
   kubectl create -f applications/jenkins/jenkins.yaml
```
--> open jenkins in browser:
```
   minikube service jenkins
```
--> Display the Jenkins admin password with the following command, and right-click to copy it:
```
   kubectl exec -it `kubectl get pods --selector=app=jenkins 

--output=jsonpath={.items..metadata.name}` cat 

/var/jenkins_home/secrets/initialAdminPassword
```

- Go back to the Jenkins UI. Paste the Jenkins admin password in the box and click Continue. Click Install suggested plugins. Plugins have actually been pre-downloaded during the Jenkins image build.
- Continue by creating an admin username and password, save and finish
