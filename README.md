# Istio in Action Demo - KCD 2025

This repository contains three Istio demos:

1. Path-based Routing Demo
2. Canary Release Demo
3. Traffic Mirroring Demo

## Prerequisites

- Minikube or kind for running a local Kubernetes cluster
- `kubectl` command-line tool for interacting with the cluster
- `curl` installed

## Cluster Setup

1. Create a Kubernetes cluster using minikube:

```bash
minikube start --cpus=4 --memory=6G
```

2. Download and extract Istio 1.25.1:

```bash
curl -L https://istio.io/downloadIstio | sh -
```

3. Install Istio:

```bash
cd istio-1.25.1
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo
```

4. Enable Kiali for observability:

```bash
kubectl apply -f samples/addons/kiali.yaml
```

5. Start minikube tunnel in a separate terminal:

```bash
minikube tunnel
```

6. Verify the Istio ingress gateway is running:

```bash
kubectl get svc -n istio-system istio-ingressgateway
```

## Demo 1: Path-based Routing

This demo shows how to use Istio to route traffic based on URL paths.

### Installation

```bash
kubectl apply -k base
```

### Testing

1. Test version 1 (default route):

```bash
curl -v http://localhost
```

You should see the message "hola version 1"

2. Test version 2 (/v2 route):

```bash
curl -v http://localhost/v2
```

You should see the message "hola version 2"

## Demo 2: Canary Release

This demo shows how to implement a canary release using Istio, sending 10% of the traffic to the new version.

### Installation

```bash
kubectl apply -k canary
```

### Testing

1. Test the service (traffic will be distributed between v1 and v2):

```bash
curl -v -H "Host: canary-demo.kcd.local" http://localhost
```

2. To see the traffic distribution, you can make multiple requests:

```bash
for i in {1..10}; do curl -s -H "Host: canary-demo.kcd.local" http://localhost | grep hola; done
```

You should see approximately:

- 9 responses of "hola version 1" (90%)
- 1 response of "hola version 2" (10%)

## Demo 3: Traffic Mirroring

This demo shows how to implement traffic mirroring (shadow traffic) using Istio. It splits traffic between v1 and v2 while also mirroring all traffic to v2 for testing.

### Installation

```bash
kubectl apply -k mirroring
```

### Testing

1. Test the service (traffic will be split and mirrored):

```bash
curl -v -H "Host: mirroring-demo.kcd.local" http://localhost
```

2. To see the traffic distribution and mirroring in action, make multiple requests:

```bash
for i in {1..10}; do curl -s -H "Host: mirroring-demo.kcd.local" http://localhost | grep hola; done
```

You should see approximately:

- 8 responses of "hola version 1" (80%)
- 2 responses of "hola version 2" (20%)

3. To verify mirroring is working, check the logs of v2:

```bash
kubectl logs -n mirroring-demo -l app=nginx-v2 -c nginx
```

You should see that v2 is receiving all requests, even though only 20% of responses come from it.

### How it works

- 80% of traffic goes to v1
- 20% of traffic goes to v2
- 100% of traffic is mirrored to v2
- Mirror responses are not sent back to the client
- This allows testing v2 with real production traffic without affecting users

## Troubleshooting

If you're having issues accessing the services:

1. Make sure `minikube tunnel` is running in a separate terminal
2. Verify the Istio ingress gateway is running:

```bash
kubectl get svc -n istio-system istio-ingressgateway
```

3. Check if the pods are running:

```bash
kubectl get pods -n demo-kcd-2025
```

4. Check the Istio proxy logs:

```bash
kubectl logs -n demo-kcd-2025 -l app=nginx-v1 -c istio-proxy
```

## Cleanup

To remove the demos:

```bash

# Clean up canary release demo
kubectl delete -k canary

# Clean up mirroring demo
kubectl delete -k mirroring

# Clean up traffic-routes
kubectl delete -k traffic-routes

# Clean up path-based routing demo
kubectl delete -k base
```

# Remove Istio

```bash
istioctl uninstall
```

To clean up the cluster:

```bash
minikube delete
```
