# Ubuntu Kubernetes Continous Integration Machine Setup

* ubuntu 18.04 with mirok8s installed on `cloudburo1`

## Post-Installation Steps

### Kubernetes Web UI


* Checking your admin setup 


    > kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')


* https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
* http://cloudburo1:8080/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login

### Enable Storage 

* Enable the Kubernetes Storage Feature, which will create a default storage class which is aimed for a 
  local disk (`hostpath`).
* By default this disk is part of the microk8s installation path. 

    > microk8.enable storage
    > microk8s.status
    
    ... storage: enabled
    
    > kubectl get storageclass
    NAME                          PROVISIONER            AGE
    microk8s-hostpath (default)   microk8s.io/hostpath   57m


* A`StorageClass` provides a way for administrators to describe the “classes” of storage they offer. 
* Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary 
  policies determined by the cluster administrators. 
* Kubernetes itself is unopinionated about what classes represent. 
* This concept is sometimes called “profiles” in other storage systems.


### Create a new  Physical Volume on Local Host

* We create a new Kubernetes Physical Volume, which is poiting to antoher disk attached `/media/disk1`
* Persistent Volume (PV) in Kubernetes represents a real piece of underlying storage capacity in the 
  infrastructure. 
* Before using Kubernetes to mount anything, you must first create whatever storage that you plan to mount
* Persistent volumes are intended for "network volumes" like GCE Persistent Disks, NFS shares, and 
  AWS ElasticBlockStore volumes. 
* `hostPath` was included for ease of development and testing but not really suitable for production installastion
[Link](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/getting_started_with_kubernetes/get_started_provisioning_storage_in_kubernetes#kubernetes_persistent_volumes)
                                                           

* Create a local host directory for the new physical volume (pv)


    > mkdir /media/disk1/microk8s-default-storageclass

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


     > kubectl create -f ~/kubernetes/physicalVolume1.yaml
     persistentvolume/pv0001 created
     
     > kubectl get pv
     NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
     pv0001   50Gi       RWO            Retain           Available                                   107s
     
     > kubectl describe pv
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


### Enable the Docker Registry

* We now switched the default storage class to our new media and will enable the local Docker registry
* Which will create its repository using the default storage class for storage claims which maps to our physical volume


    > microk8s.enable registry
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


    > kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                               STORAGECLASS        REASON   AGE
    pv0001                                     50Gi       RWO            Retain           Available                                       microk8s-hostpath            117m
    pvc-f4cb680b-165c-11e9-ad63-10bf48e10813   20Gi       RWX            Delete           Bound       container-registry/registry-claim   microk8s-hostpath            2m30s


* Pods access storage by using the claim as a volume. Claims must exist in the same namespace as the pod using the claim. 
* The cluster finds the claim in the pod’s namespace and uses it to get the PersistentVolume backing the claim. 
* The volume is then mounted to the host and into the pod.






    
    

    
    
    

