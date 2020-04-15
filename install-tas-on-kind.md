# Installing TAS for Kubernetes on Kind
*Tas is currently in the early stages of development and features are changing rapidly*

## Required 

Download TAS for kubernetes, extract it and go to this directory - [Download TAS for Kubernetes here](https://network.pivotal.io/products/tas-for-kubernetes/)
```
#after downloading and placing in prefered location
cd tas-for-kubernetes
```
We need a directory to keep yaml files that will configure TAS for k8s to our kind cluster
```
mkdir configuration-values
```
CF for k8s is the open source project TAS for k8s suppports, we will need files from their repo so we clone it:
```
curl https://raw.githubusercontent.com/cloudfoundry/cf-for-k8s/master/config-optional/remove-resource-requirements.yml > custom-overlays/remove-resource-requirements.yml

curl https://raw.githubusercontent.com/cloudfoundry/cf-for-k8s/master/config-optional/use-nodeport-for-ingress.yml > custom-overlays/use-nodeport-for-ingress.yml
```

We also need to move this file
```
mv custom-overlays/replace-loadbalancer-with-clusterip.yaml config-optional/.
```

[Download Kind for your OS](https://kind.sigs.k8s.io/docs/user/quick-start/)

You need the following CLIs on your system to be able to run the install script `bin/install-tas.sh` and the value generation script `bin/generate-values.sh`:

[`Install CLI tools`](http://docs-pcf-staging.cfapps.io/tas-kubernetes/0-1/installing-command-line-tools.html)


Create your kubernetes cluster
```
kind create cluster --config=cf-for-k8s/deploy/kind/cluster.yml --image kindest/node:v1.16.4 
```
Access your cluster
```
kubectl cluster-info --context kind-kind
```
Use Bosh to generate your SSL certificates and passwords
```
bin/generate-values.sh -d "vcap.me" > ./configuration-values/tas-values.yml
```

In the file you just generated  `./configuration-values/tas-values.yml`, paste the following to the bottom of it adding your credentials for Tanzu Network and DockerHub: 
```
# Add your password for Tanzu Network
system_registry: 
    hostname: registry.pivotal.io 
    username: username@vmware.com
    password: “password123”

# You can modify your repo, username, and password.
app_registry:
  hostname: https://index.docker.io/v1/
  repository: cf-workloads
  username: <username>
  password: “password123”
```

## Deploy TAS to your Kind cluster

Feed the script the configuration for your cluster
```
./bin/install-tas.sh configuration-values
```

## Authenticate and Configure TAS

Use the CF CLI to target the installation at the system domain configured earlier:
```
cf api --skip-ssl-validation https://api.vcap.me
```
The following command sets the CF_ADMIN_PASSWORD environment variable to the key cf_admin_password in the tas-values.yml file:
```
CF_ADMIN_PASSWORD="$(bosh interpolate tas-values.yml --path /cf_admin_password)"
```
Log in as admin
```
cf auth admin "$CF_ADMIN_PASSWORD"
```

Create, then target an org and space for your team
```
# Create an org
cf create-org test-org
# Create a space in your org for your team
cf create-space -o test-org test-space
# Target your team's space and org.
cf target -o test-org -s test-space
# Enable docker containers feature
cf enable-feature-flag diego_docker
```

## Deploy an app from source code 

For a sample app you can youse the one from CF for K8s available from their repo
```
git clone https://github.com/cloudfoundry/cf-for-k8s.git
```

You can now push the code and soon visit your URL
```
cf push test-node-app -p ./cf-for-k8s/tests/smoke/assets/test-node-app
```

## To Delete TAS for Kubernetes
```
kapp delete -a cf
```