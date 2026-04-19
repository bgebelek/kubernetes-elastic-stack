# Elastic Stack on Plain Kubernetes

## Purpose
This repository is intended for an audience familiar with Kubernetes to set up, learn about, and develop with the [Elastic Stack](https://www.elastic.co/docs).

## Usage
[Minikube](https://minikube.sigs.k8s.io/docs/) was used for development. Below are instructions on how to set up and use Elasticsearch and Kibana with Minikube.

> [!TIP]
> To get started with Minikube, please refer to this [guide](https://minikube.sigs.k8s.io/docs/start) on their website.

After you have installed and configured Minikube, clone this repository by running the command below:

```bash
git clone https://github.com/bgebelek/kubernetes-elastic-stack
```

Then, navigate to the cloned repository, ensure you are on the correct tag or branch for your intended release, and execute the command below:

```bash
kubectl apply -k .
```

This command creates all objects declared in the manifest files using Kustomize. Once the objects are created, access Kibana using one of the following methods depending on your setup:

### Docker Desktop

Since Docker Desktop runs Minikube inside a restricted virtual machine, a direct connection to the node is not possible. Use the command below to establish a connection with `kb-service`:

```bash
minikube service kb-service --url=true
```

> [!TIP]
> If you prefer to be directed to the service automatically, exclude the `--url=true` option from the command above.

> [!WARNING]
> If the Docker driver is being used, the terminal may need to remain open for the duration of the Kibana session. If this applies to you, you will see the following warning after running the command above: `❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.`

### Other

Obtain the IP address of the Minikube node by running:

```bash
minikube ip
```

Then retrieve the node port for `kb-service`:

```bash
kubectl get service kb-service
```

Since TLS is not configured for this release, use the HTTP protocol to connect to Kibana:

```bash
http://<node-ip>:<node-port>
```

## Cleanup

Once you are done, you can delete all objects created by Kustomize by running the following command from inside the repository:

```bash
kubectl delete -k .
```
