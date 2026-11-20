# Program 8 – Run a Python Pod in Kubernetes

## File Creation Command

Generate a starter `python-pod.yaml` (then edit it in nano to match the file in this folder):

```bash
kubectl run python-pod \
  --image=python:3.12-slim \
  --restart=Never \
  --dry-run=client -o yaml \
  --command -- python -u -c "import time; print('Welcome to Kubernetes!'); print('Python Pod is running'); time.sleep(3600)" \
  > python-pod.yaml

nano python-pod.yaml
```

## Run

```bash
minikube start                     # start the local cluster

kubectl apply -f python-pod.yaml   # create the pod
kubectl get pods                   # wait until python-pod is Running
kubectl logs -f python-pod         # watch the program output (Ctrl+C to stop)

kubectl exec -it python-pod -- python --version

kubectl exec -it python-pod -- python          # interactive Python shell (exit() to quit)

kubectl exec -it python-pod -- python -c "print('Hello Students!')"

kubectl exec -it python-pod -- python -c "a=10; b=20; print('Sum =', a+b)"

kubectl delete -f python-pod.yaml  # clean up
```

## Expected log output

```
Welcome to Kubernetes!
Python Pod is running successfully.
Python Execution: 1
Python Execution: 2
Python Execution: 3
Python Execution: 4
Python Execution: 5
Python program execution completed.
Container is waiting...
```
