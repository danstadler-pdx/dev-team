## dev-team

This project represents the OpenTelemetry Demo in rehydrated form and kustomized.

This project will be used with demo projects showing Flux bringing in an external repo for application code.


<br>

## how to deploy this project directly (vs. using Flux to deploy it)

clone the repo

cd into the top level directory

test the output (note no patches are applied, so demo is the same as base):
```
kustomize build opentelemetry-demo/demo
```

create a namespace:
```
kubectl create namespace open-telemetry
```

deploy the demo into a cluster (note that without Alloy available at collector.alloy, the services will write error logs that they can't ship their telemetry):
```
kustomize build opentelemetry-demo/demo | kubectl apply -f -
```


### teardown 
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