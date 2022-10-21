Follow the instructions as per the Docs: </br>

    https://docs.starburst.io/starburst-enterprise/admin-topics/autoscaling.html
  
Customize the yaml to match the SEP Cluster : </br>

    autoDiscovery:
        clusterName: grm-lab01
        tags:
          - k8s.io/cluster-autoscaler/enabled
          - k8s.io/cluster-autoscaler/your_cluster_name_here

    extraArgs:
      skip-nodes-with-local-storage: "false"
      skip-nodes-with-system-pods: "false"
      balance-similar-node-groups: "true"
      expander: "least-waste"

    priorityClassName: "node-autoscaler"

Note: The priorityClassName needs to match what is defined in the Priority yaml file later. </br> 

Deploy the Auto-Scaler : </br>
  
    helm upgrade autoscaler autoscaler/cluster-autoscaler --install --version 9.21.0 --values auto-scaler.yaml 
  
Set the safe-to-evict : </br> 

    kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
  
Deploy the Metrics Server : </br>

    helm upgrade metricsserver metrics-server/metrics-server --install --version 3.8.2 --set apiService.create=true --values metrics-server.yaml 
  
Validate Metrics Server and Auto-Scaler are running : </br> 

    kubectl get deployment metricsserver-metrics-server  
  
    kubectl get deployment autoscaler-aws-cluster-autoscaler 
    
Create a Priority yaml file : </br>

        apiVersion: scheduling.k8s.io/v1
        kind: PriorityClass
        metadata:
          name: node-autoscaler
        value: 1000000
        globalDefault: false
        description: "Highest priority for node autoscaler pods"

The Auto-Scale should have a higher priority than most other pods.

Apply the Priority file: </br>

        kubectl apply -f priority.yaml

  
To Test Auto-Scaling, run the command:


        select sum(quantity) from tpch.sf100000.lineitem;




