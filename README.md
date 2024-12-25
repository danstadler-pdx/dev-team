## dev-team

This project represents the OpenTelemetry Demo in rehydrated form and kustomized.

This project will be used with demo projects showing Flux bringing in an external repo for application code.

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