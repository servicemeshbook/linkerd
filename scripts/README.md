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




