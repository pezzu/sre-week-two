# SRE: Troubleshooting deployment issues and reporting incidents

## Setup

1. Setup kube

```sh
minikube start
```

2. Create namespace

```sh
kubectl create namespace sre
```

3. Install prometheus

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus -f prometheus.yml --namespace sre
```

4. Install grafana

```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana --set adminPassword="admin" --namespace sre
```

5. Create a deployment and service

```sh
kubectl apply -f deployment.yml -n sre
kubectl apply -f service.yml -n sre
```

6. Port Forward

- Prometheus server

```sh
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace sre port-forward $POD_NAME 9090
```

- Prometheus Alertmanager

```sh
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace sre port-forward $POD_NAME 9093
```

- Prometheus PushGateway

```sh
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/instance=prometheus,app.kubernetes.io/name=prometheus-pushgateway" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace sre port-forward $POD_NAME 9091

```

- Grafana server

```sh
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace sre port-forward $POD_NAME 3000
```

- Application

_This may not work. It's expected_

```sh
minikube service upcommerce-service-two -n sre --url

kubectl port-forward service/upcommerce-service-two -n sre 31936:5000
```

7. Check status

```sh
kubectl get deployment -n sre
```

The output:

```sh
NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
grafana                             1/1     1            1           73m
prometheus-kube-state-metrics       1/1     1            1           74m
prometheus-prometheus-pushgateway   1/1     1            1           74m
prometheus-server                   1/1     1            1           74m
upcommerce-app-two                  0/1     1            0           64m
```

