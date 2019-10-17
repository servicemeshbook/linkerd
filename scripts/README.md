# Linkerd - Commands 

## Install Linkerd

Copy and paste command as you practice.

### Command to list Linkerd releases so far

```
curl -Ls https://api.github.com/repos/linkerd/linkerd2/releases | grep tag_name
```

### To install Linkerd CLI in the VM environment

```
cd ## Switch to the home directory
export LINKERD2_VERSION=stable-2.6.0 
curl -s -L https://run.linkerd.io/install | sh - 
```

### Edit and source your local .bashrc and add linkerd2 to the path

```
vi ~/.bashrc

## Add these two lines

export LINKERD2_VERSION=stable-2.6.0
export PATH=$PATH:$HOME/.linkerd2/bin

source ~/.bashrc
echo $LINKERD2_VERSION
```

### Validate Linkerd client version
```
linkerd version
```

### Prerequisites required for installing Linkerd
```
linkerd check --pre
```

### Grant cluser_admin privilege to the service account default for the linkerd namespace
```
kubectl create clusterrolebinding linkerd-cluster-role-binding \
--clusterrole=cluster-admin --group=system:serviceaccounts:linkerd
```

### Run the linkerd install command
```
linkerd install | kubectl apply -f -
```


### Run linkerd check to make sure if an installation has succeeded or stuck at some step
```
linkerd check
```

### Run linkerd version to check the client and the server version
```
linkerd version
```

### Verify the Linkerd deployments
```
kubectl -n linkerd get deployments
```

### Verify the Linkerd services
```
kubectl -n linkerd get services
```

### Verify Linkerd pods
```
kubectl -n linkerd get pods
```

## Separation of Roles and Responsibilities

### Cluster Administrator

### To create objects that require cluster-admin role run
```
linkerd install config | kubectl apply -f -
```

### To validate the objects, run:
```
linkerd check config
```

### Application Administrator

### To install control pane run:

```
linkerd install control-plane | kubectl apply -f -
```

### Above failed since linkerd is already installed

### Accessing Linkerd Dashboard

```
linkerd dashboard
```

### Ingress Gateway

### We will now install nginx Ingress controller

```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm install nginx-stable/nginx-ingress --name nginx --namespace kube-system \
--set fullnameOverride=nginx \
--set controller.name=nginx-controller \
--set controller.config.name=nginx-config \
--set controller.service.name=nginx-controller \
--set controller.serviceAccount.name=nginx
```

### Check the ingress controller service
```
kubectl -n kube-system get services -o wide -l app.kubernetes.io/instance=nginx
```
### Create an entry in /etc/hosts for the following host
```
export INGRESS_HOST=$(kubectl -n kube-system get service nginx-controller -o jsonpath='{.status.loadBalancer.ingress..ip}') ; echo $INGRESS_HOST

sudo sed -i '/dashboard.linkerd.local/d' /etc/hosts
echo "$INGRESS_HOST dashboard.linkerd.local" | sudo tee -a /etc/hosts
```

### Define an ingress rule to route traffic from dashboard.linkerd.local to Linkerd internal dashboard
```
cat 01-create-linkerd-ingress.yaml
kubectl -n linkerd apply -f 01-create-linkerd-ingress.yaml
```

### Check if the ingress is working
```
curl -s -H "Host: dashboard.linkerd.local" http://$INGRESS_HOST | grep -i title
```

### Installing Linkerd demo emoji app

### Grant grant cluster_admin role to emojivoto
```
kubectl create clusterrolebinding emojivoto-cluster-role-binding \
--clusterrole=cluster-admin --group=system:serviceaccounts:emojivoto
```

### Installing emojivoto application
```
curl -Ls https://run.linkerd.io/emojivoto.yml | kubectl apply -f -
```

### Check the application status
```
kubectl -n emojivoto get deployments,services,pods
```

### Create emojivoto.linked.local entry in /etc/hosts
```
export INGRESS_HOST=$(kubectl -n kube-system get service nginx-controller -o jsonpath='{.status.loadBalancer.ingress..ip}') ; echo $INGRESS_HOST
sudo sed -i '/emojivoto.linkerd.local/d' /etc/hosts
echo "$INGRESS_HOST emojivoto.linkerd.local" | sudo tee -a /etc/hosts
```

### Create emojivoto ingress rule
```
cat 02-create-emojivoto-ingress.yaml
kubectl -n emojivoto apply -f 02-create-emojivoto-ingress.yaml
```

### Check emojivoto app
```
curl -s -H "Host: emojivoto.linkerd.local" http://$INGRESS_HOST | grep -i title
```

### Inject linkerd sidecar proxy to the emoji application
```
kubectl get -n emojivoto deploy -o yaml | linkerd inject - | kubectl apply -f -
```

### let's check deployment, services, and pods
```
kubectl -n emojivoto get deployments
```

### We will check pods
```
kubectl -n emojivoto get pods
```

###  let's check services
```
kubectl -n emojivoto get services
```

## Installing booksapp application

### Admission webhook is enabled automatically when we installed a linkerd control plane
```
kubectl -n linkerd get deploy -l linkerd.io/control-plane-component=proxy-injector
```

### Grant cluster-admin role to linkerd-lab namespace
```
kubectl create clusterrolebinding linkerd-lab-cluster-role-binding \
--clusterrole=cluster-admin --serviceaccount=linkerd:default
```

### Create a namespace linkerd-lab
```
cat 03-create-namespace-sidecar-enabled-annotation.yaml
kubectl apply -f 03-create-namespace-sidecar-enabled-annotation.yaml
```

### Install booksapp microservice application from linkerd.io
```
curl -Ls https://run.linkerd.io/booksapp.yml | kubectl -n linkerd-lab apply -f -
```

### Check the network services.
```
kubectl -n linkerd-lab get svc
```

### Check the pod status of booksapp
```
kubectl -n linkerd-lab get pods
```

### Describe one of the pods above to see its contents
```
kubectl -n linkerd-lab describe pod -l app=authors
```

### Create booksapp.linked.local entry in /etc/hosts
```
sudo sed -i '/booksapp.linkerd.local/d' /etc/hosts
echo "$INGRESS_HOST booksapp.linkerd.local" | sudo tee -a /etc/hosts
```

### Create booksapp ingress rule
```
cat 04-create-booksapp-ingress.yaml
kubectl -n linkerd-lab apply -f 04-create-booksapp-ingress.yaml
```

### Check web app through curl
```
curl -s -H "Host: booksapp.linkerd.local" http://$INGRESS_HOST | grep -i /title
```

### End of Linkerd install and demo apps