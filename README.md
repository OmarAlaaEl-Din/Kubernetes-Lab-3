# Kubernetes Networking, DNS Discovery & Ingress Traffic Management

## 📌 Lab Overview
This repository showcases the implementation of complex networking scenarios in Kubernetes (K3s). The lab demonstrates the ability to manage inter-namespace communication, configure service discovery via SRV records, and implement advanced L7 routing using Traefik Ingress Controller with custom Middlewares.

---

## 🏗️ Phase 1: Service Discovery & Cluster DNS

### 1. Workload Isolation (Namespacing)
To ensure resource isolation and security, a dedicated namespace `iti` was provisioned. An Nginx-based deployment was launched with 2 replicas to demonstrate internal load balancing.

* **Action:** Created `iti` namespace and `web` deployment.

![Step 1: Namespace & Deployment](./Screenshot%202026-04-13%20010214.png)

### 2. Service Exposure & Port Naming
The deployment was exposed via a `NodePort` service on port `5000`. 
**Technical Challenge:** Standard `kubectl expose` doesn't name ports, which prevents CoreDNS from generating SRV records.
**Solution:** Applied a JSON patch to name the port `http`, enabling full DNS discovery capabilities.

* **Action:** Expose deployment and patch service for SRV compatibility.

![Step 2: Service Exposure](./Screenshot%202026-04-13%20010247.png)

![Step 3: JSON Patching for DNS](./Screenshot%202026-04-13%20010248.png)

### 3. DNS Validation (SRV & FQDN)
Verification was conducted using a `dnsutils` pod. We successfully resolved the SRV record `_http._tcp.web.iti.svc.cluster.local`, confirming that the cluster DNS is correctly tracking service metadata.

* **Validation:** SRV record lookup and cross-namespace `curl` test.

![Step 4: DNS Pod Setup](./Screenshot%202026-04-13%20012228.png)

![Step 5: SRV Resolution Proof](./Screenshot%202026-04-13%20014857.png)

![Step 6: FQDN Connectivity Test](./Screenshot%202026-04-13%20020136.png)

![Step 7: Final 200 OK Response](./Screenshot%202026-04-13%20020218.png)

---

## 🌐 Phase 2: Advanced Ingress & Path Manipulation

### 1. Multi-App Orchestration
Simulating a multi-regional production environment by deploying `africa` and `europe` applications within a `world` namespace. Both apps are abstracted behind `ClusterIP` services on port `8888`.

* **Action:** Deploying regional apps and cluster services.

![Step 8: World Namespace Setup](./Screenshot%202026-04-13%20184909.png)

![Step 9: Regional Workloads](./Screenshot%202026-04-13%20185054.png)

![Step 10: ClusterIP Definition](./Screenshot%202026-04-13%20185210.png)

### 2. Solving the "Path-to-Root" 404 Issue
**The Problem:** The applications are designed to serve content at the root `/`. However, the Ingress was configured to route via `/africa` and `/europe`. This resulted in `404 Not Found` as the pods received the full path.

**The Solution (Traefik Middleware):** Implemented a `StripPrefix` Middleware. This logic interceptor strips the path prefix before forwarding the request to the backend service, allowing the app to respond correctly.

* **Configuration:**
    * **Domain:** `world.universe.mine` (mapped via `/etc/hosts`).
    * **Logic:** Integrated `strip-prefix` middleware via Ingress annotations.

* **Final Verification:** Path-based routing successfully returning region-specific content.

![Step 11: Final Middleware & Ingress Implementation](./Screenshot%202026-04-13%20195904.png)

---

---

## 🛠️ Ingress Manifest (Technical Highlights)

The configuration of the Ingress resource is crucial for the path-based routing success. Below is the full YAML manifest used, highlighting two critical technical aspects:

1.  **Traefik Middleware Integration:** The `annotations` section references the `world-strip-prefix@kubernetescrd` middleware to handle path stripping.
2.  **Path Routing Rules:** The `spec.rules` define how traffic for `world.universe.mine` is routed to the `africa` service on port `8888` based on the `/africa` prefix.

### Full Ingress Manifest via `kubectl edit`:

![Step 12: Detailed Ingress Manifest YAML](./Screenshot%202026-04-13%20195937.png)
