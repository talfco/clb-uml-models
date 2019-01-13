# Ubuntu Kubernetes CI Machine Setup Based on mircok8s

* ubuntu 18.04 with mirok8s installed on `cloudburo1`
* Convention `kb alias kubectl`


## Kubernetes Web UI


* Checking your admin setup 


      $ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')


* https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
* http://cloudburo1:8080/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login

## Cluster Config Overview

* A **cluster** is made up of 
    * **name-spaces**: `ns` which consists of 
        * **deployments**: `deploy` which is made up of
            * **replica-sets**: `ns` which controls the 
                * **pods** executing a docker image.
  


      $ kb get ns
    
        NAME                 STATUS   AGE
        container-registry   Active   6h28m
        default              Active   18d
        kube-public          Active   18d
        kube-system          Active   18d
    
      $ kb get --namespace=kube-system deploy
    
        NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
        heapster-v1.5.2                  1/1     1            1           2d
        hostpath-provisioner             1/1     1            1           34h
        kubernetes-dashboard             1/1     1            1           2d
        monitoring-influxdb-grafana-v4   1/1     1            1           2d
  
      $ kb get --namespace=container-registry deploy
    
        NAME       READY   UP-TO-DATE   AVAILABLE   AGE
        registry   1/1     1            1           6h34m
    
      $ kb get --namespace=container-registry rs
    
        NAME                  DESIRED   CURRENT   READY   AGE
        registry-7fc4594d64   1         1         1       6h41m

      $ kb get --namespace=container-registry pod
        NAME                        READY   STATUS    RESTARTS   AGE
        registry-7fc4594d64-5gg2d   1/1     Running   0          6h49m

      $ kb describe --namespace=container-registry  pod
    
        Name:               registry-7fc4594d64-5gg2d
        Namespace:          container-registry
        Priority:           0
        PriorityClassName:  <none>
        Node:               cloudburo1/192.168.1.125
        Start Time:         Sat, 12 Jan 2019 11:26:51 +0000
        Labels:             app=registry
                            pod-template-hash=7fc4594d64
        Annotations:        <none>
        Status:             Running
        IP:                 10.1.1.23
        Controlled By:      ReplicaSet/registry-7fc4594d64
        Containers:
          registry:
            Container ID:   docker://8da63018db3bba680c78376caed3b71d45b972b2f976ce9a02c69f94ecb74c81
            Image:          cdkbot/registry-amd64:2.6
            Image ID:       docker-pullable://cdkbot/registry-amd64@sha256:9dc5ad9e0b3ed59afa4febe8d663fd545628c60886a2c9fcd0e186e5cf6343ef
            Port:           5000/TCP
            Host Port:      0/TCP
            State:          Running
              Started:      Sat, 12 Jan 2019 11:26:52 +0000
            Ready:          True
            Restart Count:  0
            Environment:
              REGISTRY_HTTP_ADDR:                         :5000
              REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY:  /var/lib/registry
            Mounts:
              /var/lib/registry from registry-data (rw)
              /var/run/secrets/kubernetes.io/serviceaccount from default-token-pxlcl (ro)
        Conditions:
          Type              Status
          Initialized       True 
          Ready             True 
          ContainersReady   True 
          PodScheduled      True 
        Volumes:
          registry-data:
            Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
            ClaimName:  registry-claim
            ReadOnly:   false
          default-token-pxlcl:
            Type:        Secret (a volume populated by a Secret)
            SecretName:  default-token-pxlcl
            Optional:    false
        QoS Class:       BestEffort
        Node-Selectors:  <none>
        Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                         node.kubernetes.io/unreachable:NoExecute for 300s
        Events:
          Type     Reason             Age                   From                 Message
          ----     ------             ----                  ----                 -------
          Warning  MissingClusterDNS  3s (x329 over 6h51m)  kubelet, cloudburo1  pod: "registry-7fc4594d64-5gg2d_container-registry(f4d04d09-165c-11e9-ad63-10bf48e10813)". kubelet does not have ClusterDNS IP configured and cannot create Pod using "ClusterFirst" policy. Falling back to "Default" policy.

* `registry-data` of the POD is of type `PersistentVolumeClaim` with the `ClaimName: registry-claim`


    $ kb describe --namespace=container-registry  pvc
    
    Name:          registry-claim
    Namespace:     container-registry
    StorageClass:  microk8s-hostpath
    Status:        Bound
    Volume:        pvc-f4cb680b-165c-11e9-ad63-10bf48e10813
    Labels:        <none>
    Annotations:   control-plane.alpha.kubernetes.io/leader:
                     {"holderIdentity":"b502cd6b-1646-11e9-bf54-3eb4d231b7f7","leaseDurationSeconds":15,"acquireTime":"2019-01-12T11:26:51Z","renewTime":"2019-...
                   kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"registry-claim","namespace":"container-registry"},"...
                   pv.kubernetes.io/bind-completed: yes
                   pv.kubernetes.io/bound-by-controller: yes
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      20Gi
    Access Modes:  RWX
    VolumeMode:    Filesystem
    Events:        <none>
    Mounted By:    registry-7fc4594d64-5gg2d


