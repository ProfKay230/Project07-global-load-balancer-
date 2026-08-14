# Project 7 — Google Cloud Global HTTP Load Balancer

## Project Overview

In this project, I deployed two Nginx web servers on Google Compute Engine and placed them behind a Google Cloud global HTTP load balancer.

The load balancer used health checks to determine which backend servers were healthy and distributed incoming HTTP traffic across the available backends.

I also simulated a backend failure and verified that traffic automatically stopped going to the unhealthy server and continued through the healthy backend.


## Architecture

The project follows this traffic flow:

Internet
↓
Global HTTP Load Balancer
↓
Forwarding Rule
↓
HTTP Proxy
↓
URL Map
↓
Backend Service
↓
Health Check
↓
Unmanaged Instance Group
├── web-server-1
└── web-server-2

Both backend VMs run Nginx and serve a unique webpage so that traffic distribution can be identified during testing.
## What I Built

- Created two Google Compute Engine VMs:
  - `web-server-1`
  - `web-server-2`
- Installed and configured Nginx on both servers.
- Created unique webpages to identify each backend.
- Created an unmanaged instance group and added both VMs.
- Created an HTTP health check on port 80.
- Created a global backend service.
- Attached the backend group to the backend service.
- Created a URL map and target HTTP proxy.
- Reserved a global static external IP address.
- Created a global forwarding rule on port 80.
- Verified that both backend servers became healthy.
- Tested traffic distribution through the public load-balancer IP.
- Simulated a backend failure and verified automatic traffic failover.


## Implementation

### 1. Create the Web Servers

Created two Compute Engine VM instances using the same zone and machine type:

- `web-server-1`
- `web-server-2`

### 2. Install Nginx

SSH'd into each VM, updated the package repositories, and installed Nginx.

Each server was given a unique webpage:

- Web Server 1
- Web Server 2

This made it possible to identify which backend handled each request.

### 3. Create the Backend Group

Created an unmanaged instance group named `web-backend-group` and added both VM instances to it.

### 4. Configure Health Checking

Created an HTTP health check named `web-health-check` on port 80.

The health check allows the load balancer to determine whether each backend is available to receive traffic.

### 5. Configure the Backend Service

Created `web-backend-service` and attached the unmanaged instance group to it.

The backend service uses the health check to monitor the backend instances.

### 6. Configure the Load Balancer Frontend

Created:

- URL map: `web-map`
- HTTP proxy: `web-http-proxy`
- Global static IP: `web-lb-ip`
- Forwarding rule: `web-http-forwarding-rule`

The forwarding rule listens for HTTP traffic on port 80 and sends it to the HTTP proxy.
## Health Checks & Testing

### Backend Health

The load balancer health check was verified using:

`gcloud compute backend-services get-health web-backend-service --global`

Both backend servers were confirmed as:

- `web-server-1` — HEALTHY
- `web-server-2` — HEALTHY

### Traffic Distribution

The public load balancer IP was:

`34.95.79.71`

Multiple HTTP requests were sent through the load balancer to verify that traffic reached both backend servers.

The test returned responses from both:

- Web Server 1
- Web Server 2

This confirmed that the load balancer was distributing traffic across the healthy backends.


## Failure & Failover Test

To test the load balancer's fault-tolerance behavior, I deliberately removed the `http-server` network tag from `web-server-1`.

This prevented the health-check firewall rule from applying to that VM.

The health-check status then showed:

- `web-server-1` — UNHEALTHY
- `web-server-2` — HEALTHY

I then sent multiple HTTP requests to the load balancer's public IP.

All successful responses came from `web-server-2`, demonstrating that the load balancer stopped routing traffic to the unhealthy backend and continued serving requests through the healthy backend.

### Recovery Test

After restoring the `http-server` tag on `web-server-1`, I checked the backend health again.

Both servers returned to:

- `web-server-1` — HEALTHY
- `web-server-2` — HEALTHY

This demonstrated backend recovery and automatic re-entry into the load-balancing pool.


## What I Learned

- How a Google Cloud global HTTP load balancer routes external HTTP traffic.
- The role of backend services in connecting the load balancer to backend infrastructure.
- How unmanaged instance groups can be used to group existing Compute Engine VMs.
- How health checks determine backend availability.
- How firewall rules can affect load-balancer health checks.
- How traffic can be distributed across multiple healthy backend servers.
- How a load balancer responds when one backend becomes unhealthy.
- How automatic failover improves application availability and resilience.
- How to verify backend health and troubleshoot unhealthy instances.
- 
