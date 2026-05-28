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

Before applying resources, an internal Certificate Authority (CA) will need to be created to sign digital certificates created by init containers in the pods of statefulset objects. First, navigate to the cloned repository and execute the command below to create a pod running an Elasticsearch container:

```bash
kubectl run es-temp --image elasticsearch:9.3.2
```

Then, create a digital certificate and a private key for the CA:

```bash
kubectl exec es-temp -- /usr/share/elasticsearch/bin/elasticsearch-certutil ca --pem --out ca.zip
```

Afterward, encode the ZIP archive using base64:

```bash
kubectl exec es-temp -- base64 ca.zip
```

Copy the base64 encoded output from the command above and decode it for local storage:

```bash
echo '<base64 encoded output>' | base64 -d > ca.zip
```
> [!NOTE]
> `kubectl cp` cannot be used as the binary for `tar` is absent in the container image for Elasticsearch.

The pod can now be deleted:

```bash
kubectl delete pod es-temp
```

A TLS secret type is used to store the certificate and key of the CA. Before referencing these assets, unzip the ZIP archive:

```bash
unzip ca.zip && rm ca.zip
```

The extracted assets can be stored anywhere that is desired. Once relocated, the placeholders in the manifest `ca-tls-secret.yml` can be replaced with the new locations:

```yaml
data:
  tls.crt: path/to/ca/cert/file
  tls.key: path/to/ca/key/files
```

Next, a password will need to be created for the `elastic` user. The value for the key `password` in the manifest `es-auth-secret.yml` can be changed to accomplish this.

```yaml
stringData:
  username: elastic
  password: changeme #assign your new password here
```

. . .

```bash
kubectl apply -k .
```

This command creates all objects declared in the manifest files using Kustomize. Once the objects are created, access Kibana using one of the following methods depending on your setup:

### Docker Desktop

Since Docker Desktop runs Minikube inside a restricted virtual machine, a direct connection to the node is not possible. Use the command below to establish a connection with `kb-nodeport-service`:

```bash
minikube service kb-nodeport-service --url=true
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

Then retrieve the node port for `kb-nodeport-service`:

```bash
kubectl get service kb-nodeport-service
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
