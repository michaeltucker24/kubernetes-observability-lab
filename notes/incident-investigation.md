# Incident Investigation Note

## Summary

A curl-based load generator was deployed to create traffic against the `nginx-observability-demo` service.

The purpose of this test was to observe pod-level CPU activity using Grafana, Prometheus, and `kubectl top pods`.

## Detection

Grafana Explore showed measurable CPU usage from the `nginx-load-generator` pods.

The load-generator pods created visible activity that could be queried through Prometheus using Grafana Explore.

## PromQL Query Used

The following PromQL query was used in Grafana Explore:

```promql
sum(rate(container_cpu_usage_seconds_total{pod=~"nginx-load-generator.*"}[5m])) by (pod)
```

This query showed CPU usage for pods with names starting with `nginx-load-generator`.

## Command-Line Verification

The following command was used to verify live pod CPU usage from the terminal:

```bash
kubectl top pods
```

The output showed that the `nginx-load-generator` pods were using noticeably more CPU than the `nginx-observability-demo` pods.

This confirmed that the load generator was creating measurable activity inside the Kubernetes cluster.

## Investigation Steps

1. Confirmed the NGINX deployment was running.

```bash
kubectl get deployment
```

2. Confirmed the NGINX pods were running.

```bash
kubectl get pods
```

3. Confirmed the NGINX service existed.

```bash
kubectl get svc
```

4. Created a curl-based load-generator deployment.

```bash
kubectl create deployment nginx-load-generator \
  --image=curlimages/curl \
  -- /bin/sh -c 'while true; do curl -s http://nginx-observability-demo.default.svc.cluster.local > /dev/null; done'
```

5. Scaled the load generator to multiple replicas.

```bash
kubectl scale deployment nginx-load-generator --replicas=3
```

6. Verified live pod CPU usage.

```bash
kubectl top pods
```

7. Opened Grafana Explore and queried Prometheus directly using PromQL.

```promql
sum(rate(container_cpu_usage_seconds_total{pod=~"nginx-load-generator.*"}[5m])) by (pod)
```

8. Compared CPU activity between the load-generator pods and the NGINX workload pods.

9. Deleted the load-generator deployment after testing.

```bash
kubectl delete deployment nginx-load-generator
```

## Resolution / Cleanup

The load-generator deployment was removed after testing.

```bash
kubectl delete deployment nginx-load-generator
```

## Verification After Cleanup

After cleanup, the original NGINX workload remained running.

```bash
kubectl get pods
```

The `nginx-load-generator` pods were no longer present.

## What I Learned

- Grafana dashboards may not always show the exact metric needed.
- Grafana Explore can be used to query Prometheus directly.
- PromQL can help investigate pod-level CPU usage.
- `kubectl top pods` is useful for confirming live Kubernetes resource usage.
- A load generator can be used in a lab to create visible monitoring activity.
- Prometheus and Grafana can be used together to investigate Kubernetes workload behavior.