# Headless Services

- Sometimes you don’t need load-balancing and a single Service IP. In this case, you can create what are termed “headless” Services, by explicitly specifying "None" for the cluster IP (`.spec.clusterIP`).
- You can use a headless Service to interface with other service discovery mechanisms, without being tied to Kubernetes’ implementation.
- For headless Services, a cluster IP is not allocated, kube-proxy does not handle these Services, and there is no load balancing or proxying done by the platform for them. 
- How DNS is automatically configured depends on whether the Service has selectors defined:
    - **With selectors**
        - For headless Services that define selectors, the endpoints controller creates Endpoints records in the API, and modifies the DNS configuration to return records (addresses) that point directly to the Pods backing the Service.
    - **Without selectors**
        - For headless Services that do not define selectors, the endpoints controller does not create Endpoints records. However, the DNS system looks for and configures either:
            - CNAME records for ExternalName-type Services.
            - A records for any Endpoints that share a name with the Service, for all other types.