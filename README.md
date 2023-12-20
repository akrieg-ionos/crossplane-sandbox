# Crossplane

## prerequisites

### k8s tools

```bash
brew install kind
brew install helm
```
### Install Crossplane CLI

```bash
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
```

## Create a cluster

```bash
kind create cluster --name crossplane --config kind-cluster.yaml
```

## Install Crossplane

```bash
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane
```

## Install Crossplane Provider K8s

```bash
crossplane xpkg install provider crossplanecontrib/provider-kubernetes:main
```

## Create a k8s provider config

```bash
kubectl apply -f k8s-provider-config.yaml
SA=$(kubectl -n crossplane-system get sa -o name | grep provider-kubernetes | sed -e 's|serviceaccount\/|crossplane-system:|g')
kubectl create clusterrolebinding provider-kubernetes-admin-binding --clusterrole cluster-admin --serviceaccount="${SA}"
```

## Create a k8s object

```bash
kubectl apply -f k8s-namespace-object.yaml
```

## ionoscloud provider


### install CRDs

```bash
wget https://github.com/ionos-cloud/crossplane-provider-ionoscloud/archive/main.zip
unzip main.zip && mv crossplane-provider-ionoscloud-master crossplane-provider-ionoscloud && rm main.zip
kubectl apply -f crossplane-provider-ionoscloud/package/crds/ -R
```

### create secret for ionos cloud

create a file called `.ionos_credentials` with the following content:

```bash
export IONOS_USERNAME="<your ionos username>"
export IONOS_PASSWORD="<your ionos password>"
export BASE64_PW=$(echo -n "${IONOS_PASSWORD}" | base64)
```

```bash
. ./.ionos_credentials
kubectl create secret generic --namespace crossplane-system ionos-provider-secret --from-literal=credentials="{\"user\":\"${IONOS_USERNAME}\",\"password\":\"${BASE64_PW}\"}"
```

### install provider

```bash
kubectl apply -f ionos-provider.yaml
```

### create a datacenter

```bash
kubectl apply -f datacenter.yaml
```

### create a lan

```bash
kubectl apply -f lan.yaml
```

### create a k8s cluster

```bash
kubectl apply -f k8s-cluster.yaml
```

### create a lan

```bash
kubectl apply -f lan.yaml
```

