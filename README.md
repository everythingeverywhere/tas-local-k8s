# Installing TAS for Kubernetes on Kind (WIP)
### *Currently in the early stages of development and features are changing rapidly*

Create your kubernetes cluster
```
kind create cluster --config=./cf-for-k8s/deploy/kind/cluster.yml --image kindest/node:v1.16.4 
```
 Access your cluster
```
kubectl cluster-info --context kind-kind
```
Use Bosh to generate your SSL certificates and passwords
```
./bin/generate-values.sh -d "vcap.me" > ./tas-values.yml
```

Modify ./tas-values.yml you just generated and include your credentials for Tanzu Network and 
```
system_registry: 
    hostname: registry.pivotal.io 
    username: username@vmware.com
    password: “password123”

app_registry:
  hostname: https://index.docker.io/v1/
  repository: cf-workloads
  username: tortillas
  password: “password123”
```

#render templates with extra configuration 
```  
ytt -f config/ \
  -f custom-overlays/ \
  -f ./tas-values.yml \
  -f ./cf-for-k8s/config-optional/remove-resource-requirements.yml \
  -f ./cf-for-k8s/config-optional/use-nodeport-for-ingress.yml \
  | kbld -f - -f image_overrides.yml > tas.rendered.yml
```

Deploy TAS for K8S

```
kapp deploy -a cf -f tas.rendered.yml
```

Deploy an app from source code

```
cf push test-node-app -p ./cf-for-k8s/tests/smoke/assets/test-node-app
```


