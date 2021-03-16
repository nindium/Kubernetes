Scope:
    Create EKS cluster.
    Add another IAM user as Kubernetes cluster user and give Full Administrator rights to it.
    Install container insights to provide cluster monitoring in CloudWatch.
    Install and configure Influxdb and Heapster to get cluster monitoring ability.

Exit creteria:
    EKS cluster in AWS is created with 2 working nodes.
    AWS <Admin> user is able to get access to cluser objects
    and perform kubectl command on them.
    Container Insights is installed. Cluster's metrics are collecting
    and viewable via CloudWatch. 
    InfluxDB and heapster are installed. Cluster's metrics are collecting
    and viewable via Kubernetes dashboard. 

Solution:
To create EKS cluster "eksctl" tool was used.
At first, I created yaml configuration file "eks-conf.yaml" where described basic 
parameters for the cluster: name, region, availability zones, number of working nodes (with min and max sizes) etc.
It takes some time to spin up cluster.

When cluster was created I tried to map AWS user to give him "admin permissions".
For this, according to AWS documentation, I tried to edit aws-auth config map with the command:
    kubectl edit -n kube-system configmap/aws-auth
and added next block under mapUsers section:
    - userarn: <IAM user arn>
      username: <user name to use with kubernetes cluster>
      groups:
      - system:masters

Operation was successful, but I faced with the issue - when tried to switch user with command
    export AWS_PROFILE="aws user name"
and then get cluster information with the command
    kubectl get all
I always got: error: "You must be logged in to the server (Unauthorized)".
When I tried to get access to the cluster objects inside AWS - I got
"Your current user or role does not have access to Kubernetes objects on this EKS cluster." message.
I tried to search solutions, but all of them were pretty same:
    - edit aws-auth configmap to add IAM user to the cluster
    - make RBAC policy to grant this user permissions

I tried to make rbac (RoleBasedAccessControl) policy, but it didn't solve the issue - user was still Unathorized.
So, I would like to check what users existed in the kubernetes cluster.
To do that I found command:
    eksctl get iamidentitymapping --cluster demo-cluster --region us-east-1
I found out that doing user mapping via kubectl was not a choice for this case - new user did not appear in cluster.
Because of that I got "Unauthorized" error.
Actually, I should do this mapping in other way via eksctl, because I created
cluster with it.
So, command will be look like this:
    eksctl create iamidentitymapping --cluster  demo-cluster --arn <IAM user arn> --username <username> --group system:masters --region us-east-1
Because I maped user to system:masters group - I didn't need to make another RBAC policy. It would be enough to grant
new user "admin" permissions inside the cluster.
After that I was able to perform any cluster related command via kubectl,
got rid of error message in AWS console and was able to see cluster resources inside AWS.
First "Exit createria" is met.
Other usefull command:
    eksctl delete iamidentitymapping --cluster  my-cluster-1 --arn arn:aws:iam::123456:role/testing / delete mapping role or user /
    eksctl delete cluster -f eks-conf.yaml   / delete cluster /
    kubectl --user=eks@demo-cluster.us-east-1.eksctl.io get pods -n default / run kubectl with other user's permissions /
    kubectl -n kube-system get configmap aws-auth -o yaml / show current aws-auth configmap in yaml format /
    kubectl logs <podname>  -n <namespace>
    kubectl cluster-info dump | grep -m 1 service-cluster-ip-range / show cluster services network CIDR /
    kubectl cluster-info dump | grep -m 1 cluster-cidr / show cluster POD network CIDR /


To configure Container Insights you need to perform next steps:
1. In IAM Assign CloudWatchAgentServerPolicy to "cluser service role" and "node instace role".
2. Install CloudWatch agent and FluentD:
    - Create a Namespace for CloudWatch:
        kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
    - Create a Service Account in the Cluster:
        kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-serviceaccount.yaml
    - Create and apply ConfigMap for the CloudWatch Agent
        kubectl apply -f cwagent-configmap.yaml
    - Deploy the CloudWatch Agent as a DaemonSet:
        kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-daemonset.yaml
    - Deploy FluentD image (can be replaced by own)
        curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/cluster-name/;s/{{region_name}}/cluster-region/" | kubectl apply -f -
        

InfluxDB and Heapster.
1. Install Kubernetes metric server:
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
2. Install Kubernetes DashBoard (Important!! Choose dashboard version that supports version of your Kubernetes cluster):
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
3. Make deployment file and install InfluxDB in the kube-system namespace.
4. Make ServiceAccount and deployment for heapster and apply it via kubectl in the kube-system namespace.
5. Make Secret, ServiceAccount and deployment for dashboard and apply it via kubectl in the kube-system namespace.
6. Start kubectl proxy.
7. Obtain token to login into dashboard
    kubectl -n kube-system describe secrets $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
    (dashboard url: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login  )




