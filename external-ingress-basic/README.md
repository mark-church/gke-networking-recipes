# Basic External Ingress

External Ingress on GKE deploys a global External HTTP(S) Load Balancer for public internet load balancing. This example deploys an application on GKE and exposes the application with a public load balanced IP address.


**Use Cases:**
- Public exposure of a GKE application on the internet
- HTTP host and path-based load balancing for one to many Services behind the same public VIP


![basic external ingress](images/external-ingress-basic.png)

### Ingress Manifests

In this example an external Ingress resource matches for HTTP traffic with `foo.example.com` and sends it to the `foo` Service at port 80. A public IP is automatically provisioned by the Ingress controller which listens for internet traffic on port 80. The Ingress resource below shows that there is one host match. Any traffic which does not match this is sent to the default backend to provide 404 responses. 


```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: foo-external
spec:
  rules:
  - host: foo.example.com
    http:
      paths:
      - backend:
          serviceName: foo
          servicePort: 80
```

The following `foo` Service selects across the Pods from the `foo` Deployment. This Deployement consists of three Pods which will get load balanced across. Note the use of the `cloud.google.com/neg: '{"ingress": true}'` annotation. This enables container native load balancing which is a best practice. In GKE 1.17+ this is annotated by default.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: foo
  annotations:
    cloud.google.com/neg: '{"ingress": true}'
spec:
  ports:
  - port: 80
    targetPort: 8080
    name: http 
  selector:
    app: foo
  type: ClusterIP
```

### Example Setup

1. Download this repo and navigate to this folder

```sh
$ git clone https://github.com/mark-church/gke-networking-recipes
$ cd gke-networking-recipes/external-ingress-basic
```

2. Deploy the Ingress, Deployment, and Service resources.

```sh
$ kubectl apply -f external-ingress-basic.yaml
ingress.networking.k8s.io/foo-external created
service/foo created
deployment.apps/foo created

```


3. It will take up to a minute for the Pods to deploy and up to a few minutes for the Ingress resource to be ready. Validate their progress and make sure that no errors are surfaced in the resource events.


### Comments



### Cleanup

```sh
kubectl delete -f external-ingress-basic.yaml
```