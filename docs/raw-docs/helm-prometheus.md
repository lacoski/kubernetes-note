https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack#configuration

# Cập nhật Repo

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update

# Install Chart

helm show values prometheus-community/kube-prometheus-stack

mkdir -p /root/prometheus-stack
cd /root/prometheus-stack

[root@master1181 prometheus-stack]# cat custom-values.yaml 
grafana:
  adminPassword: demoprometheus
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
    hosts:
      - hello-app-1.local

helm install k8s-prometheus prometheus-community/kube-prometheus-stack --dry-run --debug -f /root/prometheus-stack/custom-values.yaml

helm install k8s-prometheus prometheus-community/kube-prometheus-stack -f /root/prometheus-stack/custom-values.yaml

[root@master1181 prometheus-stack]# helm install k8s-prometheus prometheus-community/kube-prometheus-stack -f /root/prometheus-stack/custom-values.yaml
W1104 18:01:06.316654   32073 warnings.go:67] rbac.authorization.k8s.io/v1beta1 Role is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 Role
W1104 18:01:06.322885   32073 warnings.go:67] rbac.authorization.k8s.io/v1beta1 RoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 RoleBinding
W1104 18:01:06.357148   32073 warnings.go:67] networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
W1104 18:01:16.999729   32073 warnings.go:67] rbac.authorization.k8s.io/v1beta1 Role is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 Role
W1104 18:01:17.013615   32073 warnings.go:67] rbac.authorization.k8s.io/v1beta1 RoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 RoleBinding
W1104 18:01:17.295778   32073 warnings.go:67] networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME: k8s-prometheus
LAST DEPLOYED: Wed Nov  4 18:01:04 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace default get pods -l "release=k8s-prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.

[root@master1181 prometheus-stack]# kubectl get services
NAME                                      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                     ClusterIP      None             <none>        9093/TCP,9094/TCP,9094/UDP   46s
k8s-ingress-nginx-ingress                 LoadBalancer   10.106.37.130    10.10.11.94   80:30307/TCP,443:31958/TCP   6h52m
k8s-prometheus-grafana                    ClusterIP      10.102.137.207   <none>        80/TCP                       49s
k8s-prometheus-kube-promet-alertmanager   ClusterIP      10.100.35.184    <none>        9093/TCP                     49s
k8s-prometheus-kube-promet-operator       ClusterIP      10.104.240.239   <none>        443/TCP                      49s
k8s-prometheus-kube-promet-prometheus     ClusterIP      10.103.5.14      <none>        9090/TCP                     49s
k8s-prometheus-kube-state-metrics         ClusterIP      10.96.187.126    <none>        8080/TCP                     49s
k8s-prometheus-prometheus-node-exporter   ClusterIP      10.106.65.178    <none>        9100/TCP                     49s
kubernetes                                ClusterIP      10.96.0.1        <none>        443/TCP                      2d9h
prometheus-operated                       ClusterIP      None             <none>        9090/TCP                     46s

[root@master1181 prometheus-stack]# kubectl get ingress
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME                     CLASS    HOSTS               ADDRESS       PORTS   AGE
k8s-prometheus-grafana   <none>   hello-app-1.local   10.10.11.94   80      78s
