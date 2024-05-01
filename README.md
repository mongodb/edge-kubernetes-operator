#### Atlas Edge Server - Kubernetes Operator

Atlas Edge Server is a MongoDB instance with a sync server that can be deployed on local or remote infrastructure, enabling real-time sync, conflict resolution, and disconnection tolerance. This helps ensure that mission-critical applications and devices function seamlessly, even with intermittent connectivity.

The Edge Server is packaged in Docker containers, so deploying it can be done using either __docker compose__ or __kubernetes__.

For deploying on __kubernetes__, we supply an [operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) which includes Custom Resource Definitions and a Controller to make it easy to run the Edge Server in your own cluster with minimal setup. The Edge Server also uses a MongoDB container for its internal storage, which is managed by the official [MongoDB Kubernetes Operator](https://github.com/mongodb/mongodb-enterprise-kubernetes).

For testing or development, we recommend using [minikube](https://minikube.sigs.k8s.io/docs/start/) or [k3s](https://k3s.io/).

To install the Edge Server in your kubernetes cluster, follow these steps:

1. Add the MongoDB Helm repo and install the MongoDB Community Operator
```sh
helm repo add mongodb http://mongodb.github.io/helm-charts
helm upgrade --install community-operator mongodb/community-operator --set mongodb.name=mongodb-enterprise-server --set mongodb.repo=quay.io/mongodb
```

3. Fetch the resource file for the Edge Server Operator and `kubectl apply` it.

```sh
curl "https://raw.githubusercontent.com/mongodb/edge-kubernetes-operator/main/release.yaml" | kubectl apply -f -
```

4. Install `edgectl`.

`edgectl` is a command line tool for generating and managing configurations for Edge Server deployments. It can be used to generate a kubernetes resource file to provision a Pod, configured for your app, that will run the edge server in your cluster

To install `edgectl`, run

```
curl https://services.cloud.mongodb.com/edge/install.sh | bash -s
```

5. Generate a configuration for your edge server

```sh
edgectl init --platform kubernetes --app-id <MY-APPLICATION-ID>
```

This will prompt you for a registration token, which is provided from the UI when creating a new edge server. It will generate a YML file and print its path.

6. Use `kubectl` to apply the configuration to the cluster

Use the path produced from the previous step to `kubectl apply`, for example:
```sh
kubectl apply -f /Users/mike/.mongodb-edge/profiles/stores-NY-42/kube-resource.yml
```

This should be all you need! Use `kubectl get pods` to monitor the deployment. A completed deployment will look something like this:

```
$ kubectl get pods
NAME                                                              READY   STATUS    RESTARTS      AGE
mongodb-atlas-edge-operator-controller-manager-565b79476d-4t248   2/2     Running   0             11m
mongodb-kubernetes-operator-f6f956684-rbtz4                       1/1     Running   0             62m
stores-NY-42-0                                                    2/2     Running   0             58m
stores-NY-42-backing-db-0                                         2/2     Running   0             58m
```