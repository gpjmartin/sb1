Follow the instructions as per the Docs: </br>

    https://docs.starburst.io/starburst-enterprise/admin-topics/autoscaling.html
  
Deploy the Auto-Scaler : </br>
  
    helm upgrade autoscaler autoscaler/cluster-autoscaler --install --version 9.21.0 --values auto-scaler.yaml 
  
Set the safe-to-evict : </br> 

    kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
  
Deploy the Metrics Server : </br>

    helm upgrade metricsserver metrics-server/metrics-server --install --version 3.8.2 --set apiService.create=true --values metrics-server.yaml 
  
Validate Metrics Server and Auto-Scaler are running : </br> 

    kubectl get deployment metricsserver-metrics-server  
  
    kubectl get deployment autoscaler-aws-cluster-autoscaler 
    

  





