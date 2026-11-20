# Program 7 – Deploy an Nginx Web App on Kubernetes using Minikube

This program runs **2 copies of the Nginx web server** on a local Kubernetes cluster (Minikube), opens them to the browser with a **Service**, and then cleans everything up.

---

## Key terms

| Term           | What it is                                                                                                    |
| -------------- | ------------------------------------------------------------------------------------------------------------- |
| **Minikube**   | Runs a single-node Kubernetes cluster on your own machine (inside Docker).                                    |
| **kubectl**    | Command-line tool used to talk to the Kubernetes cluster.                                                     |
| **Pod**        | The smallest unit in Kubernetes. It wraps one or more running containers (here, one nginx container).         |
| **ReplicaSet** | Keeps a fixed number of identical pods running. If a pod dies, it starts a new one.                           |
| **Deployment** | Manages ReplicaSets. You describe _what_ you want (image, number of copies), and it makes it happen.          |
| **Service**    | Gives a set of pods one stable address and load-balances traffic between them.                                |
| **NodePort**   | A Service type that opens a port (30000–32767) on the node so you can reach the app from outside the cluster. |

**Flow:** `Deployment` → creates a `ReplicaSet` → creates `Pods` ← `Service` sends traffic to them using **labels** (`app: nginx`)

---

## Project structure

```
7thprog/
├── nginx-deployment.yaml
├── nginx-service.yaml
└── README.md
```

---

## Step 0 (Optional) – Generate the YAML files with kubectl

Instead of typing the YAML by hand, kubectl can write it for you.

```bash
kubectl create deployment nginx-deployment \
  --image=nginx:latest \
  --replicas=2 \
  --port=80 \
  --dry-run=client -o yaml > gen-deployment.yaml
```

```bash
kubectl create service nodeport nginx-service \
  --tcp=80:80 \
  --node-port=30080 \
  --dry-run=client -o yaml > gen-service.yaml
```

| Part                                    | Meaning                                                                   |
| --------------------------------------- | ------------------------------------------------------------------------- |
| `create deployment nginx-deployment`    | Make a Deployment named `nginx-deployment`                                |
| `--image=nginx:latest`                  | Container image to run                                                    |
| `--replicas=2`                          | Run 2 pods                                                                |
| `--port=80`                             | Port the container listens on                                             |
| `create service nodeport nginx-service` | Make a NodePort Service named `nginx-service`                             |
| `--tcp=80:80`                           | `service port : container (target) port`                                  |
| `--node-port=30080`                     | Fixed outside port (must be 30000–32767)                                  |
| `--dry-run=client`                      | **Don't create anything in the cluster**, only show what would be created |
| `-o yaml > file.yaml`                   | Save that output as a YAML file                                           |

> ⚠️ **Note:** The generated files use different labels (`app: nginx-deployment` and `app: nginx-service`). A Service only sends traffic to pods whose labels match its `selector`, so these two **won't work together** as-is. The ready-made files below are already fixed. Only use the `gen-*.yaml` files after changing every label/selector to `app: nginx`.
>
> The `\` at the end of a line means "the command continues on the next line". It must be the **very last character** on the line (no space after it).

---

## The YAML files

### `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2 # run 2 copies (pods) of nginx
  selector:
    matchLabels:
      app: nginx # manage pods that have the label app=nginx
  template: # blueprint for each pod
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          imagePullPolicy: IfNotPresent # use the local image if it's already there
          ports:
            - containerPort: 80
              protocol: TCP
```

### `nginx-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: NodePort # expose the service on a port of the node
  selector:
    app: nginx # send traffic to pods labelled app=nginx (must match the deployment)
  ports:
    - name: 80-80
      protocol: TCP
      port: 80 # port of the service inside the cluster
      targetPort: 80 # containerPort of the nginx pods
      nodePort: 30080 # port opened on the Minikube node
```

---

## Step 1 – Start the cluster

```bash
minikube start
```

Starts the local Kubernetes cluster and configures `kubectl` to use it.

```bash
minikube status
```

Checks that the cluster is running. Expected output:

```
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

