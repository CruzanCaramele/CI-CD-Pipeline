####  Setting up a local Image Registry

Instead of pushing/pulling images from the official docker registry, docker hub, we will set up a local repository as many teams do so for security purposes.

##### Steps

- From the root directory of the cloned repository, set up the cluster registry by creating the .yaml manifest files:

``` 
    kubectl create -f docker-registry/volume.yaml
    kubectl create -f docker-registry/registry-deploymeny.yaml
    kubectl create -f docker-registry/registry-services.yaml
```

The above will take sometime to finish creating the local image registry (several minutes):
```
   kubectl rollout status deployments/registry
```

