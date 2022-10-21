# Auto-Scaling Starburst on an EKS Cluster

Follow the instructions as per the Docs: </br>

https://docs.starburst.io/starburst-enterprise/admin-topics/autoscaling.html

Important: When the Cluster is created (e.g. using EKS yaml), the following policy should be defined.</br>

Sample EKS yaml: </br> 

        iam:
          withAddonPolicies:
          autoScaler: true
          
The value of $your_cluster_name_here in the Autoscaler yaml and needs to match the SEP Cluster name. </br>

For EKS the related docs can be found below: </br>

https://eksctl.io/usage/autoscaling/#enable-auto-scaling

Customize the Auto-scaler yaml to match the SEP Cluster : </br>

    autoDiscovery:
        clusterName: $your_cluster_name_here
        tags:
          - k8s.io/cluster-autoscaler/enabled
          - k8s.io/cluster-autoscaler/$your_cluster_name_here

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
        
The Cluster yaml should have a Min/Max Replicas set (don't set the "replicas" explicitly). </br>

        worker:
          autoscaling:
            enabled: true
            minReplicas: 1
            maxReplicas: 3
            targetCPUUtilizationPercentage: 50 # Default is 80 but for demo and testing you can set this lower to something like 50.
          # The deploymentTerminatinoGracePeriodSeconds controls how long the worker waits before beginning the graceful shutdown process.
          # The following value was lowered for demo purposes 
          deploymentTerminationGracePeriodSeconds: 10 # default is 300; it is actually how long the graceful shutdown waits after it receives the SIGTERM
          starburstWorkerShutdownGracePeriodSeconds: 120 # default is 120
  
To Test Auto-Scaling, run the command below in multiple tabs (e.g. 5 ) in the Query Editor: </br> 


        select sum(quantity) from tpch.sf100000.lineitem;

In the Starburst Insights UI, monitor the Worker Node(s) getting added to the Cluster. </br> 
There is a lag in the new worker appearing in the UI. </br>

The auto-scaling new pod(s) can also be seen being added in k8s (e.g. using k9s to monitor). 



