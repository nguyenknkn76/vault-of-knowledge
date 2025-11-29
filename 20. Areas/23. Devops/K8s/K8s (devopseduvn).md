---
tags:
  - k8s
---

## 

- What is K8s?
- Why use K8s? 
- When K8s be used? big project, microservices, multi environment, scaling, self healing

### K8s Architecture

![[assets/K8s (devopseduvn)/file-20251126103948306.png]]

![[assets/K8s (devopseduvn)/file-20251126103812620.png]]


![[assets/K8s (devopseduvn)/file-20251126103839473.png]]

### K8s Install Methods

- manual: `kubeadm`
- auto: `kops`, ... 

### Deploy Project on K8s Flows

![[assets/K8s (devopseduvn)/file-20251126104816482.png]]


### K8s file (.yaml)

```yaml
apiVersion:
 
# type of resources
kind:
metadata:
spec: 
```