## Enable Storage 

* Enable the Kubernetes Storage Feature, which will create a default storage class which is aimed for a 
  local disk (`hostpath`).
* By default this disk is part of the microk8s installation path. 


    $ microk8.enable storage
    $ microk8s.status
    
    ... storage: enabled
    
    $ kubectl get storageclass
    
    NAME                          PROVISIONER            AGE
    microk8s-hostpath (default)   microk8s.io/hostpath   57m


* A`StorageClass` provides a way for administrators to describe the “classes” of storage they offer. 
* Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary 
  policies determined by the cluster administrators. 
* Kubernetes itself is unopinionated about what classes represent. 
* This concept is sometimes called “profiles” in other storage systems.


## Create a new  Physical Volume on Local Host

* We create a new Kubernetes Physical Volume, which is poiting to antoher disk attached `/media/disk1`
* Persistent Volume (PV) in Kubernetes represents a real piece of underlying storage capacity in the 
  infrastructure. 
* Before using Kubernetes to mount anything, you must first create whatever storage that you plan to mount
* Persistent volumes are intended for "network volumes" like GCE Persistent Disks, NFS shares, and 
  AWS ElasticBlockStore volumes. 
* `hostPath` was included for ease of development and testing but not really suitable for production installastion
[Link](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/getting_started_with_kubernetes/get_started_provisioning_storage_in_kubernetes#kubernetes_persistent_volumes)
                                                           

* Create a local host directory for the new physical volume (pv)


    $ mkdir /media/disk1/microk8s-default-storageclass

* Create a persistentVolume.yaml. We will associate the default storage-class of microk8s to this new media: `storageClassName: microk8s-hostpath`

    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: pv0001
      labels:
        type: local
    spec:
      capacity:
        storage: 50Gi
      accessModes:
        - ReadWriteOnce
      storageClassName: microk8s-hostpath
      hostPath:
        path: "/media/disk1/microk8s-pv1"
        

* Run the `kubectl create` command, btw. in case you want to update a yaml confiugration run `kubectl apply`  


     $ kubectl create -f ~/kubernetes/physicalVolume1.yaml
     
     persistentvolume/pv0001 created
     
     $ kubectl get pv
     
     NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
     pv0001   50Gi       RWO            Retain           Available                                   107s
     
     $ kubectl describe pv
     
       Name:            pv0001
       Labels:          type=local
       Annotations:     kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"v1","kind":"PersistentVolume","metadata":{"annotations":{},"labels":{"type":"local"},"name":"pv0001"},"spec":{"accessModes"...
       Finalizers:      [kubernetes.io/pv-protection]
       StorageClass:    microk8s-hostpath
       Status:          Available
       Claim:           
       Reclaim Policy:  Retain
       Access Modes:    RWO
       VolumeMode:      Filesystem
       Capacity:        50Gi
       Node Affinity:   <none>
       Message:         
       Source:
           Type:          HostPath (bare host directory volume)
           Path:          /media/disk1/microk8s-pv1
           HostPathType:  
       Events:            <none>


## Enable the Docker Registry

* We now switched the default storage class to our new media and will enable the local Docker registry
* Which will create its repository using the default storage class for storage claims which maps to our physical volume


    $ microk8s.enable registry
    
    Enabling the private registry
    Enabling default storage class
    deployment.extensions/hostpath-provisioner unchanged
    storageclass.storage.k8s.io/microk8s-hostpath unchanged
    Storage will be available soon
    Applying registry manifest
    namespace/container-registry created
    persistentvolumeclaim/registry-claim created
    deployment.extensions/registry created
    service/registry created
    The registry is enabled


* The registry claims some disk space for it's registry `persistentvolumeclaim/registry-claim`.


    $ kubectl get pv
    
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                               STORAGECLASS        REASON   AGE
    pv0001                                     50Gi       RWO            Retain           Available                                       microk8s-hostpath            117m
    pvc-f4cb680b-165c-11e9-ad63-10bf48e10813   20Gi       RWX            Delete           Bound       container-registry/registry-claim   microk8s-hostpath            2m30s


* Pods access storage by using the claim as a volume. Claims must exist in the same namespace as the pod using the claim. 
* The cluster finds the claim in the pod’s namespace and uses it to get the PersistentVolume backing the claim. 
* The volume is then mounted to the host and into the pod.








    
    

    
    
    

