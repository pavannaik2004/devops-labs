minikube start --cpus=2 --memory=4096 # start the cluster with 2 CPUs and 4 GB RAM
minikube status # check it's running

minikube dashboard # opens the Kubernetes Dashboard in the browser (keep this terminal open, use a new one below)

cd 10thprog # folder with deployment.yaml and service.yaml

kubectl apply -f deployment.yaml # create nginx-deployment (2 replicas)
kubectl apply -f service.yaml # create nginx-service (NodePort)

kubectl get pods # 2 nginx-deployment-xxxx pods (Running)
kubectl get nodes # minikube node (Ready)
kubectl get svc # kubernetes (ClusterIP) + nginx-service (NodePort 80:3xxxx/TCP)

kubectl logs <pod-name> # copy a pod name from "kubectl get pods"
kubectl describe pod <pod-name> # full details: node, IP, image, events

minikube service nginx-service # opens nginx "Welcome to nginx!" page in the browser

# Now go to the Dashboard tab -> Workloads: see Deployments, Pods, Replica Sets (all green / Running)

kubectl delete pods --all # clean-up: delete all pods (deployment recreates them)
kubectl delete svc --all # delete all services
kubectl delete deployments --all # delete all deployments (pods go away for good)
