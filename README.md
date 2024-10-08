# infrastructure
Shared infrastructure

# deploy infra
```bash
# deploys the contents of helmfile.d (via container, as helmfile requires linux helm plugins)
# NB: The helmfile.d approach helps us achieve 1-click deploy. (cert-man/otel the most problematic.. mostly due to CRDs)
# NB: helmfile "sync" might come in handy, e.g. if a release has previously failed it will be upgraded
docker run --rm --net=host -v "${HOME}/.kube:/helm/.kube" -v "${PWD}:/wd" --workdir /wd ghcr.io/helmfile/helmfile:v0.167.1 helmfile apply

# deploy apps separately from general infra
kubectl apply -f ./manifests/my-apps/

# ports to facilitate local development against common cluster services
start -nnw kubectl 'port-forward -n mq service/rabbitmq 32000:amqp'
start -nnw kubectl 'port-forward -n fileman service/clamav-service 32001:3310'
start -nnw kubectl 'port-forward -n fileman service/gotenberg-service 32002:3000'
start -nnw kubectl 'port-forward -n default service/azurite-service 32003:10000'
start -nnw kubectl 'port-forward -n monitoring service/otel-collector 32004:4317'

# find process for one of the above (e.g. on 32000) and kill by pid (final column on 1st cmd)
netstat -ano | findstr :32000
taskkill /pid NNNNN /F
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
  - 127.0.0.1 prometheus.goaldiggers.dev
  - 127.0.0.1 grafana.goaldiggers.dev
  - 127.0.0.1 rabbit.goaldiggers.dev
  - 127.0.0.1 portalapi.goaldiggers.dev
  - 127.0.0.1 portal.goaldiggers.dev

Apps are (/will be) accessible on:
  - https://prometheus.goaldiggers.dev
  - https://grafana.goaldiggers.dev
  - https://rabbit.goaldiggers.dev
  - https://portalapi.goaldiggers.dev
  - https://portal.goaldiggers.dev

For local/dev purposes, add the `development_ca.crt` to your trusted CAs

In Lens, go to (Cluster) > Settings > Metrics and set:
  - PROMETHEUS: Prometheus Operator
  - PROMETHEUS SERVICE ADDRESS: monitoring/prometheus-server:80

With the above, the CPU and Memory dashboards should show up in Lens on the pods (once prometheus is deployed!)

## Configure grafana
You should now be able to access grafana on the above url (admin:goaldiggersrule!)
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

Use your cluster to save spinning containers up separately! (can be done via NodePorts too..?)
  - kubectl port-forward -n mq service/rabbitmq 32000:amqp
  - kubectl port-forward -n fileman service/clamav-service 32001:3310
  - kubectl port-forward -n fileman service/gotenberg-service 32002:3000
  - kubectl port-forward -n default service/azurite-service 32003:10000

Viewing the promtail-daemonset pod logs also proved useful when writing this guide!

To create a tls secret from cert file(s): (nb: for dev this is now done instead via self-signed cert)
  - kubectl create secret tls ingress-tls-cert --key=tls.key --cert=tls.crt -n NAMESPACE

# Sync helm repositories
Having matching helm repos allows us to identify upgrades / available versions more easily, (inc. in Lens!)

```bash
# match the repos in helmfile!
helm repo add bitnami https://charts.bitnami.com/bitnami; `
helm repo add jetstack https://charts.jetstack.io; `
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx; `
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts; `
helm repo add grafana https://grafana.github.io/helm-charts; `
helm repo add otel https://open-telemetry.github.io/opentelemetry-helm-charts; `
helm repo update;
```