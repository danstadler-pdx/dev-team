# dev-team

This project is based on the [OpenTelemetry Demo](https://opentelemetry.io/docs/demo/architecture/).

In this project, the OTel Demo's Helm chart has been rehydrated into manifests, and [kustomize](https://github.com/kubernetes-sigs/kustomize) has been set up alongside them.

The kustomization includes a normal "base" directory, as well as one overlay called "demo".

The rehydration has already removed the OTel Demo's built-in Observability stack. See further down for how that was done. This means that the deployments do not include:
- OTel Collector
- Jaeger
- Prometheus
- OpenSearch
- Grafana

Instead, this deployment of the OTel Demo assumes that you will be running Grafana Alloy behind a K8s Service at "alloy.collector". It is OK to run the project without Alloy ready; you will just get error logs from the services about not being able to ship their telemetry.

This project is being used by other demo projects, showing how Flux can first stand up your infrastructure (Alloy, telemetry backends, etc.), and then bring in kustomized application manifests from an external repo (this project).

FWIW: It is intentional here that the OTel Demo is structured as K8s manifests and Kustomize files; i.e. not by the Demo's Helm chart. This is for demonstration purposes.


<br>

## how to deploy this project directly (vs. using Flux to deploy it)

Clone the repo;

CD into the top level directory;

Test the output (note no patches are applied, so "demo" is the same as base):
```
kustomize build opentelemetry-demo/demo
```

create a namespace:
```
kubectl create namespace open-telemetry
```

deploy the demo into a cluster (note that without Alloy available at alloy.collector, the services will write error logs that they can't ship their telemetry):
```
kustomize build opentelemetry-demo/demo | kubectl apply -f -
```

OTel Demo should load into your cluster at this point; again just the microservices, not the additional O11y stack.

<br>

## teardown 
```
kubectl delete namespace open-telemetry
```


<br>

## how this was created

Here's how the hydration worked:

Set up a values file called "oteldemo-values.yaml"; this one just produces the demo microservices, not the O11y stack:

```
# point the services to Alloy as their OTel Collector
default:
  envOverrides:
    - name: OTEL_COLLECTOR_NAME
      value: alloy.collector

# turn off the included olly stack
opentelemetry-collector:
  enabled: false

jaeger:
  enabled: false

prometheus:
  enabled: false

grafana:
  enabled: false

opensearch:
  enabled: false
```

Then run `helm template` as follows (change the helm chart version as you see fit):

```
helm template otel-demo open-telemetry/opentelemetry-demo \
  --version 0.33.8 \
  --namespace otel-demo \
  --values ./oteldemo-values.yaml \
  --output-dir=./oteldemo-output
```

The output files were copied into this project.
