# Ubuntu CI Machine

* ubuntu 18.04 with mirok8s installed on `cloudburo1`

## Post-Installation Steps

### Kubernetes Web UI


* Checking your admin setup 


    > kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')



* https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
 
* http://cloudburo1:8080/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login

### Enable Storage and Switch Default to another media

* Enable the Kubernetes Storage Feature, which will create by default a local default storage class

    > microk8.enable storage
    > microk8s.status
    
    ... storage: enabled
    
    > kubectl get storageclass
    NAME                          PROVISIONER            AGE
    microk8s-hostpath (default)   microk8s.io/hostpath   57m

* let's introduce a new default storage location on another media


    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: local-storage
    provisioner: kubernetes.io/no-provisioner
    volumeBindingMode: WaitForFirstConsumer


    > mkdir /media/disk1/microk8s-default-storageclass
    

    
    
    
    


* https://kubernetes.io/docs/concepts/storage/storage-classes/#local
