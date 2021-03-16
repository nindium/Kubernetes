Install kubernetes dashboard, InfluxDB and Heapster.
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