# Problem 3: Diagnose Me Doctor

## Scenario

A VM running Ubuntu 24.04 with 64GB storage is consistently showing 99% memory usage. This VM only runs NGINX as a load balancer to route traffic to upstream services.

## Initial Investigation Steps

1. **Assess current memory usage**

```bash
free -h
top -o %MEM
ps aux --sort=-%mem | head -n 20
```

2. **Examine NGINX processes**

```bash
ps aux | grep nginx
pmap -x $(pidof nginx) | sort -n -k3
```

3. **Check NGINX and system logs**

```bash
tail -f /var/log/nginx/error.log
dmesg | grep -i "out of memory"
journalctl -k | grep -i "out of memory"
```


## Potential Root Causes and Solutions

### 1. Excessive Worker Processes

**Cause**: Too many NGINX worker processes configured relative to available memory.

**Impact**: System memory exhaustion, degraded performance, and potential service outages.

**Recovery Steps**:

- Adjust worker processes to match available CPU cores:

```nginx
worker_processes auto;  # Or set to specific number based on CPU cores
```

- Restart NGINX: `sudo systemctl restart nginx`


### 2. High Worker Connections Setting

**Cause**: Excessive concurrent connections allowed per worker.

**Impact**: Memory overutilization as each connection requires memory allocation.

**Recovery Steps**:

- Optimize worker connections:

```nginx
events {
    worker_connections 1024;  # Adjust based on workload
}
```

- Restart NGINX to apply changes


### 3. Oversized Buffer Configuration

**Cause**: Excessively large buffer sizes in NGINX configuration.

**Impact**: Inefficient memory usage, reduced capacity for concurrent connections.

**Recovery Steps**:

- Optimize buffer settings:

```nginx
client_body_buffer_size 16k;
client_header_buffer_size 1k;
large_client_header_buffers 4 8k;
proxy_buffer_size 16k;
proxy_buffers 4 32k;
```

- Restart NGINX to apply changes


### 4. Memory Leak

**Cause**: Memory leak in NGINX or a third-party module.

**Impact**: Continuous memory growth until system becomes unstable.

**Recovery Steps**:

- Update NGINX to latest stable version
- Disable problematic modules if identified
- Implement monitoring to detect and restart NGINX when memory thresholds are exceeded


### 5. Excessive Caching

**Cause**: Overly aggressive in-memory caching configuration.

**Impact**: High memory utilization for cached content.

**Recovery Steps**:

- Review and adjust caching directives:

```nginx
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;
```

- Consider moving cache to disk for less frequently accessed content


### 6. Suboptimal Load Balancing Algorithm

**Cause**: Inefficient load balancing method causing resource exhaustion.

**Impact**: Uneven distribution of traffic and memory utilization.

**Recovery Steps**:

- Implement more efficient load balancing:

```nginx
upstream backend {
    least_conn;  # Direct traffic to servers with fewer active connections
    server backend1.example.com;
    server backend2.example.com;
}
```


### 7. Long-lived Keepalive Connections

**Cause**: Excessive keepalive timeout values causing too many idle connections.

**Impact**: Memory consumed by idle connections that could be used for active requests.

**Recovery Steps**:

- Optimize keepalive settings:

```nginx
keepalive_timeout 65s;
keepalive_requests 100;
```


## Long-term Monitoring and Prevention

1. **Implement performance monitoring**
    - Set up memory usage alerts
    - Monitor NGINX metrics (connections, requests/second)
2. **Regular load testing**
    - Simulate high traffic to identify bottlenecks
    - Adjust configuration based on results
3. **Scheduled maintenance**
    - Regular updates to latest stable NGINX version
    - Periodic configuration review and optimization
