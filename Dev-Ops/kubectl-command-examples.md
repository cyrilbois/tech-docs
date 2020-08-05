---
title:  kubectl 命令示例 
...

## Deployment
```
kubectl create -f mydeployment-definition.yaml  --record
kubectl rollout status  deployment/mydeployment
kubectl rollout history  deployment/mydeployment
kubectl apply -f mydeployment-definition.yaml  
kubectl set image deployment/frontend simpele-webapp=kodekloud/webapp-color:v2
```
