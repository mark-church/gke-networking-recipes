# gRPC Health Checks

This is a guide for how to implement proper load balancer health checking for gRPC applications built on top of GKE. Specifically, we are concerned with gRPC applications that are exposed via an L7 load balancer using the Ingress API.

We will also discuss a special use case that leverages Extensible Server Proxy (ESP) on GCP.

## Configuring Health Check

The BackendConfig CRD now supports customizing HealthCheck resources. Below are the fields supported in the specification:

```
apiVersion: cloud.google.com/v1
kind: BackendConfig
metadata:
  name: http-hc-config
spec:
  healthCheck:
    checkIntervalSec: interval
    timeoutSec: timeout
    healthyThreshold: health-threshold
    unhealthyThreshold: unhealthy-threshold
    type: [HTTP | HTTPS | HTTP2]
    requestPath: path
```

In order to implement proper health checks for gRPC applications, we will especially pay attention to the `port` and `protocol` fields.

## Limitations

Note that health checks with a protocol of “gRPC” are not supported yet. In other words, only HTTP-like health checking is supported. This leaves two options:

Run a proxy on a separate port that translates HTTP health check requests into gRPC and then forwards the transformed request to the application port that serves a gRPC health check.
Run a dedicated health checker on a separate port that speaks HTTP. This health checker responds to health check requests directly rather than proxying them.

We will provide example YAMLs for both cases, as well as describing nuances in the configuration when using InstanceGroup backends or NetworkEndpointGroup (NEG) backends.

## Examples

### Proxy

Below is an example Deployment specification when using a health check proxy. You may use the image we have referenced below or write your own. Full documentation on our provided image can be found [here](https://github.com/salrashid123/grpc_health_proxy).

```
spec:
 containers:
 - name: hc-proxy
   image: docker.io/salrashid123/grpc_health_proxy
   args: [
     "--http-listen-addr=localhost:8081",
     "--grpc-addr=localhost:8080",
     "--service-name=echo.EchoServer",
   ]
   ports:
   - containerPort: 8081
 - name: grpc-app
   image: gcr.io/mygcr/myapp
   ports:
   - containerPort: 8080
```

Remember that your gRPC application must implement the gRPC health protocol
(grpc.health.v1.Health).

#### Service & BackendConfig

For InstanceGroup backends, the port used for the health check must be exposed in the Service specification. This is needed so that a NodePort can be allocated. This NodePort is then applied to the BackendConfig resource (See below)

For NEG backends, exposing the health check port in the Service is not required.

```
type: Service
spec:
  type: NodePort
  ports:
name: app
port: 80
targetPort: 8080
nodePort: 30000
  # Not needed if using NEG
  - name: proxy
    port: 8081
    targetPort: 8081
    nodePort: 30001

# If using InstanceGroup
type: BackendConfig
spec:
  healthCheck:
    # Note that this is the NodePort for the proxy’s target port.
    port: 30001
    protocol: "HTTP"
    requestPath: [PATH]
---
# If using NEG
type: BackendConfig
spec:
  healthCheck:
    # Note that this is the container port on the Deployment for the proxy.
    port: 8081
    protocol: "HTTP"
    requestPath: [PATH]
```

### Non-Proxy

```
spec:
 containers:
 - name: esp
   image: gcr.io/myrepo/healthchecker
   ports:
   - containerPort: 8081
 - name: grpcapp
   image: gcr.io/mygcr/myapp
   ports:
   - containerPort: 8080
```

#### Service & BackendConfig

```
type: Service
spec:
  type: NodePort
  ports:
name: app
port: 80
targetPort: 8080
nodePort: 30000
  # Not needed if using NEG
  - name: proxy
    port: 8081
    targetPort: 8081
    nodePort: 30001
---
# If using InstanceGroup
type: BackendConfig
spec:
  healthCheck:
    # Note that this is the NodePort for the health checkers target port.
    port: 30001
    protocol: "HTTP"
    requestPath: [PATH]
---
# If using NEG
type: BackendConfig
spec:
  healthCheck:
    # This is the container port on the Deployment for the health checker.
    port: 8081
    protocol: "HTTP"
    requestPath: [PATH]
```

### ESP

TODO
