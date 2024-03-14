# Pre-k8s one-time (/ anytime)
Add hosts file entries as follows:
  - 127.0.0.1 prometheus.local.ne1410s.co.uk
  - 127.0.0.1 grafana.local.ne1410s.co.uk
  - 127.0.0.1 jaeger.local.ne1410s.co.uk
  - 127.0.0.1 rabbit.local.ne1410s.co.uk
  - 127.0.0.1 fileman.local.ne1410s.co.uk
  - 127.0.0.1 portalapi.local.ne1410s.co.uk
  - 127.0.0.1 portal.local.ne1410s.co.uk

Apps are (/will be) accessible on:
  - https://prometheus.local.ne1410s.co.uk
  - https://grafana.local.ne1410s.co.uk
  - https://jaeger.local.ne1410s.co.uk
  - https://rabbit.local.ne1410s.co.uk
  - https://fileman.local.ne1410s.co.uk
  - https://portal.local.ne1410s.co.uk
  
In OpenLens, go to (Cluster) > Settings > Metrics and set:
  - PROMETHEUS: Prometheus Operator
  - PROMETHEUS SERVICE ADDRESS: monitoring/prometheus-service:8080

With the above, the CPU and Memory dashboards should show up in OpenLens on the pods (once prometheus is deployed!)

Install helm and add grafana repo:
  - helm repo add grafana https://grafana.github.io/helm-charts
  - helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
  - helm repo update

# Deployment
Run the following in order. Wait 20s or so between each:
  - kubectl apply -f "<REPO>\k8s\stage01"
  - kubectl apply -f "<REPO>\k8s\stage02"
  - <ISSUE_CERTS> - see below!
  - helm upgrade --install loki --namespace monitoring --values "<REPO>\k8s\stage03_helm\loki-helm-values.yaml" grafana/loki
  - helm upgrade --install tempo --namespace monitoring grafana/tempo
  - helm upgrade --install opentelemetry-operator --namespace monitoring open-telemetry/opentelemetry-operator
  - kubectl apply -f "<REPO>\k8s\stage04"
  - kubectl apply -f "<REPO>\k8s\stage05"

## Apply ssl certificate payloads as secrets
  - cd to directory containing SSL cert files
  - add a secret for each ingress namespace
    - `kubectl create secret tls ingress-tls-cert -n NAMESPACE --key=tls.key --cert=tls.crt`
  - current ingress namespaces:
    - `portal | mq | fileman | monitoring`
*NB: prometheus, grafana, etc are configured on the monitoring namespace, so don't require separate secrets).*

## Configure grafana
You should now be able to access grafana on the above url (admin:admin)
There should be preconfigured datasources for loki and prometheus (we added this in grafana/monitoring manifest)
We're now free to add prometheus- and loki- related dashboards!!
  - Starter prometheus dashboard id: 8588
  - Starter loki dashboard id: 12019

# Troubleshooting
One way to directly access loki-specific metrics is by port-forwarding the service:
  - kubectl port-forward -n monitoring service/loki 3100
  - http://localhost:3100/metrics

To directly access promtail targets, etc we can port-forward the daemonset:
  - kubectl port-forward -n monitoring daemonset/promtail-daemonset 9080
  - http://localhost:9080/targets
  
To view zpages (opentel experimental ui), port-forward as follows:
  - kubectl port-forward -n monitoring service/otel-collector 55679
  - http://localhost:55679/debug/servicez
  - http://localhost:55679/debug/tracez
  
To view open-telemetry metrics directly:
  - kubectl port-forward -n monitoring service/otel-collector 59090
  - http://localhost:59090/metrics
  
Use your cluster mq to save spinning up separately:
  - kubectl port-forward -n mq service/rabbitmq-service 5670:5672
  - can now specify RabbitMq:Hostname = 5670 locally

Viewing the promtail-daemonset pod logs also proved useful when writing this guide!
