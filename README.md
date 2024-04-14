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

## Access Grafana

1. In the terminal window make sure port for grafana (3000) is public

![ports](doc/ports.png)

2. Open the link and login (admin/admin)

3. In grafana go to Dashboards -> Create Dashboard -> Add visualization -> Configure new datasource

![create dashboard](doc/create-dashboard.png)

![add visualization](doc/add-visualization.png)

![configure new datasource](doc/new-datasource.png)

4. Select prometheus, in the connection field enter URL for prometheus:

![prometheus link](doc/prometheus-link.png)

![connection url](doc/connection-url.png)

5. Press Save and Test

6. Repeat step 3. but at the end select new created datasource

![new visualization](doc/new-visualization.png)

7. Here you can add and test various metrics and save to Dashboard


## Troubleshooting

1. get deployments status

```sh
kubectl get deployment -n sre
```

outpouts

```
NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
grafana                             1/1     1            1           27h
prometheus-kube-state-metrics       1/1     1            1           27h
prometheus-prometheus-pushgateway   1/1     1            1           27h
prometheus-server                   1/1     1            1           27h
upcommerce-app-two                  0/1     1            0           29s
```

2. get detailed info of the deployment

```sh
kubectl describe deployment upcommerce-app-two -n sre
```

outputs:

```
Name:                   upcommerce-app-two
Namespace:              sre
CreationTimestamp:      Sun, 14 Apr 2024 21:26:48 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=upcommerce-app-two
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=upcommerce-app-two
  Containers:
   upcommerce:
    Image:      uonyeka/upcommerce:v3
    Port:       5000/TCP
    Host Port:  0/TCP
    Limits:
      cpu:        10
      memory:     4Gi
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   upcommerce-app-two-56cff9c64d (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m23s  deployment-controller  Scaled up replica set upcommerce-app-two-56cff9c64d to 1
```

_which already indicates the problem (MinimumReplicasUnavailable) but we will continue_

3. get the logs