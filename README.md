```bash
oc create ns ai-sno-cluster001
oc create secret generic ai-bmh-secret \
  -n ai-sno-cluster001 \
  --from-literal=username="admin" \
  --from-literal=password="redhat"
  oc create secret generic pull-secret   -n ai-sno-cluster001   --from-file=.dockerconfigjson=./pull-secret   --type=kubernetes.io/dockerconfigjson
```
