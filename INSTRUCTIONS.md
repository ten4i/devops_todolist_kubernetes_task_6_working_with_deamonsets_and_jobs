### 1. Create namespace: 
`kubectl apply -f .infrastructure/namespace.yml`

### 2. Deploy the todoapp application and its ClusterIP service:
```
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/clusterIP.yml
```

### 3. Deploy the DaemonSet and CronJob:
```
kubectl apply -f .infrastructure/daemonset.yml
kubectl apply -f .infrastructure/cronjob.yml
```

### 4. Verify the todoapp Deployment and Service are up:
```
kubectl get pods -n todoapp -l app=todoapp
kubectl get svc -n todoapp todoapp-service
```

### 5. Verify DaemonSet pods were created: 
```
kubectl get pods -n mateapp -l app=todoapp-curl
```

### 6. Check DaemonSet pod logs to confirm it can reach the todoapp service:
```
kubectl logs -n mateapp -l app=todoapp-curl
```

### 7. Verify the CronJob was created and check its schedule:
```
kubectl get cronjob -n mateapp todoapp-health-check
```

### 8. Wait for a scheduled run (up to 4 minutes) and list the Jobs it creates:
```
kubectl get jobs -n mateapp
```

### 9. Check the logs of the latest CronJob pod to confirm the health check succeeded:
```
kubectl logs -n mateapp -l app=mateapp --tail=20
```
