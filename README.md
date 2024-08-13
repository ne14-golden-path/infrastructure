# infrastructure
Shared infrastructure

# deploy infra
```bash
# deploys the contents of helmfile.d (via container, as helmfile requires linux helm plugins)
# NB: The helmfile.d approach is a workaround to allow 1-click deploy, as otel apprarently cant be made to wait for cert-man CRDs :/
# NB: helmfile sync might also come in handy, e.g. if a release has previously failed it will be upgraded
docker run --rm --net=host -v "${HOME}/.kube:/helm/.kube" -v "${PWD}:/wd" --workdir /wd ghcr.io/helmfile/helmfile:v0.167.1 helmfile apply
```

# build docker
```bash
# portal.web image (VS Code)
docker build . -t ne1410s/portal.web:0.0.1 --push

# portal.api image (VS)
docker build . -t ne1410s/portal.api:0.0.1 -f ".\ne14.portal.api\Dockerfile" --force-rm --secret id=nuget_config_file,src="C:\temp\nuget-docker.golden-path.config" --push

# services.docs image (VS)
docker build . -t ne1410s/services.docs:0.0.1 -f ".\ne14.services.docs.app\Dockerfile" --force-rm --secret id=nuget_config_file,src="C:\temp\nuget-docker.golden-path.config" --push
```

# other info
# Pre-k8s one-time (/ anytime)
Add hosts file entries as follows:
  - 127.0.0.1 prometheus.local.ne1410s.co.uk
  - 127.0.0.1 grafana.local.ne1410s.co.uk
  - 127.0.0.1 rabbit.local.ne1410s.co.uk
  - 127.0.0.1 portalapi.local.ne1410s.co.uk
  - 127.0.0.1 portal.local.ne1410s.co.uk

Apps are (/will be) accessible on:
  - https://prometheus.local.ne1410s.co.uk
  - https://grafana.local.ne1410s.co.uk
  - https://rabbit.local.ne1410s.co.uk
  - https://portal.local.ne1410s.co.uk

In OpenLens, go to (Cluster) > Settings > Metrics and set:
  - PROMETHEUS: Prometheus Operator
  - PROMETHEUS SERVICE ADDRESS: monitoring/prometheus-server:80

With the above, the CPU and Memory dashboards should show up in OpenLens on the pods (once prometheus is deployed!)

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

To view rabbitmq's metrics for prometheus:
  - kubectl port-forward -n mq service/rabbitmq 9419
  - http://localhost:9419/metrics

Use your cluster to save spinning containers up separately!  (the below is done automatically in stage06 using NodePorts!)
  - kubectl port-forward -n mq service/rabbitmq-service 32000:5672
  - kubectl port-forward -n fileman service/clamav-service 32001:3310
  - kubectl port-forward -n fileman service/gotenberg-service 32002:3000
  - kubectl port-forward -n default service/azurite-service 32003:10000

Viewing the promtail-daemonset pod logs also proved useful when writing this guide!

# Sync helm repositories
Having matching helm repos allows us to identify upgrades / available versions more easily, (inc. in Lens!)

```bash
# match the repos in helmfile!
helm repo remove bitnami; `
  helm repo add jetstack https://charts.jetstack.io; `
  helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx; `
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts; `
  helm repo add grafana https://grafana.github.io/helm-charts; `
  helm repo add otel https://open-telemetry.github.io/opentelemetry-helm-charts; `
  helm repo update;
```