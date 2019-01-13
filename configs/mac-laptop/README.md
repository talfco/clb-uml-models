# Mac Developer Machine Setup

## Docker

* Docker Desktop manages Docker and Kubernetes
* https://blog.docker.com/2018/07/kubernetes-is-now-available-in-docker-desktop-stable-channel/


      $ git clone https://github.com/dockersamples/k8s-wordsmith-demo.git
      $ cd k8s-wordsmith-demo
      $ docker-compose build  
       
         ...
         Successfully tagged dockersamples/k8s-wordsmith-web:latest
     
      $ kubectl apply -f kube-deployment.yml
     
         service "db" created
         deployment.apps "db" created
         service "words" created
         deployment.apps "words" created
         service "web" created
         deployment.apps "web" created
     
      $ kubectl get pods
     
         NAME                     READY     STATUS    RESTARTS   AGE
         db-6d46b7775-dn8mn       1/1       Running   0          4m
         web-d4f4cf785-vvxfk      1/1       Running   0          4m
         words-55c744cc9d-b4qgr   1/1       Running   0          4m
         words-55c744cc9d-jc66m   1/1       Running   0          4m
         words-55c744cc9d-jznzx   1/1       Running   0          4m
         words-55c744cc9d-pqnsf   1/1       Running   0          4m
         words-55c744cc9d-r5q8q   1/1       Running   0          4m
     
      $ kubectl get svc
     
         NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
         db           ClusterIP      None             <none>        5432/TCP         5m
         kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP          54m
         web          LoadBalancer   10.105.163.121   localhost     8081:30145/TCP   5m
         words        ClusterIP      None             <none>        8080/TCP         5m

* So Kubernetes run in the same container as docker and it is ensured that the 
  docker stays update. Often kubernetes installation comes with outdated
  docker binaries
* You benefit that you can mix it with docker-compose, i.e. you can use the 
   `docker-compose.yml` as a deployment configuration to the kubernetes
   stack by running `docker stack deploy -c ./docker-compose.yml words`
   
   
      $ kubectl get ns
     
       NAME          STATUS    AGE
       default       Active    1h
       docker        Active    1h
       kube-public   Active    1h
       kube-system   Active    1h

      $ kubectl get -n default  deploy
    
        NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
        db        1         1         1            1           34m
        web       1         1         1            1           34m
        words     5         5         5            5           34m
        
      $  kubectl delete deployment db -n default
      $  kubectl delete deployment web -n default
      $  kubectl delete deployment words -n default