**Only if `minikube start` fails**, delete the cluster and start fresh with the Docker driver:

```bash
minikube delete && minikube start --driver=docker
```

---

## Step 2 – Go to the folder with the YAML files

```bash
cd devops-labs/7thprog
```

---

## Step 3 – Check that nothing exists yet

```bash
kubectl get deployments
kubectl get pods
```

Expected output: `No resources found in default namespace.`

---

## Step 4 – Create the Deployment and the Service

```bash
kubectl apply -f nginx-deployment.yaml
```

Creates the Deployment, which creates a ReplicaSet and then 2 nginx pods.
Output: `deployment.apps/nginx-deployment created`

```bash
kubectl apply -f nginx-service.yaml
```

Creates the NodePort Service that points to the nginx pods.
Output: `service/nginx-service created`

> `apply -f` creates (or updates) whatever the YAML file describes.

---

## Step 5 – Verify

```bash
kubectl get deployments
```

Shows the Deployment and how many pods are ready → `nginx-deployment   2/2`

```bash
kubectl get replicasets
```

Shows the ReplicaSet created by the Deployment (DESIRED 2, CURRENT 2, READY 2).

```bash
kubectl get pods
```

Shows the 2 nginx pods → both `Running`

```bash
kubectl get services
```

Shows the Service and its ports → `nginx-service   NodePort   80:30080/TCP`

---

## Step 6 – Open the app in the browser

Creates a NodePort Service automatically from the Deployment. It copies the Deployment's selector, so the labels always match.

```bash
kubectl expose deployment nginx-deployment \
  --type=NodePort \
  --port=80
```

```bash
minikube service nginx-deployment
```

> If you get `Error from server (AlreadyExists)`, a service with that name is left over from an earlier run. Delete it and run `expose` again:
>
> ```bash
> kubectl delete service nginx-deployment
> ```

**Expected result:** the browser shows the **"Welcome to nginx!"** page.

> 💡 With the Docker driver on Linux, `http://localhost:30080` will **not** work. The app runs on Minikube's own IP. Use `minikube ip` to see it, or `minikube service ... --url`.

---

## Step 7 – Clean up

Delete exactly what the YAML files created:

```bash
kubectl delete -f nginx-service.yaml
kubectl delete -f nginx-deployment.yaml
```

If you used `kubectl expose` (Option B), also delete that service:

```bash
kubectl delete service nginx-deployment
```

**Or** delete everything in the default namespace:

```bash
kubectl delete svc --all
kubectl delete deployments --all
```

> `kubectl delete svc --all` also removes the built-in `kubernetes` service, but Kubernetes recreates it automatically, so it's harmless.

Optionally, stop the cluster:

```bash
minikube stop
```

---

## Quick reference

| Command                                                      | What it does                             |
| ------------------------------------------------------------ | ---------------------------------------- |
| `minikube start`                                             | Start the local cluster                  |
| `minikube status`                                            | Check the cluster is running             |
| `minikube ip`                                                | Show the cluster's IP address            |
| `kubectl apply -f <file>`                                    | Create/update resources from a YAML file |
| `kubectl get deployments`                                    | List Deployments                         |
| `kubectl get replicasets`                                    | List ReplicaSets                         |
| `kubectl get pods`                                           | List Pods                                |
| `kubectl get services`                                       | List Services and their ports            |
| `kubectl expose deployment <name> --type=NodePort --port=80` | Create a Service for a Deployment        |
| `minikube service <service-name>`                            | Open a Service in the browser            |
| `kubectl delete -f <file>`                                   | Delete resources defined in a YAML file  |
| `kubectl delete svc --all`                                   | Delete all Services                      |
| `kubectl delete deployments --all`                           | Delete all Deployments (and their pods)  |
| `minikube stop`                                              | Stop the cluster                         |

---

## Expected outcome

- 2 nginx pods running under one Deployment
- A NodePort Service exposing them on port 30080
- The **"Welcome to nginx!"** page opens in the browser
