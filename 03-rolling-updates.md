# Replicas and Rolling update:

Increase replicas:
```
kubectl scale deployment nginx --replicas=4
```

From another terminal check (using load balancer IP) which version is currently running and to see changes when rollout is happening:
```
while true; do  curl -sI 35.205.60.29  | grep Server; sleep 2; done
```

Rollout an update to  the image:
```
kubectl set image deployment/nginx nginx=nginx:1.9.1
```

Check the rollout status:
```
kubectl rollout status deployment/nginx
```

Undo the rollout and restore the previous version:
```
kubectl rollout undo deployment/nginx
```
