# Ingress Controllers

- In order for the Ingress resource to work, the cluster must have an ingress controller running.
- Unlike other types of controllers which run as part of the kube-controller-manager binary, Ingress controllers are not started automatically with a cluster.
- Kubernetes as a project currently supports and maintains GCE and nginx controllers.

## Additional controllers
- AKS Application Gateway Ingress Controller is an ingress controller that enables ingress to AKS clusters using the Azure Application Gateway.
- Ambassador API Gateway is an Envoy based ingress controller with community or commercial support from Datawire.
- AppsCode Inc. offers support and maintenance for the most widely used HAProxy based ingress controller Voyager.
- AWS ALB Ingress Controller enables ingress using the AWS Application Load Balancer.
- Contour is an Envoy based ingress controller provided and supported by VMware.
- Citrix provides an Ingress Controller for its hardware (MPX), virtualized (VPX) and free containerized (CPX) ADC for baremetal and cloud deployments.
- F5 Networks provides support and maintenance for the F5 BIG-IP Controller for Kubernetes.
- Gloo is an open-source ingress controller based on Envoy which offers API Gateway functionality with enterprise support from solo.io.
- HAProxy Ingress is a highly customizable community-driven ingress controller for HAProxy.
- HAProxy Technologies offers support and maintenance for the HAProxy Ingress Controller for Kubernetes.
- Istio based ingress controller Control Ingress Traffic.
- Kong offers community or commercial support and maintenance for the Kong Ingress Controller for Kubernetes.
- NGINX, Inc. offers support and maintenance for the NGINX Ingress Controller for Kubernetes.
- Skipper HTTP router and reverse proxy for service composition, including use cases like Kubernetes Ingress, designed as a library to build your custom proxy
- Traefik is a fully featured ingress controller (Let’s Encrypt, secrets, http2, websocket), and it also comes with commercial support by Containous.