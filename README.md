## Multi-node k8s cluster with loadbalancer and local, private registry

### To enable local registry, add following to `~/.docker/daemon.json`
```json
"insecure-registries": [
	"registry.localhost:5000"
],
"registry-mirrors": [
	"https://registry-1.docker.io"
]
```

### Start local registry server in docker container
```shell
k3d registry create registry.localhost --port 5000
```

#### Create kubernetes cluster
```shell
k3d cluster create --api-port 0.0.0.0:6550 -p "8081:80@loadbalancer" --registry-use k3d-registry.localhost:5000 --registry-config k3d-registries.yaml --agents 2 mycluster
```

### Build docker image for application
```shell
docker build -t freakynit/go-k8s:1.0.0 .
```

### Tag this image with local registry host
```shell
docker tag freakynit/go-k8s:1.0.0 k3d-registry.localhost:5000/go-k8s:1.0.0
```

### Add following entries to `/etc/hosts`
```text
127.0.0.1 loadbalancer
127.0.0.1 k3d-registry.localhost
::1       k3d-registry.localhost
```

### Push this image to local registry
```shell
docker push k3d-registry.localhost:5000/go-k8s:1.0.0
```

#### Apply kubernetes config file
> Make sure that current context is of this cluster (latest version as of now does this already after creating new cluster)
```shell
kubectl apply -f k8s-deployment.yaml
```

### Check pods are running
```shell
kubectl get pod
```

#### Access service
```shell
curl localhost:8081/
# OR
curl loadbalancer:8081/
```

#### Delete cluster
```shell
k3d cluster delete mycluster
```
